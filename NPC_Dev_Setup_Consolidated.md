# NPC Development Environment Setup — Consolidated Guide

> **Purpose of this document**
> This page merges the client-provided setup documentation with the additional steps, deviations, and observations our team recorded while actually setting up the development environment. It is organized into three parts:
> 1. **Consolidated Development Environment Setup Guide** — a single, end-to-end walkthrough (original steps + our additions merged in).
> 2. **Additional Steps Added by Our Team (For Client Confirmation)** — everything we added that wasn't in the original document, called out for your review.
> 3. **Open Questions / Clarifications Required from Client** — items we need your input on before we can finalize this guide.
>
> Throughout Section 1, any step contributed by our team is marked with a 📌 **Team Addition** callout. These are also listed on their own in Section 2 for easy review.

---

# 1. Consolidated Development Environment Setup Guide

## Visual Studio 2026 Professional

- **Prerequisite:**
  - Install Visual Studio 2015
    - Downloadable from `\\newforma.local\aperus\SQA\Installation Kits\Microsoft\Visual Studio 2015 Pro`
    - Accept the defaults and add these options to install:
      - Microsoft Office Developer Tools
      - Visual C++
        - Common Tools for Visual C++ 2015
        - Microsoft Foundation Classes for C++
        - Windows XP Support for C++ (because it includes Windows 7.1 SDK)
  - **NOTE:** Various errors may be encountered while installing Visual Studio 2015, but most can be safely ignored. As long as installation completes despite the errors, the installation can be considered a success.
  - **NOTE:** (Keep in mind for when you clone the repos and open VS 2026) If you encounter errors when opening VS 2026 and numerous projects say "incompatible" or have issues loading, try reloading those specific projects with dependencies. If that does not work, open the VS 2026 install and choose the "Repair" option under the "More" combo box on the installer main screen, then close and reopen VS 2026.

- Install Visual Studio 2026 **Professional** from https://learn.microsoft.com/en-us/visualstudio/releases/2026/release-notes.

  > 📌 **Team Addition:** In practice, we downloaded the installer via the https://newformacorporate.sharepoint.com/:f:/s/NPC-VDI-Files/IgB_h1V5dQq4TJBPQk5r58AIAR6qhJghuf2dT8ztQDcOqdc?e=gdSE2B shared path rather than the public Microsoft Learn URL above. We downloaded the entire folder and unzipped it to our chosen directory, then ran the installer executable **as Administrator**. See Section 2 for the exact path to confirm with the client.

  > 📌 **Team Addition:** After launching the installer, choose the **Custom** installation option. Several features will already be pre-selected — leave those as-is and additionally select the workloads/components shown in the screenshots/transcripts below. On top of the originally documented components, also:
  > - Expand **Windows and Web Development**
  > - Expand **Universal Windows App Development Tools**
  > - Select **Windows 10 SDK (10.0.10240)** — this avoids a build issue when building the C++ plugins.

  > 📌 **Team Addition:** During the install, you may see repeated "package is missing" prompts — this can occur 5–10 times. Select **Skip** each time and continue; installation will complete and you'll land on the Launch screen, which indicates a successful install. This aligns with the existing note below that various errors can be safely ignored, and in our testing the "missing package" error was the only one we actually encountered.

