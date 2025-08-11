---
title: Range Based Loops in C++ 11
layout: note
tags:
  - Cpp
  - loops
---

I came across this the other day.
Take a look at the loop structure in the following code:

```cpp
void AppController::fetchLocalIpAddress()
{
    QList<QHostAddress> hostAddressList = QNetworkInterface::allAddresses();
    for (const QHostAddress &address : hostAddressList) {
        bool isGoodAddress = address.protocol() == QAbstractSocket::IPv4Protocol
                            && !address.isLoopback();
        if (!isGoodAddress) { continue; }
        _localIpAddress = address.toString();
        emit localIpAddressChanged();
        break;
    }
}
```

This is not the usual `for(int i=0 ...)` style loop we are used to seeing in C++!

This loop structure is known as a range-based for loop, introduced in C++11. It is designed to simplify iterating over elements in a container. Let's break down your method to understand it better:

### Breakdown of the Method

1. **Fetching all IP Addresses**: 
   ```cpp
   QList<QHostAddress> hostAddressList = QNetworkInterface::allAddresses();
   ```
   This line retrieves all IP addresses associated with the network interfaces on the machine and stores them in a `QList` of `QHostAddress` objects.

2. **Range-based for Loop**:
   ```cpp
   for (const QHostAddress &address : hostAddressList) {
   ```
   This loop iterates over each `QHostAddress` object in the `hostAddressList`. The `address` variable is a reference to the current `QHostAddress` object.

3. **Checking for IPv4 and non-loopback Addresses**:
   ```cpp
   bool isGoodAddress = address.protocol() == QAbstractSocket::IPv4Protocol
                        && !address.isLoopback();
   if (!isGoodAddress) { continue; }
   ```
   This section checks if the address is an IPv4 address and not a loopback address (i.e., not `127.0.0.1` or `::1`). If the address does not meet these criteria, the loop continues to the next iteration.

4. **Assigning and Emitting**:
   ```cpp
   _localIpAddress = address.toString();
   emit localIpAddressChanged();
   break;
   ```
   If a valid address is found, it converts the address to a string, assigns it to `_localIpAddress`, emits a signal `localIpAddressChanged()`, and breaks out of the loop.

### Equivalent Classic `for` Loop

If you are more familiar with the traditional `for` loop structure, here's how the same logic would be implemented using an index-based loop:

```cpp
void AppController::fetchLocalIpAddress()
{
    QList<QHostAddress> hostAddressList = QNetworkInterface::allAddresses();
    for (int i = 0; i < hostAddressList.size(); ++i) {
        const QHostAddress &address = hostAddressList.at(i);
        bool isGoodAddress = address.protocol() == QAbstractSocket::IPv4Protocol
                            && !address.isLoopback();
        if (!isGoodAddress) { continue; }
        _localIpAddress = address.toString();
        emit localIpAddressChanged();
        break;
    }
}
```


# Qt gives you `foreach`!

The `foreach` construct is actually a macro provided by Qt for convenience. However, the standard C++ way to achieve the same functionality is using range-based for loops or iterators.

### Using Qt's `foreach` Macro:
Qt provides a `foreach` macro for iterating over containers, which is a convenient shorthand:

```cpp
#include <QDir>
#include <QStringList>
#include <QDebug>
#include <QCoreApplication>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // Specify the resource path
    QDir resourceDir(":/BarrowCrawlResources/_Sporedeep/portraits/");

    // Check if the directory exists
    if (!resourceDir.exists()) {
        qDebug() << "Directory does not exist!";
        return -1;
    }

    // Set the filter for JPG files
    QStringList filters;
    filters << "*.jpg";

    // Get the list of JPG files
    QStringList jpgFiles = resourceDir.entryList(filters, QDir::Files);

    // Print the list of files using foreach
    foreach (const QString &file, jpgFiles) {
        qDebug() << file;
    }

    return a.exec();
}
```

#### Don't Use `foreach` With `QJsonArray`
It's important to note that `foreach` will copy the container, while the range-based loop will not. In the case of `QJsonArray`, the use of `foreach` is deprecated. There's a risk that you'll be looping through the JSON and making changes to it, and in this case you'd be making changes to the copy, and not your actual JSON structure.

As of this writing, you'll get a compiler warning for this.

### Using Standard C++ Range-Based For Loop:
The equivalent code using the standard C++11 range-based for loop is as follows:

```cpp
#include <QDir>
#include <QStringList>
#include <QDebug>
#include <QCoreApplication>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // Specify the resource path
    QDir resourceDir(":/BarrowCrawlResources/_Sporedeep/portraits/");

    // Check if the directory exists
    if (!resourceDir.exists()) {
        qDebug() << "Directory does not exist!";
        return -1;
    }

    // Set the filter for JPG files
    QStringList filters;
    filters << "*.jpg";

    // Get the list of JPG files
    QStringList jpgFiles = resourceDir.entryList(filters, QDir::Files);

    // Print the list of files using range-based for loop
    for (const QString &file : jpgFiles) {
        qDebug() << file;
    }

    return a.exec();
}
```

### Using Iterators:
If you prefer using iterators, here's how you can do it:

```cpp
#include <QDir>
#include <QStringList>
#include <QDebug>
#include <QCoreApplication>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // Specify the resource path
    QDir resourceDir(":/BarrowCrawlResources/_Sporedeep/portraits/");

    // Check if the directory exists
    if (!resourceDir.exists()) {
        qDebug() << "Directory does not exist!";
        return -1;
    }

    // Set the filter for JPG files
    QStringList filters;
    filters << "*.jpg";

    // Get the list of JPG files
    QStringList jpgFiles = resourceDir.entryList(filters, QDir::Files);

    // Print the list of files using iterators
    for (QStringList::const_iterator it = jpgFiles.begin(); it != jpgFiles.end(); ++it) {
        qDebug() << *it;
    }

    return a.exec();
}
```

- **Qt `foreach` Macro**: Provides a convenient way to iterate over Qt containers.
- **Range-Based For Loop**: Standard C++11 feature for iterating over containers.
- **Iterators**: Traditional method for iterating over containers, available in all versions of C++. 

All these methods achieve the same result, and you can choose the one that best fits your coding style or project requirements.