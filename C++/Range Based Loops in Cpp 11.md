---
title: Range Based Loops in C++ 11
layout: note
tags:
  - Cpp
  - loops
---

I came accross this the other day.
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

