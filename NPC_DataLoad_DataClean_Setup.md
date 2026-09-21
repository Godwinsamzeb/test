# NPC Demo Data Load & DataClean - Setup Guide

## Overview

This document describes the procedure to restore Newforma demo/backup data (NGB + NPB files) into a local NPCS development environment, and to run the DataClean tool to sanitize contact data before bringing services online.

**Scope covered:**
- Restoring global database (NGB) and project databases (NPB)
- Fixing known environment issues encountered during restore
- Building and running the DataClean tool
- Known issues and their workarounds

---

## Prerequisites

- NPCS dev environment fully set up (see `NPC_Dev_Setup_Consolidated.md`)
- MySQL running with root credentials configured
- Access to the `enterprise-suite` repo (for N4 CLI)
- Access to the `enterprise-tools` repo (for DataClean)
- Backup data folder containing:
  - `<ServerName>.REG`
  - One or more `<ServerName>-<timestamp>.ngb` files
  - Multiple `<projectGuid>-<ProjectName>-<timestamp>.npb` files
  - Optionally an `NCS Backup` folder with `.bak` files (purpose unconfirmed — verify with source team before use)

---

## Understanding the Backup File Types

| Extension | Contents | Restore Method |
|---|---|---|
| `.REG` | Registry export identifying the source server | Import into registry directly |
| `.NGB` | Zipped MySQL dump (`.sql`) of the `newformaglobal` database | Extract + `N4 Database Restore -g`, or manual `mysql` import |
| `.NPB` | Zipped set of numbered binary table dumps (`001_project.bin`, `002_project_item.bin`, etc.) for one project snapshot | Restored automatically by `N4 Database Restore -g` (per project), not manually importable |
| `.NMB` | Small manifest tying a backup set together | Used internally by the restore process |

**Do not attempt to manually parse or import `.npb`/`.nmb` files.** They are proprietary binary formats consumable only by the NPCS restore engine.

---

## Step 1 — Stop All Services

Before touching any data, stop Newforma services and disable outbound mail paths.

```powershell
Stop-NewformaServices
```

Optionally, to guarantee no email can be delivered even if a background job fires during restore:

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Wow6432Node\Newforma\20XX\SendMail" -Name "SmtpServer" -Value "127.0.0.1"
```

---

## Step 2 — Import the REG File

Double-click the `.REG` file, or:

```powershell
Start-Process -FilePath "regedit.exe" -ArgumentList "/s `"<path-to-file>.REG`"" -Wait
```

This establishes the originating server identity so project-pinning resolves correctly during restore.

---

## Step 3 — Fix Known Registry Path Issues (Dev/Debug Builds Only)

On a dev machine (Debug build), the N4 restore command computes its deployment-script folder using:

```csharp
string installDir = LocalMachineSettings.Singleton.GetNPCSDirectory();
#if DEBUG
installDir = installDir + @"\..\..\..";
#endif
```

`GetNPCSDirectory()` reads the registry value:

```
HKLM\SOFTWARE\Wow6432Node\Newforma\20XX\NPCSGeneral\NPCSInstallDir
```

On a fresh dev machine this value is often left at its installer default (`C:\Program Files (x86)\Newforma\20XX\Newforma Project Center Server`), which does not exist on a source-only dev box. This causes:

```
Unable to import the global DB: Failed to create the database: newformaglobal
Script: C:\Program Files (x86)\SQL\globalDatabaseDeployment.xml
ERROR: invalid script: Could not find a part of the path...
```

**Fix:** point this key at the actual repo build output folder, 3 levels above `Solutions`:

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Wow6432Node\Newforma\20XX\NPCSGeneral" `
  -Name "NPCSInstallDir" `
  -Value "C:\<repo-root>\Solutions\ProjectCenterServer\bin\Debug"
```

Verify:

```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Newforma\20XX\NPCSGeneral" -Name "NPCSInstallDir"
```

> Note: `N4Path` (a separate registry value under the same `20XX` key) is **not** related to this issue — it only affects how other tools locate `n4.exe` itself. Do not confuse the two.

---

## Step 4 — Restore Global DB + All Projects (Disaster Recovery Mode)

Locate the N4 CLI:

```powershell
Invoke-N4 help Database Restore
```

Usage:

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

- `-g <path-to-ngb>` — Disaster recovery mode. Restores the global DB **and every associated project automatically** in one pass. Omit `-G` to restore all projects.
- `-d` — ⚠️ Re-GUIDs all projects and project items. Only use if duplicating data intentionally.
- `-x` — Makes each restored project immediately available on Info Exchange (NIX). Recommended to omit until after DataClean has run.

**Run:**

```powershell
Invoke-N4 Database Restore -g "<path-to-newest>.ngb"
```

