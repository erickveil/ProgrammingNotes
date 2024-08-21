---
title: How to Block Code to Compile on Specific Platforms in C++
layout: note
tags:
  - Cpp
---

In C++/Qt, you can use preprocessor directives to include platform-specific code. For Windows-specific code, you can use the `#ifdef` or `#if defined` preprocessor directives to check for the Windows platform. The predefined macro `_WIN32` is typically used for this purpose.

Here's an example of how you can designate a section of your code to execute only on Windows machines:

```cpp
#ifdef _WIN32
// Your Windows-specific code here
#include <windows.h> // Example of including a Windows-specific header

void windowsSpecificFunction() {
    // Windows-specific implementation
}
#endif
```

Alternatively, you can use `#if defined` for better readability:

```cpp
#if defined(_WIN32)
// Your Windows-specific code here
#include <windows.h> // Example of including a Windows-specific header

void windowsSpecificFunction() {
    // Windows-specific implementation
}
#endif
```

By wrapping your Windows-specific code with these preprocessor directives, you ensure that the code is only compiled and executed on Windows machines. On other platforms, the code within these directives will be ignored.

If you also need to handle other platforms, you can use similar directives for those platforms. Here is an example that includes platform-specific sections for Windows, macOS, and Linux:

```cpp
#ifdef _WIN32
// Windows-specific code
#include <windows.h>

void platformSpecificFunction() {
    // Windows-specific implementation
}
#elif defined(__APPLE__)
// macOS-specific code
#include <TargetConditionals.h>
#if TARGET_OS_MAC
void platformSpecificFunction() {
    // macOS-specific implementation
}
#endif
#elif defined(__linux__)
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

This approach allows you to maintain platform-specific code within the same source file, ensuring that the appropriate code is compiled for each target platform.

# OS Preprocessor Macros

Preprocessor gates (or macros) for operating systems are predefined by the compiler and allow you to include or exclude code based on the target operating system. Here are some common ones:

1. **Windows**:
   - `_WIN32`: Defined for both 32-bit and 64-bit Windows.
   - `_WIN64`: Defined for 64-bit Windows.

2. **macOS**:
   - `__APPLE__`: Defined for Apple platforms.
   - `__MACH__`: Often used alongside `__APPLE__` to denote macOS specifically.

3. **Linux**:
   - `__linux__`: Defined for Linux systems.
   - `__linux`: Sometimes used interchangeably with `__linux__`.
   - `linux`: Also sometimes used interchangeably, but not as common.

4. **Unix and POSIX**:
   - `__unix__`: Defined for Unix systems.
   - `__unix`: Sometimes used interchangeably with `__unix__`.
   - `unix`: Also sometimes used interchangeably, but not as common.
   - `__posix`: For POSIX-compliant systems.

5. **BSD (including FreeBSD, OpenBSD, NetBSD)**:
   - `__FreeBSD__`: Defined for FreeBSD.
   - `__NetBSD__`: Defined for NetBSD.
   - `__OpenBSD__`: Defined for OpenBSD.

6. **Solaris**:
   - `__sun`: Defined for Solaris.
   - `__SVR4`: Often defined alongside `__sun` for Solaris 2.x and later.

To find the most accurate and up-to-date information on these macros, you can refer to the official documentation of the compilers you are using, such as GCC, Clang, or MSVC.

### Official Documentation Links:

- **GCC Predefined Macros**: [GCC documentation on predefined macros](https://gcc.gnu.org/onlinedocs/cpp/Common-Predefined-Macros.html)
- **Clang Predefined Macros**: [Clang documentation on predefined macros](https://clang.llvm.org/docs/LanguageExtensions.html#builtin-macros)
- **MSVC Predefined Macros**: [MSVC documentation on predefined macros](https://learn.microsoft.com/en-us/cpp/preprocessor/predefined-macros?view=msvc-170)

# Qt Gates

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

# What About QML?

In QML, you can use the `Qt.platform.os` property to determine the platform at runtime and conditionally include sections of your QML code. For marking a section of code to be executed only on Android, you can use a `Loader` or similar conditional statements to load components based on the platform.

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

You can find more details in the official Qt documentation:

- **Qt.platform.os Property**: [Qt Documentation on Qt.platform](https://doc.qt.io/qt-6/qml-qtqml-qt.html#platform-prop)
