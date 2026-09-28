# NPC Demo Data Load, DataClean and NIS Indexing — Working Notes

These are notes from actually doing this on a dev VM, not a polished spec. Follow it top to bottom.
The troubleshooting section at the end is the important part — most of us will hit at least two of
those problems.

Assumes your dev environment is already built and the five Newforma services exist
(see the main dev setup doc for VS, MySQL, IIS, Office and the initial build).

**What this has and hasn't been proven on.** This was written from one VM: a cloned, AzureAD-joined,
non-domain Windows 11 box running 64-bit Office. On that machine we got a single project restored,
DataClean run clean, and indexing started with the legacy tables cleaned up. What we did **not**
verify: a full index run to 100% on a large data set, more than two projects, and anything involving
NIX / Info Exchange. If your VM is domain-joined or uses a local account instead of AzureAD, the
`Setup-Branch` credential section probably won't apply to you.

---

## Before you start

Check these, because almost every weird failure later traces back to one of them.

**1. MySQL is running and you can reach it**

```powershell
Get-Service MySQL
& "C:\Program Files\MySQL\MySQL Server 8.4\bin\mysql.exe" --user=root --password=root -e "SHOW DATABASES;"
```

**2. `N4Path` points at your built `n4.exe`**

This one is easy to miss and it breaks the project restore with a confusing error. Use the **x64**
build if you're on 64-bit Office.

```powershell
# check
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Newforma\20XX" -Name N4Path

# set if empty or wrong
Set-ItemProperty -Path "HKLM:\SOFTWARE\Wow6432Node\Newforma\20XX" `
  -Name "N4Path" -Value "C:\<your-repo>\enterprise-suite\Solutions\n4\bin\x64\Debug"
```

**3. Server name registry keys match your machine name**

If your VM was cloned from someone else's image, these will still have the old machine name and
nothing will work. Check all three:

```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Newforma\20XX\NIS" -Name ProjectCenterServerName
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Newforma\20XX\NWS" -Name HomeServer
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Newforma\20XX\NFS" -Name IndexingServerName
```

All three should equal `$env:COMPUTERNAME`. If not, fix them:

```powershell
$me = $env:COMPUTERNAME
Set-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Newforma\20XX\NIS" -Name ProjectCenterServerName -Value $me
Set-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Newforma\20XX\NWS" -Name HomeServer -Value $me
Set-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Newforma\20XX\NFS" -Name IndexingServerName -Value $me
```

Don't set these to `localhost`. The code compares them against `Environment.MachineName` and takes a
faster path when they match, so `localhost` actually makes things worse.

**4. `NPCSInstallDir` points at your build output, not Program Files**

On a Debug build the code takes whatever is in this key and walks three levels up to find the SQL
deployment scripts. On a fresh VM it's often still the installer default
(`C:\Program Files (x86)\Newforma\...`), which doesn't exist on a source-only box.

```powershell
# check
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Newforma\20XX\NPCSGeneral" -Name NPCSInstallDir

# set if wrong
Set-ItemProperty -Path "HKLM:\SOFTWARE\Wow6432Node\Newforma\20XX\NPCSGeneral" `
  -Name "NPCSInstallDir" `
  -Value "C:\<your-repo>\enterprise-suite\Solutions\ProjectCenterServer\bin\Debug"
```

If this is wrong you'll see something like this when restoring or resetting:

```
Unable to import the global DB: Failed to create the database: newformaglobal
Script: C:\Program Files (x86)\SQL\globalDatabaseDeployment.xml
ERROR: invalid script: Could not find a part of the path...
```

This is a different key from `N4Path` and they fix different problems. Don't mix them up.

**5. Import the `.REG` file that came with the backup data**

The backup set usually ships with a `<ServerName>.REG`. It registers the server the data came from so
project pinning resolves properly during restore. Double-click it, or:

```powershell
Start-Process regedit.exe -ArgumentList "/s `"<path-to-file>.REG`"" -Wait
```

