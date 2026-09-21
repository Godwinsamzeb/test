# NPC Demo Data Load & DataClean - Setup Guide

## Overview

Procedure to restore Newforma demo/backup data (NGB + NPB files) into a local NPCS development environment, and run the DataClean tool before bringing services online.

---

## Prerequisites

- NPCS dev environment set up (see `NPC_Dev_Setup_Consolidated.md`)
- MySQL running with root credentials configured
- Access to `enterprise-suite` repo (N4 CLI)
- Access to `enterprise-tools` repo (DataClean)
- Backup data folder containing:
  - `<ServerName>.REG`
  - One or more `<ServerName>-<timestamp>.ngb` files
  - Multiple `<projectGuid>-<ProjectName>-<timestamp>.npb` files

---

## Backup File Types

| Extension | Contents | Restore Method |
|---|---|---|
| `.REG` | Registry export identifying the source server | Import into registry directly |
| `.NGB` | Zipped MySQL dump (`.sql`) of the `newformaglobal` database | `N4 Database Restore -g` |
| `.NPB` | Zipped set of numbered binary table dumps for one project snapshot | Restored automatically by `N4 Database Restore -g`, not manually importable |
| `.NMB` | Manifest tying a backup set together | Used internally by the restore process |

---

## Step 1 — Stop All Services

```powershell
Stop-NewformaServices
```

Optional safety net to block outbound mail during restore:

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Wow6432Node\Newforma\20XX\SendMail" -Name "SmtpServer" -Value "127.0.0.1"
```

---

## Step 2 — Import the REG File

```powershell
Start-Process -FilePath "regedit.exe" -ArgumentList "/s `"<path-to-file>.REG`"" -Wait
```

---

## Step 3 — Fix NPCSInstallDir (Dev/Debug Builds Only)

On a Debug build, the restore command computes its deployment-script folder from:

```
HKLM\SOFTWARE\Wow6432Node\Newforma\20XX\NPCSGeneral\NPCSInstallDir
```

If this still points at the installer default (`C:\Program Files (x86)\Newforma\20XX\...`), which does not exist on a dev-only box, restore will fail with:

```
Unable to import the global DB: Failed to create the database: newformaglobal
```

**Fix** — point it at the repo build output, 3 levels above `Solutions`:

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Wow6432Node\Newforma\20XX\NPCSGeneral" `
  -Name "NPCSInstallDir" `
  -Value "C:\<repo-root>\Solutions\ProjectCenterServer\bin\Debug"
```

---

## Step 4 — Restore Global DB + All Projects

```powershell
Invoke-N4 help Database Restore
```

```
n4 Database Restore
  [-g globalBackup(*.ngb)]
  [-G projectId]
  [-p projectBackup(*.npb)]
  [-r globalAndProjectBackup(*.nrb)]
  -m
  -d
  -x
```

- `-g <path-to-ngb>` — Disaster recovery mode. Restores the global DB and every associated project in one pass.
- `-d` — Re-GUIDs all projects/items. Only use intentionally.
- `-x` — Makes each project available on NIX immediately. Omit until after DataClean.

**Run:**

```powershell
Invoke-N4 Database Restore -g "<path-to-newest>.ngb"
```

Press Enter at the confirmation prompt. On completion:

```
Successfully restored global DB and projects!
Please start the NPCS and NIS service and hit enter to continue...
```

**Do not press Enter yet** — proceed to DataClean first.

### Known Restore Failure

| Log Pattern | Cause | Action |
|---|---|---|
| `About to restore with backup path '<folder>'` (no filename) / `Unrecognized zip file` | Project's `LastBackupPath` was empty; no matching `.npb`. Project row auto-removed. | Skip — no data available for this project. |

To list restored project IDs:

```powershell
& "<path-to-mysql.exe>" --user=root --password=root -N -e "SELECT id, project_name FROM newformaglobal.project WHERE deleted=0;"
```

---

## Step 5 — Build DataClean

```powershell
git clone https://github.com/Newforma/enterprise-tools.git
```

Path: `enterprise-tools\DataClean\`

Build via Visual Studio (`DataClean.sln` → Build Solution) or:

```powershell
cd enterprise-tools\DataClean
msbuild DataClean.sln /p:Configuration=Debug
```

Output: `enterprise-tools\DataClean\bin\Debug\DataClean.exe`

### Known Build Issue — MySql.Data Version Conflict

If build output shows a version conflict warning (`MSB3277`) for `MySql.Data`, the compiled `DataClean.exe` may fail at runtime with:

```
System.IO.FileLoadException: Could not load file or assembly 'MySql.Data, Version=9.6.0.0...'
```

**Fix** — get the actual version/token from the DLL that is present in `bin\Debug`, then add a matching binding redirect.

1. Read the strong name directly off the DLL on disk:
   ```powershell
   [System.Reflection.AssemblyName]::GetAssemblyName("<path>\bin\Debug\MySql.Data.dll").FullName
   ```
   This prints something like:
   ```
   MySql.Data, Version=<ACTUAL_VERSION>, Culture=neutral, PublicKeyToken=<ACTUAL_TOKEN>
   ```

2. Add this block inside the existing `<runtime>` section of `bin\Debug\DataClean.exe.config`, substituting the values from step 1:
   ```xml
   <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
     <dependentAssembly>
       <assemblyIdentity name="MySql.Data" publicKeyToken="<ACTUAL_TOKEN>" culture="neutral" />
       <bindingRedirect oldVersion="0.0.0.0-<HIGHER_VERSION>" newVersion="<ACTUAL_VERSION>" />
     </dependentAssembly>
   </assemblyBinding>
   ```

> The public key token is not something set in source code — it is a fixed signature embedded in the `MySql.Data.dll` binary by its publisher. Always pull the value from the DLL itself (step 1) rather than hardcoding a token from a different environment.

---

## Step 6 — Start NPCS Core Service

DataClean requires NPCS to be running to complete its update step. If not running:

```
Newforma.Core.Remote.NPCSOfflineException: The Project Center Server 'localhost' is offline
```

**Fix:**

```powershell
Start-Service "Newforma Project Center Service"
```

---

## Step 7 — Run DataClean Per Project

```
DataClean -p <projectid guid> [-f <project folder root>] [-u] [-r] [-d]
  -u  Create and update companies and contacts from project data
  -r  Repath project folders and create stub files
  -d  Diagnostics only
```

`-u` creates new global contacts (with a sanitized, non-deliverable email) for project-level contacts not already linked in the global `contact` table. Existing matched contacts are updated (name/company) but their email is left untouched.

List restored project IDs:

```powershell
& "<path-to-mysql.exe>" --user=root --password=root -N -e "SELECT id FROM newformaglobal.project WHERE deleted=0;"
```

Run per project:

```powershell
& "<path>\DataClean\bin\Debug\DataClean.exe" -p <project-guid> -u
```

### Known DataClean Failures

| Error | Cause | Action |
|---|---|---|
| `System.Data.ConstraintException: Column 'id' is constrained to be unique...` | Duplicate `contact_link` ID; missing try/catch in `DCServices.cs::CreateContacts()` for the existing-contact path. | Skip project, continue to the next. |
| `NoSuchDatabaseException: Project does not exist on this server` | No `.npb` was restored for this project (see Step 4). | Skip — no data exists. |

---

## Step 8 — Start Remaining Services

```powershell
Start-NewformaServices
Invoke-N4 npcs verifyinstallation
Invoke-N4 nix verifyinstallation
N4 npcs addnixprojects
N4 npcs syncprojects -s <npcs-name> -t <nix-name>
```