You will be prompted:

```
Please make sure that the NPCS and NIS services are not running at '<host>'.
HIT ENTER TO CONTINUE (CTRL-C TO EXIT)
```

Press Enter to proceed. On completion you will see:

```
Successfully restored global DB and projects!
Please start the NPCS and NIS service and hit enter to continue...
```

**Do not press Enter yet.** Proceed to DataClean first (Step 6) before starting services.

### Reviewing the Restore Log

Each run writes a task log to:

```
%LOCALAPPDATA%\Temp\n4 Database Restore Log <guid>.txt
```

To identify the correct (successful) log among multiple attempts:

```powershell
Get-ChildItem "$env:LOCALAPPDATA\Temp" -Filter "n4 Database Restore Log*" | ForEach-Object {
    $content = Get-Content $_.FullName -Raw
    [PSCustomObject]@{
        File = $_.Name
        Modified = $_.LastWriteTime
        Succeeded = $content -match "Successfully restored global DB and projects"
        FailedProjectCount = ([regex]::Matches($content, "Unable to restore project")).Count
    }
} | Format-Table -AutoSize
```

### Known Restore-Time Failure Patterns

| Log Pattern | Meaning | Action |
|---|---|---|
| `About to restore with backup path '<folder>'` (no filename) | Project's `LastBackupPath` in global DB was empty; no `.npb` file matched. Silently skipped during restore, no `.npr` database created. | Safe to skip — no data exists to restore for this project. |
| `Unable to restore project: Logical error occurred: unrecognized zip file '<folder>'` | Same root cause as above, surfaced as an explicit error. Project row is cleaned up (deleted) automatically. | Not a bug — export did not include this project's backup data. |
| Project restores successfully but is later a "phantom" for DataClean (`NoSuchDatabaseException: Project does not exist on this server`) | Confirms no `npr_<guid>` database exists for that project. | Skip DataClean for these project IDs. |

To identify all restored (real) project IDs directly from the database:

```powershell
& "<path-to-mysql.exe>" --user=root --password=root -N -e "SELECT id, project_name FROM newformaglobal.project WHERE deleted=0;"
```

---

## Step 5 — Build the DataClean Tool

DataClean lives in the `enterprise-tools` repo:

```powershell
git clone https://github.com/Newforma/enterprise-tools.git
```

Located at: `enterprise-tools\DataClean\`

**Build via Visual Studio:**
1. Open `DataClean.sln`
2. Build → Build Solution

**Or via Developer Command Prompt / PowerShell for VS:**
```powershell
cd enterprise-tools\DataClean
msbuild DataClean.sln /p:Configuration=Debug
```

Output: `enterprise-tools\DataClean\bin\Debug\DataClean.exe`

### Known Build Issue — MySql.Data Version Conflict

MSBuild may pick a lower `MySql.Data` version at build time than what the compiled app expects at runtime, causing:

```
Unhandled Exception: System.IO.FileLoadException: Could not load file or assembly 'MySql.Data, Version=9.6.0.0...'
```

**Fix:** Add a binding redirect to `bin\Debug\DataClean.exe.config`, inside the existing `<runtime>` section:

```xml
<assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
  <dependentAssembly>
    <assemblyIdentity name="MySql.Data" publicKeyToken="c5687fc88969c44d" culture="neutral" />
    <bindingRedirect oldVersion="0.0.0.0-9.6.0.0" newVersion="8.4.0.0" />
  </dependentAssembly>
</assemblyBinding>
```

(Adjust `newVersion` to match whatever version is actually present in `bin\Debug\MySql.Data.dll` — check with:)

```powershell
(Get-Item "<path>\bin\Debug\MySql.Data.dll").VersionInfo.FileVersion
```

---

## Step 6 — Start NPCS Core Service (Required for DataClean)

DataClean's `UpdateCompaniesAndContacts()` step notifies the running NPCS service to invalidate its cache. If NPCS is not running, it throws:

```
Unhandled Exception: Newforma.Core.Remote.NPCSOfflineException: The Project Center Server 'localhost' is offline
```

**Fix:** start just the core service (not NIX, not full service start):

```powershell
Start-Service "Newforma Project Center Service"
Get-Service "Newforma Project Center Service"
```

Confirm the SMTP safety net from Step 1 is still in place before doing this.

---

## Step 7 — Run DataClean Per Project

### Usage

```
DataClean -p <projectid guid> [-f <project folder root>] [-u] [-r] [-d]
  -u  Create and update companies and contacts from project data
  -r  Repath project folders and create stub files for related files/emails
  -d  Perform diagnostics only
