# DEV SOFTWARE - NPC Version 202X.X



## Visual Studio 2026 Professional

- Prerequisite: 
  - Install Visual Studio 2015
    - Downloadable from `\\newforma.local\aperus\SQA\Installation Kits\Microsoft\Visual Studio 2015 Pro`
    - Accept the defaults and add these options to install
      - Microsoft Office Developer Tools
      - Visual C++
        - Common Tools for Visual C++ 2015
        - Microsoft Foundation Classes for C++
        - Windows XP Support for C++ (because it includes Windows 7.1 SDK)
  - **NOTE** various errors may be encountered while installing Visual Studio 2015, but most can be safely ignored. As long as installation completes despite the errors, the installation can be considered a success.
  - **NOTE** (Keep in mind for when you clone the repos and open VS 2026) If you encounter errors when opening VS2026 and numerous projects say (incompatible) or issues with loading, then try reloading those specific projects with dependencies. If this does not work, then open the VS 2026 install and choose the "Repair" option under the "More" combo box on the installer main screen. Then, close and reopen VS 2026. You may need to reload those projects with dependencies, but now the error messages should resolve.  

- Install Visual Studio 2026 **Professional** from https://learn.microsoft.com/en-us/visualstudio/releases/2026/release-notes. Check out the screenshots of install selections below. 

