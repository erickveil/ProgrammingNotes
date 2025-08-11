---
title: How To Set the Team in an iOS Qt Project
layout: note
tags:
 - iOS
 - Qt
---

## Problem:

I'm on multiple development Teams as an App Store Developer. When I'm working on a Qt Project with iOS as a target, the default team it signs with is whatever team it pulls up first. This can cause my project to build for the wrong team, creating conflicts when I want to test on devices.

## Solution:

This is a Qt Creator Build Kit issue. It's not something you set in your *.pro file. 

A CMake project may be different, and you might also get away with using a specific PList file if you're already going that route, but I haven't explored those options yet, as it's not part of my own workflow.

To fix this issue in Qt Creator, click on the **Projects** button on the left icon menu bar. Make sure your iOS project is selected in **Active Projects** in the left sidebar, then scroll down to the **Build & Run** section and find your kit for **Qt (version number) for iOS** and select that. 

In the main **Build Settings** window, scroll down to the **iOS Settings** section. There you get a dropdown to select the **Development Team** just like you would in XCode. You may have to press the **Reset** button. 

Select your team and check **Automatically manage signing**. Your build should now be created in the apprpriate team.

## Verifying:

If you build your project, you get an `*.xcodeproj` file in your build directory. You can open that in XCode. In the left pane, click the folder icon (**Show the project navigator** icon) and then click on the root level name of your project in the folder/class tree that appears below.

The **Team** value should match the team you set in Qt Creator. 

If you do this step before the fix, you can keep XCode open as you build from Qt Creator with the correct team, and see the Team name automatically update in XCode.