```

### What `-u` Actually Does

- Scans each project's item-level contacts (`GetUniqueProjectContacts()`).
- Skips contacts whose email matches `IsEmailSkippable()` (currently: `ber.com`, `newforma.com`, `newforming.com` domains).
- For contacts **not already present** in the global `contact` table: creates a new global contact using a **sanitized** email (`user@domain.com` → `user_@_domain.com`, non-deliverable).
- For contacts **already present** in the global `contact` table (e.g., brought in via the NGB import): updates name/company only — **does not touch or sanitize their existing email address.**

**Important:** This tool only closes the specific gap of unlinked project-item contacts. It is not a blanket email-scrubbing tool for the whole database. If the source export's global contacts already use safe/fictional domains, that is a property of the source data, not something DataClean guarantees.

### Get the List of Restored Project IDs

```powershell
& "<path-to-mysql.exe>" --user=root --password=root -N -e "SELECT id FROM newformaglobal.project WHERE deleted=0;"
```

### Run Per Project

```powershell
& "<path>\DataClean\bin\Debug\DataClean.exe" -p <project-guid> -u
```

Repeat for each project ID. Review console output for:
- `Adding contact ...` — new contact created with sanitized email (safe)
- `Adding contact link for ...` — new link to a new or existing contact
- `Updating contact ...` — existing contact matched, email untouched

### Known DataClean Failure Patterns

| Error | Cause | Action |
|---|---|---|
| `System.Data.ConstraintException: Column 'id' is constrained to be unique...` | A `contact_link` row with a duplicate ID already exists; the code path for updating an existing contact's link has no try/catch around the insert (unlike the "new contact" path, which does). | Known gap in `DCServices.cs::CreateContacts()`. Skip and continue to the next project; revisit later if a permanent fix is wanted. |
| `Newforma.Core.Remote.NPCSOfflineException: ... is offline` | NPCS service is not running. | Start the core NPCS service (Step 6) and retry. |
| `Newforma.Core.Remote.NoSuchDatabaseException: Project does not exist on this server` | The project's `.npb` was never restored (see Step 4 known failure patterns); no `npr_<guid>` database exists. | Skip — no data exists for this project. |

---

## Step 8 — Post-Restore Database Cleanup (Optional, from internal merge procedure)

```sql
UPDATE project p, synchronized_project sp, SERVER s1, SERVER s2
SET p.add_to_nix = 1
WHERE p.p_key = sp.p_key AND s1.s_key = sp.s_key AND s2.s_key = p.s_key
  AND p.deleted = 0 AND sp.deleted = 0 AND s1.deleted = 0 AND s2.deleted = 0
  AND s1.server_type = 2;
```

---

## Step 9 — Start Remaining Services and Verify

Only after DataClean has been run against all restorable projects:

```powershell
Start-NewformaServices
Invoke-N4 npcs verifyinstallation
Invoke-N4 nix verifyinstallation
N4 npcs addnixprojects
N4 npcs syncprojects -s <npcs-name> -t <nix-name>
N4 npcs dumpsyncqueue
```

---

## Data Safety Checklist (Before Approving Any Load as "Complete")

- [ ] Confirm the source export's email domains — audit distinct domains present, don't assume based on a partial sample:
  ```powershell
  & "<mysql.exe>" --user=root --password=root -N -e "SELECT SUBSTRING_INDEX(email_address, '@', -1) AS domain, COUNT(*) AS cnt FROM newformaglobal.contact WHERE deleted=0 AND email_address != '' GROUP BY domain ORDER BY cnt DESC;"
  ```
- [ ] Flag any domain that resembles a real, operating company (not a known fictional/demo pattern) to the data provider before proceeding.
- [ ] Confirm SMTP relay is not installed/configured, or is pointed at an unreachable host, until go-live is intended.
- [ ] Record which project IDs failed to restore (no backup file) and which failed DataClean (duplicate link constraint) so they can be revisited.
- [ ] Do not assume DataClean alone makes a dataset "safe to email" — it only patches unlinked project contacts, not the full global contact table.

---

## Reference: Full List of Verification Commands

```powershell
# List all restored projects
& "<mysql.exe>" --user=root --password=root -N -e "SELECT id, project_name FROM newformaglobal.project WHERE deleted=0 ORDER BY project_name;"

# List existing project databases
& "<mysql.exe>" --user=root --password=root -N -e "SHOW DATABASES LIKE 'npr_%';"

# List distinct contact email domains
& "<mysql.exe>" --user=root --password=root -N -e "SELECT SUBSTRING_INDEX(email_address, '@', -1) AS domain, COUNT(*) AS cnt FROM newformaglobal.contact WHERE deleted=0 AND email_address != '' GROUP BY domain ORDER BY cnt DESC;"
```