- **Note** 
  
  - In order to build C++ projects
    - You will need MSVC in version 140 (versions 141 and 142 WILL NOT WORK), Tools for Visual Studio 2017 for Windows XP in C++, which is needed for C++ projects. Please make sure it is included in the Visual Studio. ( See its inclusion in 'desktop development of C++' screenshot below )
  -  If you had installed tools for compiling C++ in Visual Studio 2026 it's likely that you had version 141, 142 or 143 and when you opened the solution the vcxproj files changed PlatformToolset to v141, v142 or v143. In order for projects to build you must revert all the changes, except the ones to _vars.bat
  - Update the path in scripts\\set\_vstudio\_vars.bat to point VS\_INSTALL\_DIR to your custom install path of Visual Studio if you choose not to go with the default path.
  - Check the path in scripts\\set\_vstudio\_vars.bat for VS\_C\_COMPILER. If the path does not match the path to the file on your machine, update the path to point to your cl.exe file.
  - We recommend using the pre-made Visual Studio installation configuration file ([https://github.com/Newforma/enterprise-technical-documentation/blob/vs_2022_update/Newforma%20Dev%20Environment/newforma.vsconfig](newforma.vsconfig)). This configuration file specifies the necessary workloads and components. When imported into the Visual Studio installer, it will automatically include these components. For instructions, see [Import a configuration using the Visual Studio Installer](https://learn.microsoft.com/en-us/visualstudio/install/import-export-installation-configurations?view=vs-2026#import-a-configuration-using-the-visual-studio-installer).
Alternatively, you can configure the installation manually. Below are images showing the required components for the workloads being installed.
  - Do not click the "Remove out-of-support components" button. It may warn you about this a couple of times, but you can ignore those warnings, it wont hurt to have those in there for now. 
  
  - **Note** 
    - Please choose only one approach: either use the configuration file or set up the installation manually. If you choose the manual configuration, refer to the following images for guidance.
    - If any of the images below fail to load, download this document to find a transcript of each image commented out below it
    - Look at each of the images/transcriptions before proceeding with the install
    - Make sure to sign in with your newforma credentials

<img src="https://github.com/Newforma/enterprise-technical-documentation/assets/169819296/e96b87d7-09bc-4038-ba3f-6c513bb97a35" alt="image" style="zoom:50%;" />
<br>
<!--
Options to select on VS 19 install:
 ### ASP.NET and Web Development
        ##### Included
        - .NET Framework 4.7.2 development tools
        -  ASP.NET and web development prerequisites
        ##### Optional 
        - .NET Aspire SDK
        - .NET Framework 4.8 development tools
        - .NET Framework project and item templates
        - Cloud tools for web development
        - .NET profiling tools
        - Entity Framework 6 tools
        - Live Share
        - .NET Debugging with WSL
        - IntelliCode
        - Windows Communication Foundation
        - .NET Framework 4.6.2-4.7.1 development tools
        - .NET Framework 4.8.1 development tools
-->
<br/><br/>

<img src="https://github.com/Newforma/enterprise-technical-documentation/assets/169819296/cc9ad3b8-2aa6-4a89-b30b-cc52a185b335" alt="image" style="zoom:50%;" />
<br>
<!--
Options to select on VS 19 install:
 ### .NET desktop development
        ##### Included
        - .NET Desktop Development tools
        - .NET Framework 4.7.2 development tools
        -  C# and Visual Basic
        ##### Optional 
        - .NET development tools
        - Development tools for .NET
        - .NET Framework 4.8 development tools
        - Entity Framework 6 tools
        - .NET profiling tools
        - IntelliCode
        - Just-In-Time debugger
        - Live Share
        - ML.NET Model Builder
        - GitHub Copilot
        - Blend for Visual Studio
        - .NET Framework 4.6.2-4.7.1 development tools
        - Windows Communication Foundation
        - SQL Server Express 2019 LocalDB
        - JavaScript diagnostics
        - .NET Framework 4.8.1
-->
<br/><br/>

<img src="https://github.com/Newforma/enterprise-technical-documentation/assets/169819296/2fd637cf-2301-4a5c-9d46-5efac8ebcc1e" alt="image" style="zoom:50%;" />
<br>
<!--
Options to select on VS 19 install:
 ### Desktop development with C++
        ##### Included
        - C++ core desktop features
        ##### Optional 
        - MSVC v143 - VS 2022 C++ x64/x86 build tools
        - C++ ATL for latest v143 build tools (x86 & x64)
        - Security Issue Analysis
        - C++ Build Insights
        - Just-In-Time debugger
        - C++ profiling tools
        - C++ CMake tools for Windows
        - Test Adapter for Boost.Test
        - Test Adapter for Google Test
        - Live Share
        - IntelliCode
        - C++ AddressSanitizer
        - Windows 11 SDK (10.0.22621.0)
        - vcpkg package manager
        - GitHub Copilot
        - C++ MFC for latest v143 build tools (x86 & x64)
        - C++/CLI support for v143 build tools (Latest)
        - JavaScript diagnostics
        - Windows 10 SDK (10.0.19041.0)
        - MSVC v142 - VS 2019 C++ x64/x86 build tools
        - MSVC v141 - VS 2017 C++ x64/x86 build tools
        - MSVC v140 - VS 2015 C++ build tools (v140)
-->
<br/><br/>

<img src="https://github.com/Newforma/enterprise-technical-documentation/assets/169819296/1a175b45-1a11-4f77-8994-1d46fefaf6dd" alt="image" style="zoom:50%;" />
<br>
<!--
Options to select on VS 19 install:
 ### Office/Sharepoint development
        ##### Included
        - Office Developer Tools for Visual Studio
        - .NET Framework 4.7.2 development tools
        - Developer Analystics tools
        ##### Optional 
        - Visual Studio Tools for Office (VSTO)
        - Web Deploy 
        - Intellicode
        - .NET Framework 4.6.2-4.7.1 development tools 
        - .NET Framework 4.8.1 development tools
        - .NET Framework 4.8 development tools
-->
<br/><br/>

<img src="https://github.com/Newforma/enterprise-technical-documentation/assets/169819296/6bed2dd0-c750-4a35-bdfd-bc38d9efee9f" alt="image" style="zoom:50%;" />
<br>
<!--
Options to select on VS 19 install:
 ### Additional Individual components
        - C++ ATL for v141 build tools (x86 & x64)
        - C++ MFC for v141 build tools (x86 & x64)
        - C++ v14.29 (16.11) ATL for v142 build tools (x86 & x64)
        - C++/CLI support for v142 build tools (14.29-16.11)
        - C++ v14.29 (16.11) MFC for v142 build tools (x86 & x64)
-->
<br/><br/>

## NUnit 3 (Automated Testing)

- Install NUnit 3 console (runner)
  - Option 1
    - At command prompt: &gt;`dotnet add package NUnit.Runners --version 3.9.0`
  - Option 2
    - Use **NUnit.Console-3.9.0.msi** from GitHub: [https://github.com/nunit/nunit-console/releases/tag/v3.9](https://github.com/nunit/nunit-console/releases/tag/v3.9)
- Add the install location into the system PATH environment variable.
- When using option #2 the default installation location is C:\\Program Files (x86)\\NUnit.org\\nunit-console

## Source Control

### Install git
- [https://git-scm.com/download/win](https://git-scm.com/download/win)
  - If you want to use GitGui or GitBash as a UI, select it during this install
  - Make sure large file support is checked
  - If you used the Git installer and checked LFS you don't need this step
    - Install git lfs (large file support): https://git-lfs.github.com/        
- In Visual Studio, Tools-&gt;Options-&gt;Source Control-&gt;Plug-in Selection: Set to Microsoft Git provider
### Install a git GUI 
- GitHub Desktop is the recomended GUI.
  - **WARNING**: There are unlicensed branches of GitHub Desktop that may link to malicious sites. Make sure you download the correct branch from the [official website](https://desktop.github.com/download/). If in doubt, reach out to an established developer.
- From your git tool - clone some repositories.   
  - **enterprise-suite** (where Project Center stuff primarily lives)
  - **enterprise-technical-documentation** (where this document lives)
  - **enterprise-tools**
  - **enterprise-api** (contains NewformaAPI which connects third-party connectors to Project Center and NewformaLink which connects web and mobile clients to Project Center)

- Link command line tools to git repo
  - Using RegEdit, create new key `HKEY_CLASSES_ROOT\Newforma Installation` and add a new `String Value` named `DeveloperRepositoryPath` and set the value to the root folder where you cloned the enterprise-suite repository to (e.g., c:\\git\\enterprise-suite)
- Set up GPG signing using the directions here: https://newformaholdings.atlassian.net/wiki/spaces/EPH/pages/2941562227/Signing+GitHub+commits+with+GPG

- **NOTE** 
  - All repos after build need a lot of disk space. Make sure you have at least 100GB on the hard drive.
    - As of July 2024, this is closer to 200GB.
  - Please store all repos at `C:\git`, outside of your user area, in order to avoid access/permission issues among processes during development. 


## MS Office

Office/Outlook IT may have already installed this on your system. If they did it is probably the 64 bit version.

NPC must run in the same bitness as Outlook. If you want to run NPC in 32 bit mode you will need to change the bitness of your Office installation. Either way you need to uninstall and reinstall Office using the custom install option. There are some settings (can someone add the actual settings) called .Net ??? that should be turned on already. The weird thing is if you use the standard install these don't seem to get installed.

Note - most Developers use 64-bit Office (as of June 2025, developers have run into issues with 32-bit setups on their dev machines, particularly around Info Exchange, so some troubleshooting/bugfixes may be required to get it working properly).

You can determine what version you are running by starting task manager and look under processes. If the process name  has '\*32' in the name it's a 32-bit program.

- Indicate which version of office you are using through environment variable:
  - Set `OFFICE64=1` env variable if running 64-bit Office

**NOTE** Be aware that setting the environment variables above typically cause conditional references to be brought into project builds.  If you're seeing a strange assembly missing error then look into the failing project for these variables causing reference issues.

### Register Redemption which is used by our email code
- Run <br>*&lt;repo&gt;\\enterprise-suite\\Solutions\\ThirdParty\\Redemption\\install.exe* 
<br>and just point its install folder to<br> *&lt;repo&gt;\\enterprise-suite\\Solutions\\ThirdParty\\Redemption*

## MySQL

Install the currently supported version of MySQL 
- Note: if a version of MySQL is already installed on the machine, follow these instructions for upgrading: [https://newformaholdings.atlassian.net/wiki/spaces/FAT/pages/4703158278/Upgrade+to+2026.2+with+MySQL+8.4.8](https://newformaholdings.atlassian.net/wiki/spaces/FAT/pages/4703158278/Upgrade+to+2026.2+with+MySQL+8.4.8)
- Every build of the develop branch includes the MySQL installer (.msi file) for the currently supported version of MySQL in the MySQLMigrator\MySQL directory.
- To install the currently supported version of the MySQL installer, go to *\\\\newforma.local\\aperus\\QAData\\Builds\\Project_Center\\develop\\[currentNpcVersion]\\[latestCompletedBuildNumber]\\MySQLMigrator\\MySQL*
  - For example, for 2026.2, the installer can be found at *\\\\newforma.local\\aperus\\QAData\\Builds\\Project_Center\\develop\\2026.2.0\\39517\\MySQLMigrator\\MySQL*
- Under Setup Type, choose Typical 
  - Make sure to use `root` for the password
  - Make sure that the Windows Service Name is: `MySQL`
- Wordbreaker/newforma hash function
  - Copy *&lt;repo&gt;\\enterprise-suite\\Solutions\\MySqlWorkbreaker\\release\\x64\\\*.{dll,pdb}* to *C:\\Program Files\\MySQL\\MySQL Server 8.4\\lib\\plugin*

  **NOTE** If using the community installer for an older version of MySQL, and if you are encountering the MySQL installation error "unknown variable 'mysqlx_port=0.0‘", the simplest fix will be to follow the steps below (thanks to Eric Buterbaugh for providing these):
1. Cancel the MySQL installation
2. Launch MySQL Community installer and uninstall all components, including the installer
3. Confirm the ..\programdata\mysql folder is deleted
4. Launch the MySQL Community installer and upgrade the catalog. If this is not possible, first install MySQL Workbench, then launch the Catalog update again (the option to update the catalog may not appear unless at least one application is installed, such as Workbench).
5. Go to Start > MySQL > MySQL Community installer and install the correct version of MySQL Server


## IIS Installation

To run Newforma Info Exchange (NIX) you will need the following settings

- Install IIS
  - Run 'Turn Windows features on or off'
  - Expand Internet Information Services
    - Expand Web Management Tools, select the following:
      - Expand IIS 6 Management Compatibility, select the following:
        - IIS 6 Management Console
        - IIS Metabase and IIS 6 configuration compatibility
      - IIS Management Console
      - IIS Management Scripts and Tools
      - IIS Management Service
    - Expand World Wide Web Services, select the following:
      - Expand Application Development Features, select the following:
        - .Net Extensibility 4.8
        - ASP.NET 4.8 (will automatically select others when you check it)
        - CGI
        - ISAPI Extensions
        - ISAPI Filters
      - Expand Common Http Features, select the following:
        - Default Document
        - Directory Browsing
        - HTTP Errors
        - HTTP Redirection
        - Static Content
      - Health and Diagnostics (select all options)
      - Expand Performance Features, select the following:
        - Static Content Compression
      - Expand Security, select the following:
        - IP Security
        - Request Filtering
        - Windows Authentication

## Configure IIS

- Run: MMC (Microsoft Management Console)
  - Go to file -> Add/Remove Snap-in...
    - Select Certificates on the left, click Add
      - In the first screen, select Computer account
      - Leave default settings on the next screen (Local computer should be selected) and click Finish 
    - Click Ok
  - Under Certificates -> Personal -> Certificates on the left, right-click and select All Tasks -> Request New Certificate...
  - Select Next twice, with default settings
  - Check Web Server, then select the warning text "More information is required..."
  - Start under the Subject tab
    - Under Type, choose Full DN, then copy/paste one string with the following format into the text box, replacing [devMachineName] with the short name of your dev machine:
      - CN=[devMachineName].newforma.local, O=Newforma, OU=Dev, L=Manchester, ST=NH, C=US
    - Click Add
    - Under Alternative name, choose DNS and enter a string with the following format into the text box:
      - [devMachineName]
    - Click Add
    - Now enter a string with this format into the same box:
      - [devMachineName].newforma.local
    - Click Add
  - Click on the General tab
    - Under Friendly name enter: [devMachineName]
  - Click Ok
  - Click Enroll
  - After receiving a success message, click Finish
- Run: IIS (Internet Information Services (IIS) Manager)
  - Right click on Default Web Site choose 'Edit Bindings...'
    - Add a binding for `https` and bind to certificate above
- **NOTE**: Since the certificate uses both short name and fully qualified domain name as the alternative names, under RegEdit, HKLM\\software\\Wow6432Node\\Newforma\\&lt;edition&gt;\\DeveloperProvisionNixName can be set to either short or long name.
- **NOTE**: This is not required in order to build and run NPCS, so if you have problems with online certificate authority, you can skip this step for now.
- **NOTE**: From the root server node, IIS -> Authentication Only Anonymous Authentication should be Enabled. The others MUST be set to Disabled.
- **NOTE**: If you have an existing cert that is expired, make sure you are binding to the new certificate (if the old cert has the same name, it can be manually deleted from IIS -> Server -> Certificates)


## Nuget

- Copy the latest NuGet.exe (https://www.nuget.org/downloads) and put it in a folder that is in your path (e.g., C:\\windows). Verify that the destination is in your path by checking the "path" setting in your "system environment variables."
- Open Visual Studio 2026
  - Tools > NuGet Package Manager > Package Manager Settings > General  
    - Set the 'Default package management format' to 'PackageReference'
  - Tools > NuGet Package Manager > Package Manager Settings > Package Sources
    - Add a new package source:
      - Name: `Newforma`
      - Source: `https://nuget.newforma.com/api/v2/`
    - Add a new package source:
      - **NOTE** This package must be added with package manager console in visual studio, with command "NuGet Sources Add -Name "github" -Source "https://nuget.pkg.github.com/Newforma/index.json" -UserName "<your login email>" -Password "<your password>"
      - Name: `github`
      - Source: `https://nuget.pkg.github.com/Newforma/index.json`
      - Username: Your github.com username
      - Password: A personal access token generated from github.com with the following scopes:
        -repo
        -write:packages
        -read:packages
    - Add a new package source:
      - Name: `Nuget.org`
      - Source: `https://api.nuget.org/v3/index.json`

## Build and Run NPC InstallerUtility

- Install the 2010 Tools for Office runtime; get the file from here
  - [How to: Install the Visual Studio Tools for Office runtime redistributable](http://msdn.microsoft.com/en-us/library/ms178739.aspx)
  - [Visual Studio 2010 Tools for Office Runtime Download](https://www.microsoft.com/en-us/download/details.aspx?id=105522) NOTE: The correct link changes from time to time, if this one is broken try googling "Visual Studio 2010 Tools for Office" and it should appear
  .
- Using Visual Studio 2026, open &lt;repo&gt;\\enterprise-suite\\Solutions\\All.sln –
  - It may ask you if you want to install the Visual Studio Tools for Office – say yes and install
  - Build InstallerUtility project
- Open a **command prompt as an administrator** to &lt;repo&gt;\\enterprise-suite\\Solutions\\enterprise-core\\InstallerUtility\\bin\\Debug
  - Run `InstallerUtility SaveDatabaseCredentials root localhost 3306`
  - Run `InstallerUtility ConfigureMySqlIni`
  - Run `InstallerUtility ConfigurePerformanceCounters NPCS`
  - Run `InstallerUtility ConfigurePerformanceCounters NWS`

## Setup PowerShell

- Copy<br> *&lt;repo&gt;\\enterprise-technical-documentation\\Newforma Dev Environment\\Microsoft.PowerShell\_profile.ps1*<br>to <br>*C:\\Users\\[username]\\Documents\\WindowsPowerShell*<br>
   **NOTE** If you have an existing powershell profile, merge contents of existing file into Microsoft.PowerShell_profile.ps1
- Ensure read-only property is not checked on Microsoft.PowerShell\_profile.ps1
- In *Microsoft.PowerShell_profile.ps1* change variable NewformaServicesPS1 to point at *&lt;repo&gt;\\enterprise-suite\\Solutions\\scripts\\NewformaServices.ps1*
- Start PowerShell as Administrator
  - Run `Set-ExecutionPolicy RemoteSigned`, and answer Y at the prompt
  - If asked "Which branch do you want to use?", choose the Mainline option (you may need to close and re-open PowerShell to receive the prompt)
  - Ignore errors like 'n4 was not found' as these tools have not been built yet  
- **NOTE** Unless you change the execution policy, it's likely you won't be able to run the script

## Import the certificate used to sign the outlook add-in

- Double-click &lt;repo&gt;\\enterprise-suite\\Solutions\\OutlookAddIn2013\\Newforma--Inc-Newforma-Code-Signing.pfx
- While going through the wizard:
  - Import for the current user
  - Check the box to 'Mark this key as exportable'
  - Password: `Millyard`

## ProjectWise
- Changes as of v12.5.1
  - it is no longer necessary to have PW installed on you machine to perform builds as the SDK has been moved to  enterprise-suite\Solutions\ThirdParty\ProjectWise\SDK\10.00.03.262
  - If you want to test the integration then you need ProjectWise Connection Client v01.00.03 or greater installed. 
    - \\newforma.local\data\departments\Development\Installation Kits\Bentley ProjectWise\ProjectWise Explorer CONNECT Edition
        - The installer starting with `pwclt` includes both Connection Client AND ProjectWise Explorer, which is required to use the EDMS plugin.

-Pre v12.5.1     
- To BUILD the ProjectWise 8.x/10.x EDMS plugin
  - Install the ProjectWise SDK from *\\\\newforma.local\\data\\departments\\Development\\Installation Kits\\Bentley ProjectWise\\PW 10\\projectwisesdk10000167en.exe*
  - Copy the registry key: *HKEY\_LOCAL\_MACHINE\\SOFTWARE\\WOW6432Node\\Bentley\\ProjectWise SDK\\10.00* into *HKEY\_LOCAL\_MACHINE\\SOFTWARE\\WOW6432Node\\Bentley\\ProjectWise SDK\\08.11*
- To USE the ProjectWise 10.x EDMS plugin (optional)
  - Install ProjectWise Explorer from *\\\\newforma.local\\data\\departments\\Development\\Installation Kits\\Bentley ProjectWise\\PW 10\\pwclt10000167en.exe*



## Build All Projects

**IMPORTANT** – when building in Visual Studio, ensure that the build is set to `x64` when running 64bit Office, otherwise use x86
**IMPORTANT** – when building in Visual Studio, ensure that you run VS as an administrator


- Run PowerShell as an administrator
  - Run `Build-Newforma`, answering Y to the build prompts
  - You do not need to run all of the unit tests
  - Packages may not restore correctly for the AllPlugins project, but this can be ignored
  

*Troubleshooting Tip*

*The build creates a hidden .vs folder within &lt;repo&gt;\\enterprise-suite\\Solutions\\ - If the build fails, removing the contents of the .vs folder may be required to successfully build.*

- **NOTE** If CPP projects are not building it's likely that you are using a newer version of compilling tools. Please make sure that you have version 140.
- **NOTE** If after build Newforma Project Center Service is not running it's likely that there is an issue with MySQL. Try to debug ProjectCenterServer project. 
- **NOTE** If Newforma Project Center Service keeps stoping it's likely that there is an issue with MySQL. Try to debug ProjectCenterServer project. 
- **NOTE** If you have problems building project make sure you have enough space on you hard drive - at least 100GB
- **NOTE** If you have problems building project try building them step by step. Starting with only C++ projects, then Plugins and then All
- **NOTE** If build fails and you get empty logs it means that VS folder is not set up correctly in enterprise-suite\\Solutions\\scripts\\set\_vstudio\_vars.bat 
- **NOTE** If build fails because of SQL Migration remove postbuild from MySQLMigrator solution
- **NOTE** If DownloadWeb, RemoteWeb, or UserWeb appears as "unloaded" in Visual Studio, delete csproj.user file in those directories (can happen on clean dev machines)
- **NOTE** If the All solution fails to build with errors related to Leadtools version conflicts (generally these will be errors related to duplicate Leadtools files of two different versions) then run "git clean -xdf" from the command line in the enterprise-suite repo and build again.

## NPC Configuration

- Add your user account to policy groups
  - Windows 10 & 11: Run 'Local Security Policy'
  - Local Policies
    - User Rights Assignment
      - Log on as a batch job
      - Log on as a service
- Setup Mainline 'branch'
  - Restart PowerShell as Admin so it can initialize properly after Build-Newforma
    - Run `Setup-Branch newforma\[username] [password]`
    - **NOTE** If your password contains any special characters be sure to escape them to ensure that Setup-Branch runs properly ie: "pas$word" -> "pas`$word"
    - **NOTE** If you see any such "Cannot invoke method on null valued expression", complete all steps from here to "Reset the Newforma databases", then close and restart powershell as admin
- Create a share of *&lt;repo&gt;\\enterprise-suite\\Solutions\\XSLTemplates* directory as `Templates`
  (Right click on XSLTemplates => Properties => Sharing => Advanced Sharing => Give share name as `Templates`)
- In RegEdit, Create (or update) String Value HKEY_LOCAL_MACHINE(HKLM)\\SOFTWARE\\Wow6432Node\\Newforma\\[Current Edition]\\SendMail\\SmtpServer and set to `Belichick`
- In RegEdit, Create (or update) String Value HKEY_LOCAL_MACHINE(HKLM)\\SOFTWARE\\Wow6432Node\\Newforma\\[Current Edition]\\SendMail\\FromAddress and set to your email address
- Import into the registry *&lt;repo&gt;\\enterprise-suite\\Solutions\\ProjectCenterServer\\ProjectCenterServer.reg*
- From RegEdit, navigate to HKEY_LOCAL_MACHINE(HKLM)\\SOFTWARE\\Wow6432Node\\Newforma, right-click Permissions, add full control for the NETWORK SERVICE account and 'Authenticated Users' 
- From Explorer, add full control permission for the NETWORK SERVICE account and 'Authenticated Users' to &lt;repo&gt;\\enterprise-suite\\Solutions\\InfoExchangeServer (You may get Access Denied errors from bin and obj folders, and various JavaScript files)
- Windows 10 - Windows Defender Firewall > Advanced Settings
  - Inbound Rule - New Rule – Wizard
    - Rule Type: Port
    - Protocol and Ports: Specific Local Ports: 9002,9003,9004
    - Action: Allow the connection
    - Profile: Domain, Private, Public
    - Name: NPC
- **NOTE** If your computer is not in newforma domain you need to run Setup-Branch for your domain or as a local pc for example `Setup-Branch "MyPc\MyLogin" "Password"`
- **NOTE** If you are setting up the development environment on a non-developer computer and the services refuse to start (`Error 1068: The dependency service or group failed to start.`), you may need to go to Services, then in the properties of each Newforma service, in the Log On tab, change the selection to "Local System account".

## Reset the Newforma databases

- From PowerShell run
  - `Reset-Newforma`
  - `Invoke-N4 npcs verifyinstallation`
  - `Invoke-N4 nix verifyinstallation`

# Run the Newforma NPC Client
- From PowerShell run
  - `Invoke-NPC`

# Run the Newforma NPCS
- From PowerShell run
  - `Start-NewformaServices`
  
# Newforma PDF Printer
It is important to install the PDF Printer to have a print option with NewForma PDF driver (in Adobe Reader or Other)
The installation folder is in https://github.com/Newforma/enterprise-suite/blob/develop/Setup/Common/Prerequisites/Newforma%20PDF/NewformaPDFSetup.exe

# Newforma Web Services APIs (NL and N-API)

- Add Newforma nuget server to Nuget package mananger:
  - [https://nuget.newforma.com/api/v2/](https://nuget.newforma.com/api/v2/)
- Add github nuget server to Nuget package mananger:
  - [https://nuget.pkg.github.com/Newforma/index.json](https://nuget.pkg.github.com/Newforma/index.json)
  - NOTE: Your password for the github nuget source must be a personal access token generated in github.com with the following scopes:
    -repo
    -write:packages
    -read:packages
  
- Open IIS Manager
  - Click the server name on Connections pane in IIS manager
  - In the Management section, click **Feature Delegation**
  - Click **Authentication - Anonymous**, then click **Read/Write** on the right Actions pane
  - Click **Authentication - Windows**, then click **Read/Write** on the right Actions pane

## NewformaLink (NL)
For web and mobile client applications
- Setup the NewformaLink web app:
  -https://newformaholdings.atlassian.net/wiki/spaces/EPH/pages/2941567805/NewformaLink+Setup+Dev+Environment

## NewformaApi (N-API)
For partner connectors
- Setup the NewformaApi web app:
  - https://newformaholdings.atlassian.net/wiki/spaces/EPH/pages/2941567821/Newforma+Api+Dev+Environment

# SMTPRelayService, if you are using project email addressing (PEACE)

- Open up Developer Command Prompt for VS 2026 as Administrator and type
  - sc create NewformaSMTPRelayService binPath= '*&lt;repo&gt;\\enterprise-suite\\Solutions\\SMTPRelayService\\bin\\Debug\\NewformaSMTPRelayService.exe'

# Other useful tools

## Cruise Control CCTray
- CCTray allow you to trigger NPC/NAPI/NewformaLink builds
- https://newformaholdings.atlassian.net/wiki/spaces/BC/pages/2949483316/CCTray

## Resharper Ultimate
- Download https://www.jetbrains.com/dotnet/
- Optional components
  - dotCover
  - dotPeek
  - dotTrace
  - dotMemory
## Visual Studio extensions (optional)
- Markdown Editor
## Remote Desktop 
- Enable if you want to be able to work from home
## IM 
- Slack [newforma.slack.com](https://newforma.slack.com/)
## Defect tracker & story board 
- Jira [jira.newforma.com](https://jira.newforma.com/)
## Documentation WIKI 
- Confluence [newformaholdings.atlassian.net](https://newformaholdings.atlassian.net/wiki/welcome)
## RedGate Performance Profiler (optional)
## Diagnostic tools (optional)
- DebugView from http://technet.microsoft.com/en-us/sysinternals/bb896647.aspx 
- There are other good tools too you might want to look at here: http://technet.microsoft.com/en-US/sysinternals (take a long look at Process Explorer a task manager alternative)
## GoToMeeting
- if you want to run meetings, see your manager to get a GTM account

## Additional resources
- Lots of good articles in here
  - https://newformaholdings.atlassian.net/wiki/spaces/EPH/pages/2941567317/NPC+For+Dummies
- Consider setting up BIM360 as an EDMS data source for a secondary project folder
  - https://newformaholdings.atlassian.net/wiki/spaces/EPH/pages/2941567921/BIM+EDMS+For+Developers
- Consider setting up Graph as your default Email access
  - https://newformaholdings.atlassian.net/wiki/spaces/EPH/pages/2941567532/Graph+Configuration+for+Developers

## Troubleshooting
- The type or namespace name 'SafeRecipient' could not be found (are you missing a using directive or an assembly reference?)

  Please reinstall https://github.com/Newforma/enterprise-technical-documentation/tree/master/Newforma%20Dev%20Environment#register-redemption-which-is-used-by-our-email-code

- Cannot open include file: 'mapiguid.h': No such file or directory?, while building MsgAdapter
 
  Please make sure MsgAdapter -> Properties -> C/C++ -> Additional Include Directories points correctly to the SDK's 7.1 include folder

  If include directory doesn't exist in C:\Program Files (x86)\Microsoft SDKs\Windows\v7.1A, copy it from C:\Program Files\Microsoft SDKs\Windows\v7.1
