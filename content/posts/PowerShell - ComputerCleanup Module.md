---
title: "PowerShell - ComputerCleanup Module"
date: 2022-10-21T10:45:05+02:00
tags: ['Powershell', 'Module', 'Disk']
draft: false
---
`IMPORTANT: This module is still a work in progress!`

`Please use at your own risk.`

This PowerShell module is for freeing up disk space on Windows computers.
I frequently use this on some RDS servers, and my local machine.

You can install the ComputerCleanup module directly from the [PowerShell Gallery](https://www.powershellgallery.com/packages/ComputerCleanup/), or my [GitHub](https://github.com/tomskovich/ComputerCleanup)

## Installation

```powershell
# One time setup
Install-Module ComputerCleanup -AllowClobber -Force 
# -Force and -AllowClobber aren't necessary, but they do skip errors in case some appear.

# Import the module.
Import-Module ComputerCleanup -Force

# Get commands in the module
Get-Command -Module ComputerCleanup

# Get help for the module
Get-Help ComputerCleanup -Full

# Updating
Update-Module -Name ComputerCleanup
```

To install the module manually, or if you are using an earlier version of PowerShell that doesn't support Install-Module, simply download the module from GitHub, and copy the ComputerCleanup folder into your Modules folder. 

If you're not sure where your Modules folder is, open up a PowerShell prompt and type `$env:PSModulePath`.

## Functions\Public

- [Invoke-ComputerCleanup](https://github.com/tomskovich/ComputerCleanup/blob/main/Public/Invoke-ComputerCleanup.ps1) 
    - Main controller function to invoke one or multiple cleanup functions included in this module.
- [Clear-BrowserCache](https://github.com/tomskovich/ComputerCleanup/blob/main/Public/Clear-BrowserCache.ps1)
	- Removes browser cache files for all users.
    - Browsers: Microsoft Edge, Internet Explorer, Google Chrome and Firefox.
- [Clear-FontCache](https://github.com/tomskovich/ComputerCleanup/blob/main/Public/Clear-FontCache.ps1)
	- Removes user font cache files located in `C:\Windows\ServiceProfiles\LocalService\AppData\Local\`
- [Clear-SoftwareDistribution](https://github.com/tomskovich/ComputerCleanup/blob/main/Public/Clear-SoftwareDistribution.ps1)
    - Clears the `C:\Windows\SoftwareDistribution\Downloads` folder.
- [Clear-TeamsCache](https://github.com/tomskovich/ComputerCleanup/blob/main/Public/Clear-TeamsCache.ps1) 
    - Removes Microsoft Teams cache files for all users.
- [Invoke-CleanManager](https://github.com/tomskovich/ComputerCleanup/blob/main/Public/Invoke-CleanManager.ps1) 
    - Runs the Windows Disk Cleanup tool with predefined options.
- [Optimize-SystemFiles](https://github.com/tomskovich/ComputerCleanup/blob/main/Public/Optimize-SystemFiles.ps1) 
    - Removes common system-wide temporary files and folders older than `$Days` old.
    - OPTIONAL: Clears Windows Recycle Bin
- [Optimize-UserProfiles](https://github.com/tomskovich/ComputerCleanup/blob/main/Public/Optimize-UserProfiles.ps1) 
    - Removes common temporary files and folders older than `$Days` days old from user profiles.

## Functions\Private

- [Assert-RunAsAdministrator](https://github.com/tomskovich/ComputerCleanup/blob/main/Private/Assert-RunAsAdministrator.ps1) 
    - Verifies if script/function is running with Administrator privileges.
- [Get-DiskSpace](https://github.com/tomskovich/ComputerCleanup/blob/main/Private/Get-DiskSpace.ps1)
    - Gets available disk space. Used for reporting.
- [Get-UserFolders](https://github.com/tomskovich/ComputerCleanup/blob/main/Private/Get-Userfolders.ps1)
    - Gets all user folders, excluding Administrators and Default/Public users
- [Start-Logging](https://github.com/tomskovich/ComputerCleanup/blob/main/Private/Start-Logging.ps1)
    - Logging function. Very basic wrapper around `Start-Transcript`

### Examples/Usage

Generally, the only function/command you'll need is `Invoke-ComputerCleanup`.

I'll give two recommended parameter sets. One for running on a live/in-use environment, and one for outside of working hours.

#### LIVE environment:
``` powershell
Invoke-ComputerCleanup -Days 30 -UserTemp -SystemTemp -CleanManager -SoftwareDistribution -RecycleBin 
```
```txt
	- Runs the Windows Disk Cleanup tool.
	- Removes temp files in User profiles that are older than 30 days old.
	- Removes temp files in System that are older than 30 days old.
	- Cleans up the "C:\Windows\SoftwareDistribution\Download" folder.
	- Clears the Windows Recycle Bin
```
#### Outside of working hours:
- :warning: This parameter set will close running processes! Use with caution.
``` powershell
Invoke-ComputerCleanup -Days 30 -UserTemp -SystemTemp -CleanManager -SoftwareDistribution -BrowserCache -TeamsCache -FontCache -RecycleBin 
```
```txt
	- Runs the Windows Disk Cleanup tool.
	- Removes temp files in User profiles that are older than 30 days old.
	- Removes temp files in System that are older than 30 days old.
	- Cleans up the "C:\Windows\SoftwareDistribution\Download" folder.
	- Clears cache for all browsers
	- Clears Microsoft Teams Cache
	- Clears Windows Font Cache
	- Clears the Windows Recycle Bin
```

You'll be prompted for confirmation at the beginning, and there will be a report at the end.

![Invoke-ComputerCleanup](/computercleanup_example1_start.png#center)
![Invoke-ComputerCleanup](/computercleanup_example1_finish.png#center)

Some parameters will stop some processes, which can be impactful in live environments.
Therefore, I've added warnings for some parameters. Example:

``` powershell
Invoke-ComputerCleanup -Days 30 -BrowserCache -TeamsCache -UserDownloads
```
![paramwarnings](/paramwarnings.png#center)

## Invoke-ComputerCleanup parameters


### -Days (Default: 30)
- Only remove files/folders that are older than $Days old. 
	- This is based on both file CreationTime AND LastWriteTime.

- :warning: This parameter does NOT apply to the following options:
	- `-BrowserCache`
	- `-TeamsCache`
	- `-SoftwareDistribution`
	- `-FontCache`


### -CleanManager
- Runs the Windows Disk Cleanup tool with the following options enabled:
```txt
- Active Setup Temp Folders
- BranchCache
- Device Driver Packages
- Downloaded Program Files
- GameNewsFiles
- GameStatisticsFiles
- GameUpdateFiles
- Memory Dump Files
- Offline Pages Files
- Old ChkDsk Files
- Previous Installations
- Service Pack Cleanup
- Setup Log Files
- System error memory dump files
- System error minidump files
- Temporary Files
- Temporary Setup Files
- Thumbnail Cache
- Update Cleanup
- Upgrade Discarded Files
- Windows Defender
- Windows ESD installation files
- Windows Error Reporting Archive Files
- Windows Error Reporting Queue Files
- Windows Error Reporting System Archive Files
- Windows Error Reporting System Queue Files
- Windows Upgrade Log Files
```


### -UserTemp
- Removes temp files in User profiles that are older than $Days days old. Default locations:
	- `<USERPROFILE>\AppData\Local\Microsoft\Windows\WER`
	- `<USERPROFILE>\AppData\Local\Microsoft\Windows\INetCache`
	- `<USERPROFILE>\AppData\Local\Microsoft\Internet Explorer\Recovery`
	- `<USERPROFILE>\AppData\Local\Microsoft\Terminal Server Client\Cache`
	- `<USERPROFILE>\AppData\Local\CrashDumps`
	- `<USERPROFILE>\AppData\Local\Temp`


### -SystemTemp
- Removes temp files in system that are older than $Days days old. Default locations:
	- `C:\Windows\Temp`
	- `C:\Windows\Logs\CBS`
	- `C:\Windows\Downloaded Program Files`
	- `C:\ProgramData\Microsoft\Windows\WER`


### -SoftwareDistribution
- Cleans the `C:\Windows\SoftwareDistribution\Downloads` folder.


### -FontCache
- Clears user font cache files located in `"C:\Windows\ServiceProfiles\LocalService\AppData\Local"`


### -BrowserCache 
```txt
:warning: Enabling this parameter will stop ALL running browser processes. Running outside of working hours is advised!
```
- Clears browser cache files for all users.
- Browsers: Microsoft Edge, Internet Explorer, Google Chrome and Firefox.


### -TeamsCache
```txt
:warning: Enabling this parameter will stop ALL running Teams processes. Running outside of working hours is advised!
```
- Clears Microsoft Teams cache files for all users.


### -RecycleBin
- Clears Recycle Bin.


I will actively maintain this module, so feel free to leave a comment or feature request in the comments or send in a pull request on GitHub.