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