- **Note:**
  - In order to build C++ projects, you will need MSVC in version 140 (versions 141 and 142 WILL NOT WORK) and Tools for Visual Studio 2017 for Windows XP in C++. Please make sure it is included (see its inclusion in the 'Desktop development with C++' screenshot below).
  - If you had previously installed C++ compiling tools in Visual Studio 2026, it's likely you had version 141, 142, or 143 and when you opened the solution the `.vcxproj` files changed `PlatformToolset` to v141/v142/v143. To build, you must revert those changes, except the ones to `_vars.bat`.
  - Update the path in `scripts\set_vstudio_vars.bat` to point `VS_INSTALL_DIR` to your custom install path of Visual Studio, if not using the default path.
  - Check the path in `scripts\set_vstudio_vars.bat` for `VS_C_COMPILER`. If it doesn't match your machine's `cl.exe` path, update it.
  - We recommend using the pre-made Visual Studio installation configuration file ([newforma.vsconfig](https://github.com/Newforma/enterprise-technical-documentation/blob/vs_2022_update/Newforma%20Dev%20Environment/newforma.vsconfig)). When imported into the Visual Studio installer, it will automatically include the necessary workloads/components. See [Import a configuration using the Visual Studio Installer](https://learn.microsoft.com/en-us/visualstudio/install/import-export-installation-configurations?view=vs-2026#import-a-configuration-using-the-visual-studio-installer).
  - Do not click "Remove out-of-support components." It may warn you a couple of times — you can ignore those warnings.

  > 📌 **Team Addition:** This is the approach we actually followed — rather than selecting individual components manually, we used the **Import Configuration** template method:
  > 1. Open the Visual Studio Installer and choose **Import Configuration**.
  > 2. Download the config file from the path specified in the original document.
  > 3. Use it to proceed with the import, then proceed with the install.

  - **Note:** Please choose only one approach — either the configuration file or manual setup. If choosing manual configuration, refer to the images below. If any image fails to load, download this document for a transcript of each image, and review each image/transcription before proceeding. Make sure to sign in with your Newforma credentials.

*(Screenshots and component transcripts for ASP.NET and Web Development, .NET Desktop Development, Desktop Development with C++, Office/SharePoint Development, and Additional Individual Components remain unchanged from the original document — see the linked source for the full image set.)*

## NUnit 3 (Automated Testing)

- Install NUnit 3 console (runner)
  - Option 1: At command prompt: `dotnet add package NUnit.Runners --version 3.9.0`
  - Option 2: Use **NUnit.Console-3.9.0.msi** from GitHub: https://github.com/nunit/nunit-console/releases/tag/v3.9
- Add the install location to the system PATH environment variable.
- When using Option 2, the default installation location is `C:\Program Files (x86)\NUnit.org\nunit-console`

> 📌 **Team Addition:** We used **Option 2** (the MSI installer) — flagging this as our preferred/tested route for client confirmation.

## Source Control

### Install git
- https://git-scm.com/download/win
  - If you want to use GitGui or GitBash as a UI, select it during install.
  - Make sure large file support is checked.
  - If you used the Git installer and checked LFS you don't need this step; otherwise install git lfs: https://git-lfs.github.com/
- In Visual Studio, Tools → Options → Source Control → Plug-in Selection: set to Microsoft Git provider.

### Install a git GUI
- GitHub Desktop is the recommended GUI.
  - **WARNING:** There are unlicensed branches of GitHub Desktop that may link to malicious sites. Download only from the [official website](https://desktop.github.com/download/). If in doubt, check with an established developer.
- From your git tool, clone:
  - **enterprise-suite** (where Project Center stuff primarily lives)
  - **enterprise-technical-documentation** (where this document lives)
  - **enterprise-tools**
  - **enterprise-api** (NewformaAPI for third-party connectors and NewformaLink for web/mobile clients)
- Link command line tools to the git repo: using RegEdit, create key `HKEY_CLASSES_ROOT\Newforma Installation` → String Value `DeveloperRepositoryPath` → set to the root folder where you cloned enterprise-suite (e.g., `c:\git\enterprise-suite`).
- Set up GPG signing: https://newformaholdings.atlassian.net/wiki/spaces/EPH/pages/2941562227/Signing+GitHub+commits+with+GPG

  > 📌 **Team Addition:** Because this is a GUI-driven installation, the registry key setup and GPG signing steps above are easy to skip past without noticing — **do not omit them**, both are required for the environment to work correctly.

- **NOTE:**
  - All repos after build need a lot of disk space — at least 100GB (as of July 2024, closer to 200GB).
  - Store all repos at `C:\git`, outside your user area, to avoid access/permission issues.

  > 📌 **Team Addition:** If you clone repos directly from within Visual Studio (rather than a standalone git GUI), the default clone location is the `repos` folder under your Windows user profile. **Do not accept this default** — it will later cause NuGet restore failures due to excessive path length. Always redirect the clone path to a location outside the user folder (e.g., `C:\git`), consistent with the note above.

## MS Office

Office/Outlook IT may have already installed this on your system — probably the 64-bit version.

NPC must run in the same bitness as Outlook. To run NPC in 32-bit mode, you'll need to change the bitness of your Office installation. Either way, uninstall and reinstall Office using the custom install option.

Note — most developers use 64-bit Office (as of June 2025, developers have run into issues with 32-bit setups, particularly around Info Exchange, so some troubleshooting/bugfixes may be required).

You can determine your version via Task Manager → Processes; a process name with `*32` indicates a 32-bit program.

- Indicate your Office version via environment variable: set `OFFICE64=1` if running 64-bit Office.

**NOTE:** Setting the environment variable above typically brings conditional references into project builds. If you see a strange assembly-missing error, check the failing project for these variables causing reference issues.

### Register Redemption (used by our email code)
- Run `<repo>\enterprise-suite\Solutions\ThirdParty\Redemption\install.exe` and point its install folder to `<repo>\enterprise-suite\Solutions\ThirdParty\Redemption`.

  > 📌 **Team Addition:** This step is easy to overlook — **don't skip Register Redemption**.

## MySQL

Install the currently supported version of MySQL.
- If a version of MySQL is already installed, follow the upgrade instructions: https://newformaholdings.atlassian.net/wiki/spaces/FAT/pages/4703158278/Upgrade+to+2026.2+with+MySQL+8.4.8
- Every build of the develop branch includes the MySQL installer (`.msi`) for the currently supported version in the `MySQLMigrator\MySQL` directory.
- To install the currently supported version, go to `\\newforma.local\aperus\QAData\Builds\Project_Center\develop\[currentNpcVersion]\[latestCompletedBuildNumber]\MySQLMigrator\MySQL`
  - Example (2026.2): `\\newforma.local\aperus\QAData\Builds\Project_Center\develop\2026.2.0\39517\MySQLMigrator\MySQL`

  > 📌 **Team Addition:** In practice, we installed MySQL from the https://newformacorporate.sharepoint.com/:f:/s/NPC-VDI-Files/IgB_h1V5dQq4TJBPQk5r58AIAR6qhJghuf2dT8ztQDcOqdc?e=gdSE2B, rather than the build path documented above. 

- Under Setup Type, choose **Typical**.
  - Use `root` for the password.
  - Windows Service Name must be: `MySQL`
- **Wordbreaker/newforma hash function:** Copy `<repo>\enterprise-suite\Solutions\MySqlWorkbreaker\release\x64\*.{dll,pdb}` to `C:\Program Files\MySQL\MySQL Server 8.4\lib\plugin`.

  > 📌 **Team Addition:** Don't miss this Wordbreaker/hash-function copy step — it's a small step that's easy to skip but is required for correct operation.

  **NOTE:** If using the community installer for an older MySQL version and you hit the error "unknown variable 'mysqlx_port=0.0'", the fix (courtesy of Eric Buterbaugh):
  1. Cancel the MySQL installation.
  2. Launch MySQL Community installer and uninstall all components, including the installer.
  3. Confirm the `..\programdata\mysql` folder is deleted.
  4. Launch the MySQL Community installer and upgrade the catalog (install MySQL Workbench first if the catalog update option doesn't appear).
  5. Go to Start → MySQL → MySQL Community installer and install the correct version of MySQL Server.

## IIS Installation

To run Newforma Info Exchange (NIX) you will need the following settings.

- Install IIS via 'Turn Windows features on or off' → expand Internet Information Services:
  - Expand Web Management Tools:
    - Expand IIS 6 Management Compatibility: IIS 6 Management Console; IIS Metabase and IIS 6 configuration compatibility
    - IIS Management Console
    - IIS Management Scripts and Tools
    - IIS Management Service
  - Expand World Wide Web Services:
    - Expand Application Development Features: .Net Extensibility 4.8; ASP.NET 4.8 (auto-selects others); CGI; ISAPI Extensions; ISAPI Filters
    - Expand Common Http Features: Default Document; Directory Browsing; HTTP Errors; HTTP Redirection; Static Content
    - Health and Diagnostics (select all options)
    - Expand Performance Features: Static Content Compression
    - Expand Security: IP Security; Request Filtering; Windows Authentication

## Configure IIS

- Run MMC (Microsoft Management Console) → File → Add/Remove Snap-in...
  - Select Certificates → Add → Computer account → Local computer (default) → Finish → OK
  - Under Certificates → Personal → Certificates, right-click → All Tasks → Request New Certificate...
  - Select **Next** twice, with default settings.

    > 📌 **Team Addition:** In our environment, the second "Next" is disabled because a certificate is unavailable. We still need direction from the client on how to proceed here — see Open Questions.

  - Check **Web Server**, then select the warning text "More information is required..."
  - Under the Subject tab:
    - Type: Full DN → `CN=[devMachineName].newforma.local, O=Newforma, OU=Dev, L=Manchester, ST=NH, C=US` → Add
    - Alternative name: DNS → `[devMachineName]` → Add, then `[devMachineName].newforma.local` → Add
  - General tab → Friendly name: `[devMachineName]` → OK → Enroll → Finish
- In IIS Manager: right-click Default Web Site → Edit Bindings... → add an `https` binding to the certificate above.
- **NOTE:** Since the certificate uses both short and fully qualified domain names, `HKLM\software\Wow6432Node\Newforma\<edition>\DeveloperProvisionNixName` can be set to either.
- **NOTE:** This is not required to build and run NPCS — if you have certificate authority issues, you can skip this step for now.
- **NOTE:** From the root server node, IIS → Authentication should have only Anonymous Authentication Enabled; all others must be Disabled.
- **NOTE:** If you have an expired cert, make sure you bind to the new certificate.

## Nuget

- Copy the latest `NuGet.exe` (https://www.nuget.org/downloads) into a folder on your PATH (e.g., `C:\windows`).
- In Visual Studio 2026:
  - Tools → NuGet Package Manager → Package Manager Settings → General: set 'Default package management format' to `PackageReference`.
  - Tools → NuGet Package Manager → Package Manager Settings → Package Sources — add:
    - `Newforma` → `https://nuget.newforma.com/api/v2/`
    - `github` → `https://nuget.pkg.github.com/Newforma/index.json` (add via Package Manager Console: `NuGet Sources Add -Name "github" -Source "https://nuget.pkg.github.com/Newforma/index.json" -UserName "<your login email>" -Password "<your password>"`). Password must be a GitHub personal access token with scopes: `repo`, `write:packages`, `read:packages`.
    - `Nuget.org` → `https://api.nuget.org/v3/index.json`

## Build and Run NPC InstallerUtility

- Install the 2010 Tools for Office runtime ([details](http://msdn.microsoft.com/en-us/library/ms178739.aspx), [download](https://www.microsoft.com/en-us/download/details.aspx?id=105522)).
- Using Visual Studio 2026, open `<repo>\enterprise-suite\Solutions\All.sln`:
  - Accept the prompt to install Visual Studio Tools for Office, if shown.
  - Build the `InstallerUtility` project.
- Open a **command prompt as Administrator** in `<repo>\enterprise-suite\Solutions\enterprise-core\InstallerUtility\bin\Debug` and run, in order:
  - `InstallerUtility SaveDatabaseCredentials root localhost 3306`
  - `InstallerUtility ConfigureMySqlIni`
  - `InstallerUtility ConfigurePerformanceCounters NPCS`
  - `InstallerUtility ConfigurePerformanceCounters NWS`

  > 📌 **Team Addition:** Of these four commands, the first two run successfully right away. The last two will initially throw a "local disk" style error — this is expected. Ignore it, complete the remaining setup steps below, then re-run the same two commands; they should return with no output, which indicates success.

## Setup PowerShell

- Copy `<repo>\enterprise-technical-documentation\Newforma Dev Environment\Microsoft.PowerShell_profile.ps1` to `C:\Users\[username]\Documents\WindowsPowerShell`.
  - **NOTE:** If you have an existing PowerShell profile, merge its contents into `Microsoft.PowerShell_profile.ps1`.

  > 📌 **Team Addition:** On our machines, the Documents folder is redirected to OneDrive, so the path above didn't apply directly. Instead:
  > 1. Run `$PROFILE` in PowerShell to get the actual profile path — expect a response under your OneDrive-backed Documents location.
  > 2. In that directory, create a `WindowsPowerShell` folder and paste the `.ps1` script there.
  > 3. Continue with the remaining steps below.
  > 4. Ensure the read-only property is **not** checked on the script — To make sure run the below scripts:
  > ```powershell
  > Get-Item "C:\Users\[username]\OneDrive - Newforma, Inc\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1" -Stream Zone.Identifier
  > Unblock-File "C:\Users\[username]\OneDrive - Newforma, Inc\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1"
  > ```

- Ensure read-only property is not checked on `Microsoft.PowerShell_profile.ps1`.
- In `Microsoft.PowerShell_profile.ps1`, change the `NewformaServicesPS1` variable to point at `<repo>\enterprise-suite\Solutions\scripts\NewformaServices.ps1`.
- Start PowerShell as Administrator:
  - Run `Set-ExecutionPolicy RemoteSigned`, answer Y at the prompt.
  - If asked "Which branch do you want to use?", choose Mainline (you may need to close/reopen PowerShell to see the prompt).
  - Ignore errors like `'n4 was not found'` — these tools haven't been built yet.
- **NOTE:** Unless you change the execution policy, the script likely won't run.

## Import the certificate used to sign the Outlook add-in

- Double-click `<repo>\enterprise-suite\Solutions\OutlookAddIn2013\Newforma--Inc-Newforma-Code-Signing.pfx`.
- In the wizard:
  - Import for the current user.
  - Check 'Mark this key as exportable'.
  - Password: `Millyard`

## ProjectWise

- Changes as of v12.5.1: PW no longer needs to be installed locally to build (SDK moved to `enterprise-suite\Solutions\ThirdParty\ProjectWise\SDK\10.00.03.262`). To test the integration, install ProjectWise Connection Client v01.00.03+ from `\\newforma.local\data\departments\Development\Installation Kits\Bentley ProjectWise\ProjectWise Explorer CONNECT Edition` (the `pwclt` installer includes both Connection Client and ProjectWise Explorer, required for the EDMS plugin).
- Pre v12.5.1:
  - To **build** the ProjectWise 8.x/10.x EDMS plugin: install the ProjectWise SDK from `\\newforma.local\data\departments\Development\Installation Kits\Bentley ProjectWise\PW 10\projectwisesdk10000167en.exe`, then copy the registry key `HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Bentley\ProjectWise SDK\10.00` into `...\08.11`.
  - To **use** the ProjectWise 10.x EDMS plugin (optional): install ProjectWise Explorer from `\\newforma.local\data\departments\Development\Installation Kits\Bentley ProjectWise\PW 10\pwclt10000167en.exe`.

## Build All Projects

**IMPORTANT** — when building in Visual Studio, ensure the build is set to `x64` when running 64-bit Office, otherwise use `x86`.
**IMPORTANT** — run Visual Studio as an administrator when building.

- Run PowerShell as Administrator:
  - Run `Build-Newforma`, answering Y to the build prompts.
  - You do not need to run all unit tests.
  - Packages may not restore correctly for the AllPlugins project — this can be ignored.

  > 📌 **Team Addition:** Per client direction, we skipped building the AllPlugins solution specifically, since it consistently fails due to a **Moq NuGet package reference error**. We built the remaining solutions and confirmed they build without issue. Flagged for confirmation — see Open Questions.

*Troubleshooting Tip: The build creates a hidden `.vs` folder within `<repo>\enterprise-suite\Solutions\` — if the build fails, removing the contents of the `.vs` folder may be required to successfully build.*

- **NOTE:** If C++ projects aren't building, you likely have a newer version of compiling tools — you need version 140.
- **NOTE:** If the Newforma Project Center Service isn't running after build, there's likely a MySQL issue — debug the ProjectCenterServer project.
- **NOTE:** If the Newforma Project Center Service keeps stopping, there's likely a MySQL issue — debug the ProjectCenterServer project.
- **NOTE:** If you have problems building, ensure you have enough hard drive space — at least 100GB.
- **NOTE:** If you have problems building, try building step by step — C++ projects first, then Plugins, then All.
- **NOTE:** If the build fails with empty logs, the VS folder is likely not set up correctly in `enterprise-suite\Solutions\scripts\set_vstudio_vars.bat`.
- **NOTE:** If the build fails because of SQL Migration, remove the postbuild step from the MySQLMigrator solution.
- **NOTE:** If DownloadWeb, RemoteWeb, or UserWeb appear as "unloaded", delete the `.csproj.user` file in those directories (can happen on clean dev machines).
- **NOTE:** If the All solution fails with Leadtools version conflict errors, run `git clean -xdf` in the enterprise-suite repo and build again.

## NPC Configuration

- Add your user account to policy groups:
  - Windows 10 & 11: Run 'Local Security Policy' → Local Policies → User Rights Assignment:
    - Log on as a batch job
    - Log on as a service

  > 📌 **Team Addition:** In our environment, the "check names" functionality could not resolve AzureAD account names when adding them directly to the policy groups above. As a workaround, we created a **local group** and added the account to that group instead, then granted access via the local group:
  > ```
  > net localgroup NewformaServiceAccounts /add
  > net localgroup NewformaServiceAccounts "AzureAD\<user-UPN>" /add
  > ```
  > In Local Security Policy → Local Policies → User Rights Assignment, add the **local group** (`NewformaServiceAccounts`) to:
  > - Log on as a service
  > - Log on as a batch job
  >
  > Then run:
  > ```
  > gpupdate /force
  > Setup-Branch "AzureAD\<user-UPN>" "<password>"
  > ```
  > This workaround needs client confirmation — see Open Questions.

- Setup Mainline 'branch':
  - Restart PowerShell as Admin so it can initialize properly after Build-Newforma.
  - Run `Setup-Branch newforma\[username] [password]`.
  - **NOTE:** If your password has special characters, escape them (e.g., `"pas$word"` → `` "pas`$word" ``).
  - **NOTE:** If you see "Cannot invoke method on null valued expression", complete all steps through "Reset the Newforma databases", then close and restart PowerShell as admin.
- Create a share of `<repo>\enterprise-suite\Solutions\XSLTemplates` as `Templates` (right-click → Properties → Sharing → Advanced Sharing → share name `Templates`).
- In RegEdit, create/update String Value `HKEY_LOCAL_MACHINE(HKLM)\SOFTWARE\Wow6432Node\Newforma\[Current Edition]\SendMail\SmtpServer` = `Belichick`.
- In RegEdit, create/update String Value `...\SendMail\FromAddress` = your email address.
- Import `<repo>\enterprise-suite\Solutions\ProjectCenterServer\ProjectCenterServer.reg` into the registry.
- In RegEdit, at `HKLM\SOFTWARE\Wow6432Node\Newforma`, right-click Permissions → add full control for NETWORK SERVICE and 'Authenticated Users'.
- In Explorer, add full control for NETWORK SERVICE and 'Authenticated Users' to `<repo>\enterprise-suite\Solutions\InfoExchangeServer` (Access Denied errors on bin/obj folders and some JS files are expected).
- Windows 10 — Windows Defender Firewall → Advanced Settings → Inbound Rule → New Rule:
  - Rule Type: Port
  - Protocol/Ports: Specific Local Ports: `9002,9003,9004`
  - Action: Allow the connection
  - Profile: Domain, Private, Public
  - Name: `NPC`
- **NOTE:** If your computer is not in the newforma domain, run `Setup-Branch` for your domain or as a local PC, e.g. `Setup-Branch "MyPc\MyLogin" "Password"`.
- **NOTE:** On a non-developer computer, if services refuse to start (`Error 1068: The dependency service or group failed to start.`), go to Services → each Newforma service's properties → Log On tab → change to "Local System account".

  > 📌 **Team Addition:** As a development-only fallback for Error 1068 on a non-domain dev VM, this PowerShell scripts the same fix documented above:
  > ```powershell
  > $NpcServices = @("NewformaTextService", "NewformaIndexingService", "NewformaProjectCenterService", "NewformaWorkService", "NewformaEventService")
  > foreach ($NpcService in $NpcServices) { sc.exe config $NpcService obj= LocalSystem }
  > Start-NewformaServices
  > ```
  > Flagged for client confirmation that this scripted approach is acceptable for dev machines — see Open Questions.

## Reset the Newforma databases

- From PowerShell, run:
  - `Reset-Newforma`
  - `Invoke-N4 npcs verifyinstallation`
  - `Invoke-N4 nix verifyinstallation`

## Run the Newforma NPC Client
- From PowerShell, run: `Invoke-NPC`

## Run the Newforma NPCS
- From PowerShell, run: `Start-NewformaServices`

## Newforma PDF Printer
Install the PDF Printer to have a print option with the NewForma PDF driver (Adobe Reader or other). Installer: https://github.com/Newforma/enterprise-suite/blob/develop/Setup/Common/Prerequisites/Newforma%20PDF/NewformaPDFSetup.exe

## Newforma Web Services APIs (NL and N-API)

- Add Newforma NuGet server to the NuGet Package Manager: https://nuget.newforma.com/api/v2/
- Add GitHub NuGet server: https://nuget.pkg.github.com/Newforma/index.json
  - **NOTE:** Your password for the GitHub NuGet source must be a personal access token with scopes: `repo`, `write:packages`, `read:packages`.
- Open IIS Manager → click the server name in Connections pane → Management section → Feature Delegation:
  - **Authentication - Anonymous** → Read/Write
  - **Authentication - Windows** → Read/Write

### NewformaLink (NL)
For web and mobile client applications — setup: https://newformaholdings.atlassian.net/wiki/spaces/EPH/pages/2941567805/NewformaLink+Setup+Dev+Environment

### NewformaApi (N-API)
For partner connectors — setup: https://newformaholdings.atlassian.net/wiki/spaces/EPH/pages/2941567821/Newforma+Api+Dev+Environment

## SMTPRelayService (if using project email addressing / PEACE)

- Open Developer Command Prompt for VS 2026 as Administrator and run:
  - `sc create NewformaSMTPRelayService binPath= '<repo>\enterprise-suite\Solutions\SMTPRelayService\bin\Debug\NewformaSMTPRelayService.exe'`

## Other Useful Tools

- **Cruise Control CCTray** — trigger NPC/NAPI/NewformaLink builds: https://newformaholdings.atlassian.net/wiki/spaces/BC/pages/2949483316/CCTray
- **Resharper Ultimate** — https://www.jetbrains.com/dotnet/ (optional: dotCover, dotPeek, dotTrace, dotMemory)
- **Visual Studio extensions (optional)** — Markdown Editor
- **Remote Desktop** — enable if working from home
- **IM** — Slack: [newforma.slack.com](https://newforma.slack.com/)
- **Defect tracker & story board** — Jira: [jira.newforma.com](https://jira.newforma.com/)
- **Documentation WIKI** — Confluence: [newformaholdings.atlassian.net](https://newformaholdings.atlassian.net/wiki/welcome)
- **RedGate Performance Profiler (optional)**
- **Diagnostic tools (optional)** — DebugView and other Sysinternals tools (e.g., Process Explorer)
- **GoToMeeting** — see your manager for a GTM account if running meetings

## Additional Resources
- NPC For Dummies: https://newformaholdings.atlassian.net/wiki/spaces/EPH/pages/2941567317/NPC+For+Dummies
- BIM360 as an EDMS data source: https://newformaholdings.atlassian.net/wiki/spaces/EPH/pages/2941567921/BIM+EDMS+For+Developers
- Graph as default Email access: https://newformaholdings.atlassian.net/wiki/spaces/EPH/pages/2941567532/Graph+Configuration+for+Developers

## Troubleshooting
- **"The type or namespace name 'SafeRecipient' could not be found..."** — reinstall Register Redemption: https://github.com/Newforma/enterprise-technical-documentation/tree/master/Newforma%20Dev%20Environment#register-redemption-which-is-used-by-our-email-code
- **"Cannot open include file: 'mapiguid.h'"** while building MsgAdapter — ensure MsgAdapter → Properties → C/C++ → Additional Include Directories points to the SDK 7.1 include folder. If the include directory doesn't exist at `C:\Program Files (x86)\Microsoft SDKs\Windows\v7.1A`, copy it from `C:\Program Files\Microsoft SDKs\Windows\v7.1`.

---

# 2. Additional Steps Added by Our Team (For Client Confirmation)

These steps and observations were **not part of the original client-provided document**. We're flagging them here for your review and confirmation — please let us know if they're correct as documented or need modification.

1. **VS 2026 download source** — We used the **Dev Machine Setup** shared path (downloading and unzipping the full folder) rather than the Microsoft Learn URL, and ran the installer as Administrator.
2. **Additional VS 2026 component** — Under Windows and Web Development → Universal Windows App Development Tools, we additionally selected **Windows 10 SDK (10.0.10240)** to avoid a build issue with the C++ plugins.
3. **Package-missing prompts during VS install** — Expect this prompt 5–10 times during install; select Skip each time. This was the only error type we encountered.
4. **VS installer config import** — Confirmed workflow: Open the VS Installer → Import Configuration → download the config file from the documented path → import → install.
5. **NUnit 3 install method** — We used **Option 2** (the MSI installer) rather than the `dotnet add package` route.
6. **Git GUI reminders** — The registry key setup and GPG signing steps are easy to miss during the GUI-driven install; we're calling them out as mandatory, not optional.
7. **Cloning via Visual Studio** — If repos are cloned from within Visual Studio (not a standalone git client), the default path is under the Windows user profile. This must be redirected outside the user folder (e.g., `C:\git`) to avoid NuGet restore failures from excessive path length.
8. **MySQL install source** — We installed MySQL from the same SharePoint path used for the Visual Studio download, rather than the network build path documented in the original guide.
9. **Wordbreaker/hash function copy step** — Calling this out specifically as a step that's easy to miss but required.
10. **InstallerUtility command order/behavior** — The last two of the four `InstallerUtility` commands throw a "local disk" error the first time they're run; this is expected. Complete the remaining setup steps, then re-run — they should return no output.
11. **PowerShell profile setup (OneDrive redirection)** — Instead of copying the `.ps1` file directly to `Documents\WindowsPowerShell`, we ran `$PROFILE` to locate the OneDrive-redirected Documents path, created the `WindowsPowerShell` folder there, and pasted the script. We also had to confirm the file wasn't blocked and run:
    ```powershell
    Get-Item "<path>\Microsoft.PowerShell_profile.ps1" -Stream Zone.Identifier
    Unblock-File "<path>\Microsoft.PowerShell_profile.ps1"
    ```
12. **AllPlugins build failure root cause** — Confirmed the AllPlugins build failure (already noted as ignorable in the original doc) is specifically due to a **Moq NuGet package reference error**.
13. **NPC Configuration — AzureAD account workaround** — Since "check names" could not resolve AzureAD account names for the policy group assignment, we created a local group (`NewformaServiceAccounts`), added the account to it, and granted the "Log on as a service"/"Log on as a batch job" rights to that local group instead, followed by `gpupdate /force` and `Setup-Branch`.
14. **Error 1068 scripted fallback** — For development-only, non-domain VMs, we scripted the existing "set services to Local System account" fix via PowerShell (`sc.exe config ... obj= LocalSystem` for each Newforma service, then `Start-NewformaServices`), rather than doing it manually per-service in the Services console.

---

# 3. Open Questions / Clarifications Required from Client

1. **Visual Studio licensing** — The original document doesn't mention VS licensing. When downloading Visual Studio from the documented path, a license key is visible. Does the license key need to be applied to **both** the VS 2015 and VS 2026 installs, or just one?
2. **IIS certificate step blocking NIX** — During "Configure IIS," the second "Next" button in the certificate request wizard is disabled due to certificate unavailability, which blocks the Newforma Info Exchange (NIX) server from running. How should we proceed — is there an internal certificate authority or alternate process we should be using?
3. **AllPlugins build failure acceptance** — Can the client confirm it's acceptable to skip building the AllPlugins solution due to the Moq NuGet package reference error, consistent with the original doc's note that "packages may not restore correctly for the AllPlugins project, but this can be ignored"?
4. **NPC Configuration workaround confirmation** — Can the client confirm our local-group-based workaround (in place of directly adding AzureAD accounts, which "check names" couldn't resolve) is an acceptable substitute, or is there a preferred/supported method for granting these rights to AzureAD accounts?
5. **Repository scope confirmation** — All setup steps in this guide relate to the enterprise-suite and enterprise-api repos. During our assessment, we also worked with additional repos: **outlook-web-addin**, **enterprise-connector**, and **enterprise-erp-integration** (noted as "five repos" involved, though only three additional repo names were captured on our end). Could the client confirm the complete, authoritative list of repos in scope for this environment, including the two not yet named?
