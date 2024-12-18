# Guide: Using Winget-Install with Intune

This guide explains how to use the `winget-install.ps1` and `winget-detect.ps1` scripts to create and deploy Intune packages for application management using Winget.

---

## Overview

These scripts enable administrators to deploy and manage applications via Intune (or SCCM) using Microsoft's **Winget** package manager. You can:

1. Install applications.
2. Detect installed applications.
3. Customize the installation process.
4. Uninstall applications (with limitations).

---

## Preparing for Deployment

### Prerequisites

- **Winget** must be installed on target devices.
- Ensure the scripts `winget-install.ps1` and `winget-detect.ps1` are accessible.
- Use a 64-bit version of PowerShell during deployment to avoid compatibility issues.

### Files Required

1. `winget-install.ps1` (main script for installations).
2. `winget-detect.ps1` (script for detection methods).

---

## Creating an Intune Package

### 1. Package the Scripts with IntuneWinAppUtil

Use the [IntuneWinAppUtil](https://learn.microsoft.com/en-us/mem/intune/apps/apps-win32-app-management#prepare-the-win32-app-content) tool to package the `winget-install.ps1` script into a `.intunewin` file.

1. Place `winget-install.ps1` in a folder (e.g., `Winget-Scripts`).
2. Run the following command:
   ```powershell
   IntuneWinAppUtil.exe -c "Path\To\Winget-Scripts" -s winget-install.ps1 -o "Path\To\Output"
   ```

This creates an `.intunewin` file ready for upload.

---

### 2. Configure the Intune Application

1. Go to **Microsoft Intune Admin Center** > **Apps** > **Add** > **Win32 app**.
2. Upload the `.intunewin` file created earlier.
3. Configure the **Program** tab:
   - **Install Command**:  
     ```powershell
     "%systemroot%\sysnative\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -ExecutionPolicy Bypass -File winget-install.ps1 -AppIDs <AppID>
     ```
     Replace `<AppID>` with the Winget App ID of the desired application (e.g., `Notepad++.Notepad++`).
   - **Uninstall Command**:  
     ```powershell
     "%systemroot%\sysnative\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -ExecutionPolicy Bypass -File winget-install.ps1 -AppIDs <AppID> -Uninstall
     ```

4. Configure the **Detection Rules** tab:
   - Use the `winget-detect.ps1` script.
   - Set `$AppToDetect` in `winget-detect.ps1` to the App ID (e.g., `Notepad++.Notepad++`).
   - Upload the updated detection script to Intune.

5. Set any other configurations as needed and assign the application to required devices/groups.

---

## Script Details

### Installing Applications

Use the following command to install applications:
```powershell
powershell.exe -ExecutionPolicy bypass -File winget-install.ps1 -AppIDs "<AppID>"
```

- Multiple applications can be installed by separating App IDs with commas:
  ```powershell
  powershell.exe -ExecutionPolicy bypass -File winget-install.ps1 -AppIDs "7zip.7zip, Notepad++.Notepad++"
  ```

### Using Native Winget Parameters

Custom Winget parameters can be passed using the `AppIDs` argument:
```powershell
powershell.exe -ExecutionPolicy Bypass -File winget-install.ps1 -AppIDs "Citrix.Workspace --override \"/silent /noreboot /includeSSON /forceinstall\""
```

---

### Uninstalling Applications

Uninstall an application using:
```powershell
powershell.exe -ExecutionPolicy bypass -File winget-install.ps1 -AppIDs <AppID> -Uninstall
```

**Note**: Silent uninstalls may not always work. Use the application’s original uninstall command for better results.

---

## Mods (Customization)

The script supports pre/post-install and uninstall hooks. Add custom scripts to the **Mods** directory with the following naming conventions:

- **Before Install**: `<AppID>-preinstall.ps1`
- **During Install**: `<AppID>-install.ps1`
- **After Install Once**: `<AppID>-installed-once.ps1`
- **After Install**: `<AppID>-installed.ps1`
- **Before Uninstall**: `<AppID>-preuninstall.ps1`
- **During Uninstall**: `<AppID>-uninstall.ps1`
- **After Uninstall**: `<AppID>-uninstalled.ps1`

Example: To run a custom script during the installation of `Notepad++`, create `Notepad++.Notepad++-install.ps1` and place it in the **Mods** directory.

---

## Detection Methods

The `winget-detect.ps1` script checks for installed applications based on their App ID. Update the `$AppToDetect` variable with the relevant App ID:
```powershell
$AppToDetect = "Notepad++.Notepad++"
```

Use this script in SCCM or Intune as the detection method.

---

## Additional Resources

- [Winget Documentation](https://learn.microsoft.com/en-us/windows/package-manager/winget/)
- [Winget-AutoUpdate (WAU)](https://github.com/Romanitho/Winget-AutoUpdate)
- [Winget-Intune-Win32](https://github.com/o-l-a-v/winget-intune-win32)

---

# Winget-Install (Will be part of WAU - Work in progress)
Powershell scripts to install Winget Packages with SCCM/Intune (or similar) or even as standalone in system context (Inspired by [o-l-a-v](https://github.com/o-l-a-v) work)

## Install
### SCCM
- Create an application and put the "winget-install.ps1" script as sources
- For install command, put this command line:  
`powershell.exe -ExecutionPolicy bypass -File winget-install.ps1 -AppIDs Notepad++.Notepad++`

![image](https://user-images.githubusercontent.com/96626929/152222570-da527307-ecc9-4fc2-b83e-7891ffae36ee.png)

### Intune
- Create Intunewin with the "winget-install.ps1" script
- Create a Win32 application in Intune
- Put this command line as Install Cmd (Must call 64 bits powershell in order to work):  
`"%systemroot%\sysnative\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -ExecutionPolicy Bypass -File winget-install.ps1 -AppIDs Notepad++.Notepad++`

> You can also use **WingetIntunePackager** tool to package apps from a GUI: https://github.com/Romanitho/WingetIntunePackager

### Use Winget native parameters
You can add custom parameter in your `AppIDs` argument. Don't forget to escape the quote:  
`powershell.exe -Executionpolicy Bypass -File winget-install.ps1 -AppIDs "Citrix.Workspace --override \"/silent /noreboot /includeSSON /forceinstall\""`  
Details: https://github.com/Romanitho/Winget-Install/discussions/20

### Multiple apps at once
- Run this command  
`powershell.exe -Executionpolicy Bypass -Command .\winget-install.ps1 -AppIDs "7zip.7zip, Notepad++.Notepad++"`

## Detection method
- Use the "winget-detect.ps1" with SCCM or Intune as detection method.
- Replace "$AppToDetect" value by your App ID  
`$AppToDetect = "Notepad++.Notepad++"`

## Updates
https://github.com/Romanitho/Winget-autoupdate

## Uninstall
- To uninstall an app, you can use:  
`powershell.exe -ExecutionPolicy bypass -File winget-install.ps1 -AppIDs Notepad++.Notepad++ -Uninstall`

but most of the time, winget does not manage silent uninstall correcty.
- I would suggest to use the original application uninstaller method, something like this:  
`C:\Program Files\Notepad++\uninstall.exe /S`

## Custom (Mods)

The **Mods feature** allows you to run additional scripts before/when/after installing or uninstalling an app.  
Just put the script with the **AppID** followed by the suffix to be considered in the **Mods directory**:  
**AppID**`-preinstall.ps1`, `-install.ps1`, `-installed-once.ps1`, `-installed.ps1`, `-preuninstall.ps1`, `-uninstall.ps1` or `-uninstalled.ps1`  

> Example:  
> Runs before install: `AppID-preinstall.ps1`  
> Runs during install (before install check): `AppID-install.ps1`  
> Runs after install has been confirmed (one time): `AppID-installed-once.ps1`  
> Runs after install has been confirmed: `AppID-installed.ps1`  
> Runs before uninstall: `AppID-preuninstall.ps1`  
> Runs during uninstall (before uninstall check): `AppID-uninstall.ps1`  
> Runs after uninstall has been confirmed: `AppID-uninstalled.ps1`  

If you're using [**WAU** (Winget-AutoUpdate)](https://github.com/Romanitho/Winget-AutoUpdate) they get copied to the **WAU mods** directory (except `-installed-once.ps1/-preuninstall.ps1/-uninstall.ps1/-uninstalled.ps1`) and also runs when upgrading apps in **WAU**.

`AppID-installed-once.ps1` runs instead of `AppID-installed.ps1` from **Winget-Install**.  

They are deleted from **WAU** on an uninstall (if deployed from **Winget-Install**)

## Other ideas and approaches
https://github.com/o-l-a-v/winget-intune-win32
