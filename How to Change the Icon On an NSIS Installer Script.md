---
title: How to Change the Icon On an NSIS Installer Script
layout: note
tags:
  - installer
  - NSIS
---

We can add values to our script related to defining the icons used for the installer and uninstaller. They use the Modern User Interface (MUI) macros provided by NSIS to set these icons.

Here’s what each line does:

1. `!define MUI_ABORTWARNING`: This macro enables a warning message when the user attempts to abort the installation.
2. `!define MUI_ICON "${NSISDIR}\Contrib\Graphics\Icons\modern-install.ico"`: This macro sets the icon for the installer. `${NSISDIR}` is a predefined variable that points to the NSIS installation directory.
3. `!define MUI_UNICON "${NSISDIR}\Contrib\Graphics\Icons\modern-uninstall.ico"`: This macro sets the icon for the uninstaller.


```nsis
;--------------------------------------------------------------------------
;Interface Settings
;--------------------------------------------------------------------------
!define MUI_ABORTWARNING
!define MUI_ICON "path\to\your\custom-install-icon.ico"
!define MUI_UNICON "path\to\your\custom-uninstall-icon.ico"
```

Replace `path\to\your\custom-install-icon.ico` and `path\to\your\custom-uninstall-icon.ico` with the actual paths to your `.ico` files. This will set your custom icons for the installer and uninstaller.