---
title: Android Build Script
layout: note
tags:
  - android
  - command
  - shell
---

This script builds a Qt application for Android from the command line.

It can be called from Bash on Linux, a Windows MinGW64 git command line, or a Jenkins script.

The paths should be changed to match your local installation.

The version numbers in the config values can be obtained by running a successful local build in Qt Creator, and scraping the values from the compile log.

The QT_ANDROID_KEYSTORE_ values can be obtained from your Qt Creator build settings, where you do the apk signing.

```shell
# Command Line Qt Build for Android
# Erick Veil
# 
# Tested and works, generating a functioning local build of the apk file.
# This file can be installed and run on an android device.
# Can be run on Bash, or from Jenkins as a Compile.sh script
# If run from windows, run it from a git bash window (MinGW64)
# Make sure the paths follow your local system
# You can get version numbers for the config by running a successful build in
# Qt Creator and viewing the logs.


# Outputs commands before executing them.
set -o xtrace

cd /c

# Build Config
export QT_VERSION=6.4.1
export NDK_VERSION=23.1.7779620
export ANDROID_VERSION=33

# How many processor cores do you want to use when making this?
# Set to "" for unlimited
export MAKE_JOBS=8

# Paths— Change to match local environment
export QTDIR=/c/Qt/$QT_VERSION
export ANDROID_NDK_ROOT=/c/Users/ev1020497/AppData/Local/Android/Sdk/ndk/$NDK_VERSION
export NDK_BIN=$ANDROID_NDK_ROOT/prebuilt/windows-x86_64/bin
export ANDROID_SDK_ROOT=/c/Users/erickveil/AppData/Local/Android/Sdk
export BUILD_DIR=/c/MyApp
export CODE_DIR=/c/Users/erickveil/Documents/ControlSpace-Remote/code
export JDK_DIR="/c/Program Files/Android/Android Studio/jbr"

# Signing data— androiddeployqt uses these values:
export QT_ANDROID_KEYSTORE_PATH=/c/Users/erickveil/Desktop/keystore/myapp_android.jks
export QT_ANDROID_KEYSTORE_ALIAS=myapp_android

# You'll want to get the actual passwords to assign here:
export QT_ANDROID_KEYSTORE_STORE_PASS=*****
export QT_ANDROID_KEYSTORE_KEY_PASS=*****

mkdir $BUILD_DIR/
cd $BUILD_DIR

# Clean
$NDK_BIN/make.exe clean -j$MAKE_JOBS

# Qmake
$QTDIR/android_arm64_v8a/bin/qmake.bat $CODE_DIR/MyApp.pro -spec android-clang

# make
$NDK_BIN/make.exe -f $BUILD_DIR/Makefile qmake_all
$NDK_BIN/make.exe -j$MAKE_JOBS

#make install
$NDK_BIN/make.exe INSTALL_ROOT=C:$BUILD_DIR/android-build install && 
cd $BUILD_DIR && 
$NDK_BIN/make.exe INSTALL_ROOT=C:$BUILD_DIR/android-build install

cp $BUILD_DIR/libMyApp_arm64-v8a.so $BUILD_DIR/android-build/libs/arm64-v8a/

# sign
# also add --aab when we are ready to build this as an *.aab file instead of an apk.
$QTDIR/mingw_64/bin/androiddeployqt.exe --input $BUILD_DIR/android-MyApp-deployment-settings.json --output $BUILD_DIR/android-build --android-platform android-$ANDROID_VERSION --jdk $JDK_DIR --verbose --gradle --release --sign

```