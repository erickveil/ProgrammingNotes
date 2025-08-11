---
title: How to Gate Code For OS in Cpp and Qt
layout: note
tags:
  - Cpp
  - Qt6
  - QML
---

# C++

Qt provides its own set of macros to help with platform-specific code. These macros are defined in the QtCore module and can be used to determine the target platform.

Here are some of the most commonly used Qt platform-specific macros:

1. **Windows**:
   - `Q_OS_WIN`: Defined for any version of Windows (includes both 32-bit and 64-bit).

2. **macOS**:
   - `Q_OS_MAC`: Defined for macOS and OS X.

3. **Linux**:
   - `Q_OS_LINUX`: Defined for Linux systems.

4. **Unix and POSIX**:
   - `Q_OS_UNIX`: Defined for Unix and POSIX-compliant systems (includes macOS, Linux, and other Unix-like systems).

5. **FreeBSD**:
   - `Q_OS_FREEBSD`: Defined for FreeBSD.

6. **OpenBSD**:
   - `Q_OS_OPENBSD`: Defined for OpenBSD.

7. **NetBSD**:
   - `Q_OS_NETBSD`: Defined for NetBSD.

8. **Solaris**:
   - `Q_OS_SOLARIS`: Defined for Solaris.

These macros are part of the QtGlobal header, which is included in most Qt projects by default. Here's an example of how to use these macros in your code:

```cpp
#include <QtGlobal>

#if defined(Q_OS_WIN)
// Windows-specific code
#include <windows.h>

void platformSpecificFunction() {
    // Windows-specific implementation
}
#elif defined(Q_OS_MAC)
// macOS-specific code
void platformSpecificFunction() {
    // macOS-specific implementation
}
#elif defined(Q_OS_LINUX)
// Linux-specific code
void platformSpecificFunction() {
    // Linux-specific implementation
}
#else
// Code for other platforms or a default implementation
void platformSpecificFunction() {
    // Default implementation
}
#endif
```

To find the official documentation for these macros, you can refer to the Qt documentation:

- **Qt Global Macros**: [Qt Documentation on Global Macros](https://doc.qt.io/qt-5/qtglobal.html)

This documentation provides a complete list of all the global macros defined by Qt, including those used for platform-specific code.

# QML

In QML, you can use the `Qt.platform.os` property to determine the platform at runtime and conditionally include sections of your QML code. 

The best thing to do is conditionally set the visibility of the QML Item:

```qml
visible: (Qt.platform.os === "osx" || Qt.platform.os === "windows")
```


## Only Load QML on Platform
For marking a section of code to be executed only on Android, you can use a `Loader` or similar conditional statements to load components based on the platform.

Here's an example of how you can conditionally include QML code for Android only:

```qml
import QtQuick 2.15
import QtQuick.Controls 2.15

ApplicationWindow {
    visible: true
    width: 640
    height: 480

    // Define a component to be used on Android only
    Component {
        id: androidComponent
        Rectangle {
            color: "green"
            width: 200
            height: 200
            Text {
                text: "This is Android-specific content"
                anchors.centerIn: parent
            }
        }
    }

    // Use a Loader to conditionally load the component based on the platform
    Loader {
        id: platformLoader
        sourceComponent: Qt.platform.os === "android" ? androidComponent : null
    }
}
```

In this example, the `Loader` element will only load the `androidComponent` if the application is running on an Android device. The `Qt.platform.os` property is used to check the operating system, and it will be set to `"android"` on Android devices.

# Preventing QML From Even Getting Initialized

You can ensure that the androidComponent is not initialized at all unless it's needed by using the Loader element and defining the component inline. This way, the component is only created if the platform condition is met, which prevents unnecessary initialization.

Here's how you can do it:

```qml
import QtQuick 2.15
import QtQuick.Controls 2.15

ApplicationWindow {
    visible: true
    width: 640
    height: 480

    Loader {
        id: platformLoader
        sourceComponent: Qt.platform.os === "android" ? androidComponent : null
    }

    // Define the component inline within the Loader's sourceComponent condition
    Component {
        id: androidComponent
        Rectangle {
            color: "green"
            width: 200
            height: 200
            Text {
                text: "This is Android-specific content"
                anchors.centerIn: parent
            }
        }
    }
}
```

In this setup, the `androidComponent` is defined inline within the `Loader`'s `sourceComponent` condition. This ensures that the component is only instantiated if the platform condition (`Qt.platform.os === "android"`) is met. Otherwise, the component remains uninitialized.

By using this approach, you can avoid any unnecessary initialization of components that are not needed for the current platform.

Here is a link to the other platform strings you can use:
https://doc.qt.io/qt-6/qml-qtqml-qt.html#platform-prop
