---
title: Honoring Newline Characters in `qDebug`
layout: note
tags:
  - Cpp
  - Qt6
---

In Qt, when using `QDebug` to output `QString` or `QByteArray`, it escapes special characters like `\n` by default, which can indeed make JSON or any other formatted text difficult to read. To make `QDebug` honor newline characters correctly, you can convert your `QString` or `QByteArray` to a `QStringLiteral` or use `qPrintable` for `QString`, which will print the string without escaping special characters. Here are a few approaches you can take:

### Using `QStringLiteral` (for QString)

You can use `QStringLiteral` to output the string without escaping special characters:

```cpp
QString jsonString = R"({"key": "value\nnextLine"})";
qDebug().noquote() << QStringLiteral("%1").arg(jsonString);
```

### Using `qPrintable` (for QString)

You can use `qPrintable` to convert `QString` to `const char*`, which will be printed as-is without escaping special characters:

```cpp
QString jsonString = R"({"key": "value\nnextLine"})";
qDebug().noquote() << qPrintable(jsonString);
```

### Using `QByteArray` (for QByteArray)

For `QByteArray`, you can convert it to a `QString` and then apply the same method as above:

```cpp
QByteArray jsonByteArray = R"({"key": "value\nnextLine"})";
qDebug().noquote() << qPrintable(QString::fromUtf8(jsonByteArray));
```

### Complete Example

Here is a complete example that demonstrates these approaches:

```cpp
#include <QCoreApplication>
#include <QDebug>

int main(int argc, char *argv[]) {
    QCoreApplication a(argc, argv);

    QString jsonString = R"({"key": "value\nnextLine"})";
    QByteArray jsonByteArray = R"({"key": "value\nnextLine"})";

    // Using QStringLiteral
    qDebug().noquote() << QStringLiteral("%1").arg(jsonString);

    // Using qPrintable for QString
    qDebug().noquote() << qPrintable(jsonString);

    // Using qPrintable for QByteArray
    qDebug().noquote() << qPrintable(QString::fromUtf8(jsonByteArray));

    return a.exec();
}
```

1. **`noquote()`**: This method tells `QDebug` to not escape special characters.
2. **`QStringLiteral`**: It helps in using literal strings directly.
3. **`qPrintable()`**: Converts a `QString` to `const char*` which is printed as-is.

By using `noquote()` along with either `QStringLiteral` or `qPrintable`, you can make sure that newline characters and other special characters in your JSON or text are printed correctly, making it more readable in the log output.
