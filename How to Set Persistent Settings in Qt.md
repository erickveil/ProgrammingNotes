---
title: How to Set Persistent Settings in Qt
layout: note
tags:
  - android
  - Cpp
  - iOS
  - Qt6
---

To store a settings value that persists even when the app is stopped, you can use `QSettings` in Qt. `QSettings` provides a convenient way to store and retrieve application settings.

Here is an example of how to use `QSettings` to store a boolean value in your Qt-based Android app:

1. **Include the necessary headers:**

```cpp
#include <QCoreApplication>
#include <QSettings>
#include <QDebug>
```

2. **Write a function to save the boolean setting:**

```cpp
void saveSettings(bool value) {
    QSettings settings("YourCompanyName", "YourAppName");
    settings.setValue("yourBooleanKey", value);
}
```

3. **Write a function to load the boolean setting:**

```cpp
bool loadSettings() {
    QSettings settings("YourCompanyName", "YourAppName");
    return settings.value("yourBooleanKey", false).toBool(); // Default value is false if the key doesn't exist
}
```

4. **Example usage in an application:**

```cpp
int main(int argc, char *argv[])
{
    QCoreApplication app(argc, argv);

    // Save a boolean setting
    saveSettings(true);

    // Load the boolean setting
    bool value = loadSettings();
    qDebug() << "Loaded setting value:" << value;

    return app.exec();
}
```

- **QSettings settings("YourCompanyName", "YourAppName");**
  This line creates a `QSettings` object. The parameters ("YourCompanyName" and "YourAppName") define the organization and application names, respectively. These names are used to determine where the settings are stored on the device.

- **settings.setValue("yourBooleanKey", value);**
  This line sets the value of the key "yourBooleanKey" to the boolean value provided.

- **settings.value("yourBooleanKey", false).toBool();**
  This line retrieves the value associated with "yourBooleanKey". If the key doesn't exist, it returns the default value `false`.

`YourCompanyName` and `YourAppName` do matter because they define the namespace under which your settings are stored. These names help organize settings and avoid conflicts with other applications. 
### Example Scenario

If you have two applications, say "AppA" and "AppB", and both store settings, using the company name and app name helps keep their settings separate. Here is an illustration:

#### For "AppA":
```cpp
QSettings settings("MyCompany", "AppA");
settings.setValue("someKey", someValue);
```

#### For "AppB":
```cpp
QSettings settings("MyCompany", "AppB");
settings.setValue("someKey", someValue);
```

Even though both applications use "someKey", their settings are stored separately because they have different application names.

### Practical Tips

1. **Consistency**: Use consistent names for your company and app across different parts of your application to ensure settings are stored and retrieved correctly.

2. **Uniqueness**: Choose names that are unique to your company and app to avoid conflicts with other developers' settings.
