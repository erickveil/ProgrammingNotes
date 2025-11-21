---
title: How to Build a Standalone Windows App With Flutter
layout: note
tags:
  - flutter
  - Windows
---

# Building a Standalone Windows App with Flutter

To build a standalone Windows executable that you can distribute and run without requiring the recipient to build it, follow these steps:

## Prerequisites

1. Make sure you have the Windows development tools installed:
   ```
   flutter config --enable-windows-desktop
   ```

2. Check your Flutter installation supports Windows:
   ```
   flutter doctor
   ```

## Building the Windows App

1. Open a terminal in VS Code (Terminal > New Terminal)

2. Run the build command:
   ```
   flutter build windows --release
   ```

3. This will create a release build in: 
   ```
   G:\src\Winston\winston\build\windows\runner\Release\
   ```

## Creating a Distribution Package

The release folder will contain:
- `winston.exe` (your application)
- Various DLL files
- Data folder

For distribution, you need to include all these files:

1. Create a ZIP file containing the entire Release directory:
   ```
   cd G:\src\Winston\winston\build\windows\runner\
   powershell Compress-Archive -Path Release\* -DestinationPath winston_app.zip
   ```

2. Alternatively, use an installer creation tool like Inno Setup:
   - Download and install Inno Setup from https://jrsoftware.org/isinfo.php
   - Create a new script
   - Set the app name, version, and publisher
   - Add all files from the Release folder
   - Build the installer

## Testing the Standalone App

Before distributing:
1. Copy the Release folder to another computer without Flutter installed
2. Try running the .exe to verify it works as expected
3. Check if all features work correctly

## Notes on Windows Distribution

- Users don't need Flutter or any development environment to run your app
- The app includes all required dependencies
- Make sure you have rights to distribute any third-party assets (like tarot card images)
- Consider adding an app icon by modifying the app_icon.ico file before building

# Single-File Executable for Flutter Windows Apps

Flutter Windows applications cannot be compiled into a true single-file standalone executable without any DLLs or additional files. This is a limitation of how Flutter's Windows desktop support works.

## Why This Isn't Possible

1. **Flutter Engine DLLs**: Flutter requires `flutter_windows.dll` which contains the Flutter engine
2. **ICU Data**: International Components for Unicode (ICU) data files are required
3. **Assets**: Your tarot card images and JSON data need to be in the `data` folder
4. **Windows Runtime**: Additional Windows-specific DLLs may be needed

## What You Can Do Instead

### Option 1: Use an Installer (Recommended)
Create a professional installer using tools like:
- **Inno Setup**: Free, creates a single installer `.exe` that extracts and installs everything
- **NSIS**: Another free installer creation tool
- **Advanced Installer**: Has a free edition

This gives users a single file to download/run, and it handles all the dependencies.

### Option 2: Self-Extracting Archive
Use a tool like **7-Zip** or **WinRAR** to create a self-extracting archive:
```bash
# Using 7-Zip command line
7z a -sfx winston_portable.exe Release\*
```

This creates one `.exe` that automatically extracts to a temp directory when run.

### Option 3: Portable ZIP Distribution
Package everything in a ZIP file with clear instructions:
```
winston_v1.0.zip
├── winston.exe
├── flutter_windows.dll
├── data/
│   └── (assets)
└── README.txt
```

Users extract once and can run from any location.

## Alternative: Consider Electron or Tauri

If single-file distribution is critical for your use case, you might consider:

- **Tauri**: Rust-based framework that can create smaller, more portable executables
- **Electron**: Can be packaged into more compact distributions with tools like `electron-builder`

However, these would require rewriting your Flutter app.

## Best Practice for Your Tarot App

For a Windows distribution of your tarot card app, I'd recommend:

1. **Use Inno Setup** to create a professional installer
2. Include an uninstaller
3. Create desktop shortcuts
4. Register file associations if needed
5. Include version information

This gives users the best experience - they download one installer, run it once, and your app is properly installed on their system.

Would you like me to provide a sample Inno Setup script for your Winston Tarot app?