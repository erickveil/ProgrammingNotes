---
title: "Converting Qt5 to Qt6: `QOverload::of` in Signals and Slots"
layout: note
tags:
  - Cpp
  - Qt6
  - Qt5
---
I have a class I had written a long time ago in C++/Qt5. Now I'm working on a Qt6 program, and I'm using this class. Something is a little off about one of my signal/slot connections, and I'm trying to understand what I was doing.

I have the class member ` QTcpSocket *_connection;` that gets instantiated elsewhere.

Where I do the signal/slot connections, I have this interesting bit of code:

```c++
 connect(_connection, QOverload<QAbstractSocket::SocketError>::
            of(&QAbstractSocket::error),
            [=](QAbstractSocket::SocketError socketError) {
        _eventSocketError(socketError);
    });
```

So, what is `QOverload::of`?

In Qt, `QOverload` is used to help with the disambiguation of overloaded signals or slots. The `of` function is a part of `QOverload` which helps in specifying the exact signal or slot you are referring to, especially when multiple signals or slots have the same name but different parameters.

In your code:

```c++
 connect(_connection, QOverload<QAbstractSocket::SocketError>::
            of(&QAbstractSocket::error),
            [=](QAbstractSocket::SocketError socketError) {
        _eventSocketError(socketError);
    });
```

This line is setting up a connection between the `error` signal from `_connection` (which is a `QTcpSocket` object) and a lambda function that will call `_eventSocketError`.

The `QOverload<QAbstractSocket::SocketError>::of(&QAbstractSocket::error)` part specifies which overload of the `error` signal to use. `QAbstractSocket` has an `error` signal that can be overloaded, and `QOverload` helps in specifying the correct one.

Here's a breakdown of the key components:

1. **`QOverload<QAbstractSocket::SocketError>::of(&QAbstractSocket::error)`**:
    - `QOverload`: A template helper used to specify the exact overload of a signal or slot.
    - `QAbstractSocket::SocketError`: The type of the parameter for the specific `error` signal overload.
    - `of(&QAbstractSocket::error)`: The `of` function helps to get the pointer to the specific signal overload.

2. **Lambda Function**:
    - The lambda function `[=](QAbstractSocket::SocketError socketError)` is defined as the slot that will be called when the signal is emitted. It captures all local variables by value (due to `[=]`) and calls `_eventSocketError` with the `socketError` parameter.

This approach is especially useful when you have multiple signals or slots with the same name but different parameters and you need to specify which one you want to connect to.

### Example without `QOverload` in Qt5

In Qt5, you might see the connection done differently, without `QOverload`:

```c++
connect(_connection, SIGNAL(error(QAbstractSocket::SocketError)),
        this, SLOT(_eventSocketError(QAbstractSocket::SocketError)));
```

This older syntax relies on macros `SIGNAL` and `SLOT` to specify the signal and slot to connect. However, this approach is less type-safe and doesn't provide compile-time checking.

# Replacement of `QOverload` and `of`

In Qt6, the use of `QOverload` and the `of` method has been replaced by using direct function pointers for better type safety and simplicity. To adapt your code to Qt6, you can directly use the signal's function pointer in the `connect` call. Here is how you can rewrite your connection:

```c++
connect(_connection, &QAbstractSocket::errorOccurred, this, &YourClassName::_eventSocketError);
```

### Explanation

1. **Signal Function Pointer**:
    - In Qt6, you can directly use the function pointer of the signal. For the `error` signal of `QAbstractSocket`, the new signal is `errorOccurred`.

2. **Slot Function Pointer**:
    - You directly specify the member function of your class that will handle the signal.

### Example

Assuming `_eventSocketError` is a member function of your class `YourClassName`, the updated code would look something like this:

```c++
class YourClassName : public QObject {
    Q_OBJECT

public:
    YourClassName(QObject *parent = nullptr);

private slots:
    void _eventSocketError(QAbstractSocket::SocketError socketError);

private:
    QTcpSocket *_connection;
};

YourClassName::YourClassName(QObject *parent) : QObject(parent), _connection(new QTcpSocket(this)) {
    // Connect the signal to the slot
    connect(_connection, &QAbstractSocket::errorOccurred, this, &YourClassName::_eventSocketError);
}

void YourClassName::_eventSocketError(QAbstractSocket::SocketError socketError) {
    // Handle the socket error here
}
```

This approach ensures that the signal and slot connections are type-safe and checked at compile time, which is one of the improvements in Qt6's signal and slot system.

### Key Changes in Qt6

- **Signals and Slots Directly as Function Pointers**: Instead of using `QOverload` and `of`, you use the function pointer syntax directly.
- **Signal Renaming**: Some signals have been renamed for consistency. For example, `QAbstractSocket::error` has been renamed to `QAbstractSocket::errorOccurred`.

By using the direct function pointer approach, you make your code cleaner and more in line with modern C++ practices.