Fair warning: we got a project restore to work without doing this, so it may not always be required.
Do it anyway — it's in the official process and skipping it is the kind of thing that bites you later.

**6. Make sure no mail can go out**

We're loading customer-ish demo data, so kill the mail path before anything else.

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Wow6432Node\Newforma\20XX\SendMail" `
  -Name "SmtpServer" -Value "127.0.0.1"
```

To confirm nothing can send, check all four of these:

```powershell
# should be 127.0.0.1
(Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Newforma\20XX\SendMail").SmtpServer
# should be False
(Test-NetConnection 127.0.0.1 -Port 25 -WarningAction SilentlyContinue).TcpTestSucceeded
# should be "not installed"
Get-Service NewformaSMTPRelayService -ErrorAction SilentlyContinue
# should be empty
Get-NetTCPConnection -LocalPort 25 -State Listen -ErrorAction SilentlyContinue
```

Also look in the Project Center Server log for this line, which means the mailer turned itself off:

```
Notification mailer config string was empty. Using defaults and disabling the mailer.
```

---

## Part 1 — Load the project data

**Restore one project at a time from its `.NPB` file. Do not restore the `.NGB` global backup.**

The reason: the global backup drags in the whole global contact table, and DataClean does not clean
those contacts. We tried it and ended up with ~765 real email addresses sitting in the database that
nothing would sanitize. Restoring per project keeps the global data small and DataClean covers all of
it.

Steps:

1. Start the core service if it isn't up. The UI restore needs it.

   ```powershell
   Start-Service NewformaProjectCenterService
   ```

2. Open NPC.

3. Go to **Utilities → Restore Project through Backup**, pick your `.npb`, and let it run.

4. You'll get a popup at the end. If it says *"Restored project '<name>' from backup file ..."* then it
   worked, even if the same popup also says it couldn't add you as a project team member. That second
   part is harmless (see troubleshooting).

5. Check it actually landed:

   ```powershell
   $mysql = "C:\Program Files\MySQL\MySQL Server 8.4\bin\mysql.exe"
   & $mysql --user=root --password=root -N -e "SELECT id, project_name, deleted FROM newformaglobal.project;"
   ```

   You want your project with `deleted = 0`. Write the GUID down, you need it for DataClean.

Repeat for more projects if you want more data.

---

## Part 2 — DataClean

This sanitizes the email addresses that came in with the project items, so nothing in the database is
a deliverable address.

### Build it first

`enterprise-tools\DataClean\DataClean.sln`, Debug config. Clone `enterprise-tools` if you don't have it.

If it builds but blows up on startup with something like:

```
Unhandled Exception: System.IO.FileLoadException: Could not load file or assembly 'MySql.Data, Version=9.6.0.0...'
```

then MSBuild picked a different `MySql.Data` version than the app expects at runtime. Check what you
actually have:

```powershell
(Get-Item "C:\<your-repo>\enterprise-tools\DataClean\bin\Debug\MySql.Data.dll").VersionInfo.FileVersion
```

Then add a binding redirect inside the existing `<runtime>` section of
`bin\Debug\DataClean.exe.config`, with `newVersion` set to the version you just found:

```xml
<assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
  <dependentAssembly>
    <assemblyIdentity name="MySql.Data" publicKeyToken="c5687fc88969c44d" culture="neutral" />
    <bindingRedirect oldVersion="0.0.0.0-9.6.0.0" newVersion="8.4.0.0" />
  </dependentAssembly>
</assemblyBinding>
```

### Run it

Run it per project, using the GUID from Part 1:

```powershell
& "C:\<your-repo>\enterprise-tools\DataClean\bin\Debug\DataClean.exe" -p <project-guid> -u
```

It should finish with `Updating companies and contacts in database...` and no exceptions.

### Don't be confused by the console output

DataClean prints the **original** addresses as it works:

```
Adding contact fn=Adam ln=Klose 01 em=aklose01@jma-demo.com co=jma-demo
```

What actually gets written to the database is the sanitized version, `aklose01_@_jma-demo.com`. The
`_@_` makes it undeliverable. Check the database, not the console:

```powershell
$mysql = "C:\Program Files\MySQL\MySQL Server 8.4\bin\mysql.exe"

# how many are clean vs not
& $mysql --user=root --password=root -N -e "SELECT COUNT(*) FROM newformaglobal.contact WHERE deleted=0 AND email_address<>'';"
& $mysql --user=root --password=root -N -e "SELECT COUNT(*) FROM newformaglobal.contact WHERE deleted=0 AND email_address LIKE '%\_@\_%';"

# anything still a real address
& $mysql --user=root --password=root -N -e "SELECT CONCAT(first_name,' ',last_name,' | ',email_address) FROM newformaglobal.contact WHERE deleted=0 AND email_address<>'' AND email_address NOT LIKE '%\_@\_%';"

# domain breakdown
& $mysql --user=root --password=root -N -e "SELECT SUBSTRING_INDEX(email_address,'@',-1) AS domain, COUNT(*) FROM newformaglobal.contact WHERE deleted=0 AND email_address<>'' GROUP BY domain ORDER BY 2 DESC;"
```

On One Oak Street we got 26 contacts: 25 sanitized, 1 left alone. The one left alone was the logged-in
dev's own `@newforma.com` address, which DataClean skips on purpose. That's expected.

### Known DataClean failures

**`System.Data.ConstraintException: Column 'id' is constrained to be unique...`**

A `contact_link` row with that ID already exists. The code path that updates an existing contact's link
has no try/catch around the insert, unlike the path for brand new contacts. Known gap in
`DCServices.cs::CreateContacts()`. Skip that project and carry on with the next one.

**`NPCSOfflineException: The Project Center Server 'localhost' is offline`**

The core service isn't running. `Start-Service NewformaProjectCenterService` and retry.

**`NoSuchDatabaseException: Project does not exist on this server`**

The project's `.npb` never actually restored, so there's no project database for it. Nothing to clean,
skip it.

### What DataClean does not do

It only touches contacts that came from project items. It does not scrub contacts that were already in
the global contact table. So it is not a general "make this database safe" tool. If you restored a
global backup, DataClean will not save you.

---

## Part 3 — NIS indexing

Goal is to get real data into the `nis` tables so the Index Statistics screen shows actual numbers.

1. Put the sample data somewhere outside your user profile, e.g. `C:\Dev\repos\Procore Project`.

2. In NPC, open the project → **Edit Project Settings → Project Folders → Add Folder** and point it at
   the data folder. Point it at the top level folder, not a subfolder, otherwise you only index a
   fraction of the files.

3. Save. You may get an error popup about a missing `ProjectFilesDataExtract_*.dat` file. Ignore it,
   it's a dashboard thing and has nothing to do with indexing.

4. Make sure the indexing services are up:

   ```powershell
   Get-Service NewformaIndexingService, NewformaTextService, NewformaProjectCenterService
   ```

5. Watch it work. The indexer runs on about a 60 second cycle, so give it a few minutes.

   In the UI: **Project Center Administration → Servers tab → Servers dropdown → Index → Index
   Statistics → Get Statistics**.

   Or straight from the database, which is faster to check:

   ```powershell
   $mysql = "C:\Program Files\MySQL\MySQL Server 8.4\bin\mysql.exe"
   & $mysql --user=root --password=root -N -e "SELECT COUNT(*) FROM nis.scope;"
   & $mysql --user=root --password=root -N -e "SELECT COUNT(*) FROM nis.content_directory;"
   & $mysql --user=root --password=root -N -e "SELECT COUNT(*) FROM nis.content_table_set;"
   ```

   Zeros mean nothing has started. Once it's working these go up.

### How to tell indexing is actually healthy

Right after a fresh `Reset-Newforma`, the `nis` database will contain these 11 tables:

```
filter_queue, filter_server, namespace, priority_1, priority_2, priority_3,
priority_4, priority_5, server_assignment, supported_extension, text_exclusion
```

They're legacy. The schema deployment recreates them because a brand new database replays the whole
version history from 7.0 onwards. The Indexing Service is supposed to migrate them and drop them the
first time it starts up properly (`IndexingDAO.PerformUpgrade`).

So: **if those tables are still sitting there, your Indexing Service has never successfully started.**
It's a really useful health check.

```powershell
# 11 = indexing service has not initialised. 0 = healthy.
& "C:\Program Files\MySQL\MySQL Server 8.4\bin\mysql.exe" --user=root --password=root -N -e `
"SELECT COUNT(*) FROM information_schema.tables WHERE table_schema='nis' AND table_name IN ('priority_1','priority_2','priority_3','priority_4','priority_5','filter_queue','server_assignment','namespace','supported_extension','text_exclusion','filter_server');"
```

When it does start properly you'll see this in
`C:\Newforma\20XX\Logs\IndexingService\IndexingServiceLog_*.txt`:

```
upgrading filter queue
dropping filter queue tables
filter queue tables dropped
```

---

## Troubleshooting

### "Failed to restore: Failed to delete the partially created project <guid>"

`N4Path` is empty or wrong, so the server can't find `n4.exe`. The popup is misleading — the restore
failed first, and then the rollback failed too, which is what you actually see.

Confirm it in `C:\Newforma\20XX\Logs\ProjectCenterServer\*.txt`:

```
System.ApplicationException: n4.exe does not exist in current application's directory or N4Path
```

Fix `N4Path` (see Before you start), restart the Project Center Service, then run `Reset-Newforma` to
clear the half-created project rows before trying again.

### Services won't start: "Only one usage of each socket address..."

You restarted too quickly and the old process hasn't let go of its port yet. Wait 30 to 60 seconds and
start it again. If it keeps happening, check you don't have ProjectCenterServer running under the
Visual Studio debugger, because that holds the port.

### `Setup-Branch` fails with "The user name or password is incorrect"

If you log into the VM with an AzureAD / work account (`whoami` shows a domain and `Get-LocalUser` has
no real accounts), `n4 SetCredentials` can't validate it and Setup-Branch dies at the very last step.
Annoying part: it has already deleted and recreated the services by then, so they exist with no valid
logon account and won't start.

For a dev box, just run them as LocalSystem:

```powershell
$svcs = @("NewformaTextService","NewformaIndexingService","NewformaProjectCenterService",
          "NewformaWorkService","NewformaEventService")
foreach ($s in $svcs) { sc.exe config $s obj= LocalSystem }   # note the space after obj=

Start-Service NewformaProjectCenterService
Start-Sleep 25
Start-Service NewformaIndexingService,NewformaTextService,NewformaWorkService,NewformaEventService
```

Heads up: because Setup-Branch died before finishing, it never wrote the server name registry keys.
Go back and set those three keys manually (see Before you start).

### `Setup-Branch` fails with "CreateService FAILED 1072: marked for deletion"

Something still has a handle on a service that's being deleted. Usually it's the Indexing Service,
which often refuses to stop.

```powershell
Get-Process NewformaIndexingService -ErrorAction SilentlyContinue | Stop-Process -Force
sc.exe delete NewformaIndexingService     # "does not exist" here is fine, means it's gone
```

Close services.msc, Task Manager's Services tab and the NPC Server Errors window, then re-run
Setup-Branch. Those windows are what pin the service.

### "No such host is known" everywhere, nothing indexes

Open `C:\Newforma\20XX\Logs\IndexingService\*.txt` or the PCS log and you'll see this on repeat:

```
Error while ensuring project folders are in the index on server <name>
System.Net.Sockets.SocketException (0x80004005): No such host is known
```

The name it's trying to reach comes from `NIS\ProjectCenterServerName`. If your VM was cloned, that
key still holds the original machine's name, which doesn't resolve on your network.

Fix the three registry keys, then **properly restart the Indexing Service**. This is the bit that
caught us out:

```powershell
# WRONG - does nothing if the service is already running
Start-Service NewformaIndexingService

# RIGHT
Get-Process NewformaIndexingService -ErrorAction SilentlyContinue | Stop-Process -Force
Start-Service NewformaIndexingService
```

The service only reads that registry key once, at startup. `Start-Service` on an already-running
service is a no-op, so it keeps using the stale name from memory and you'll swear the fix didn't work.
`Restart-Service` often fails on this service too ("cannot be stopped"), which is why the force-kill
is there.

You can tell it worked because a new log file appears and the legacy tables get dropped.

### Popup: "Cannot add the current user as project team member to this confidential project"

Harmless. The project is flagged confidential and NPC tried to add you, but your contact isn't in the
freshly reset global contact table. The project data restored fine. Ignore it.

### Error popup about `ProjectFilesDataExtract_<guid>_5.dat` not found

Harmless. It's the dashboard's Project Files widget looking for a data extract that hasn't been
generated yet. Close it. A background maintenance task creates it later.

### NIX / Info Exchange errors in the logs

Things like `NIXOfflineException: The Info Exchange Server '<name>' is offline` or endpoint errors on
`https://<name>/RemoteWeb/NixApi.svc`. You can ignore these for data load, DataClean and indexing.
NIX isn't needed for any of it.

---

## After a `Reset-Newforma`

When you run it you get three prompts. We answer them:

```
Reset Newforma Databases?      Y
Deploy NIX?                    Y
Pause between operations?      N
```

It takes several minutes and prints a lot. Warnings like
`sql failed: Function 'newforma_wordbreaker' already exists` are fine, that just means the MySQL plugin
is already registered. It finishes with `If using Project Email Addressing, run Start-SmtpRelay` and
restarts the services.

Reset wipes the global contact table, so **you have to run DataClean again**. Check what survived
before you redo work:

```powershell
$mysql = "C:\Program Files\MySQL\MySQL Server 8.4\bin\mysql.exe"

# is the project still there?
& $mysql --user=root --password=root -N -e "SELECT id, project_name, deleted FROM newformaglobal.project;"

# how many contacts left? 1 usually means only your own, so DataClean needs rerunning
& $mysql --user=root --password=root -N -e "SELECT COUNT(*) FROM newformaglobal.contact WHERE deleted=0 AND email_address<>'';"

# did the legacy nis tables come back? 11 = restart the indexing service
& $mysql --user=root --password=root -N -e "SELECT COUNT(*) FROM information_schema.tables WHERE table_schema='nis' AND table_name IN ('priority_1','priority_2','priority_3','priority_4','priority_5','filter_queue','server_assignment','namespace','supported_extension','text_exclusion','filter_server');"
```

Depending on how much the reset took out you may need to redo Part 1 as well. In our case the project
and the indexing state survived and only the contacts were gone, so it was just DataClean.

The registry fixes (`N4Path`, the three server name keys, `SmtpServer`) survive a reset. Worth
re-checking anyway, it takes ten seconds.

---

## Quick checklist

- [ ] MySQL up, `root` works
- [ ] `N4Path` set to the x64 `n4.exe` folder
- [ ] `NPCSInstallDir` set to `...\ProjectCenterServer\bin\Debug`
- [ ] `.REG` file from the backup set imported
- [ ] `NIS\ProjectCenterServerName`, `NWS\HomeServer`, `NFS\IndexingServerName` all = machine name
- [ ] `SendMail\SmtpServer` = `127.0.0.1`, no SMTP relay installed, nothing on port 25
- [ ] Project restored from `.npb` via the UI, `deleted = 0` in `newformaglobal.project`
- [ ] No `.ngb` global backup restored
- [ ] DataClean run per project, unsanitized count is 0 (ignoring your own `@newforma.com`)
- [ ] Project folder added under Project Folders
- [ ] The 11 legacy `nis` tables are gone
- [ ] `nis.scope` and `nis.content_directory` are above 0
- [ ] Index Statistics shows files indexed
