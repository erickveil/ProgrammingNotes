---
title: REST API on the Command Line and QT
layout: note
tags:
  - Cpp
  - Linux
  - shell
  - rest
  - api
---

# Sending a Request on the Command Line

You can use `curl` to send a POST request with JSON data from the command line. Here is how you can do it:

```sh
curl -X POST http://192.168.0.152:80 \
     -H "Content-Type: application/json" \
     -d '{
           "clientId": "myId",
           "payload": {
             "type": "WRITE",
             "value": "Hello world"
           }
         }'
```

Here's a breakdown of the command:
- `-X POST` specifies the request method as POST.
- `http://192.168.0.152:80` is the URL of the server you're sending the request to.
- `-H "Content-Type: application/json"` sets the `Content-Type` header to `application/json`.
- `-d '...'` includes the JSON data as the body of the POST request.

This command will send your JSON payload to the specified server and port.

For simplicity and ease of use, `curl` is the most appropriate tool for sending a JSON POST request from the command line.

## `curl` Response

When you use `curl` to send a request, the response from the server will be printed to stdout by default. If the server responds with any data, you will see it directly in your terminal.

When you run the command, `curl` will output the server's response to the terminal. If you want to suppress this output or redirect it to a file, you can use additional options. For example, to redirect the output to a file:

```sh
curl -X POST http://192.168.0.152:80 \
     -H "Content-Type: application/json" \
     -d '{
           "clientId": "myId",
           "payload": {
             "type": "WRITE",
             "value": "Hello world"
           }
         }' > response.txt
```

This command will save the response to `response.txt` instead of printing it to stdout.

If you want to see the response headers as well, you can use the `-i` option:

```sh
curl -i -X POST http://192.168.0.152:80 \
     -H "Content-Type: application/json" \
     -d '{
           "clientId": "myId",
           "payload": {
             "type": "WRITE",
             "value": "Hello world"
           }
         }'
```

The `-i` option includes the HTTP response headers in the output, which can be useful for debugging or inspecting the server's response in detail.

## Basic Authentication

To include Basic Authentication in your `curl` request, you can use the `-u` option followed by the username and password in the format `username:password`. Here is how you can modify your `curl` command to include Basic Authentication:

```sh
curl -X POST http://192.168.0.152:80 \
     -H "Content-Type: application/json" \
     -u username:password \
     -d '{
           "clientId": "myId",
           "payload": {
             "type": "WRITE",
             "value": "Hello world"
         }
         }'
```

Replace `username` and `password` with the actual credentials provided in the documentation.

Here's a breakdown of the additional part:
- `-u username:password` specifies the username and password for Basic Authentication.

With this addition, the server will authenticate the request using the provided credentials. The response from the server will still be printed to stdout by default.

# Doing the Same Thing in C++/Qt

To write a function in C++ using the Qt framework to send a POST request with JSON data and Basic Authentication, you can use the `QNetworkAccessManager`, `QNetworkRequest`, and `QJsonDocument` classes. Here’s a step-by-step example:

1. **Include the necessary headers:**

```cpp
#include <QCoreApplication>
#include <QNetworkAccessManager>
#include <QNetworkRequest>
#include <QNetworkReply>
#include <QJsonDocument>
#include <QJsonObject>
#include <QUrl>
#include <QByteArray>
#include <QDebug>
#include <QAuthenticator>
#include <QEventLoop>
```

2. **Create the function to send the POST request:**

```cpp
void sendPostRequest() {
    QNetworkAccessManager manager;

    // Set up the URL and request
    QUrl url("http://192.168.0.152:80");
    QNetworkRequest request(url);

    // Set the Content-Type header
    request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json");

    // Set up Basic Authentication
    QString username = "username";
    QString password = "password";
    QString concatenated = username + ":" + password;
    QByteArray data = concatenated.toLocal8Bit().toBase64();
    QString headerData = "Basic " + data;
    request.setRawHeader("Authorization", headerData.toLocal8Bit());

    // Create the JSON payload
    QJsonObject payload;
    payload["type"] = "WRITE";
    payload["value"] = "Hello world";

    QJsonObject json;
    json["clientId"] = "myId";
    json["payload"] = payload;

    QJsonDocument doc(json);
    QByteArray jsonData = doc.toJson();

    // Send the request and wait for the reply
    QNetworkReply *reply = manager.post(request, jsonData);

    // Use an event loop to wait for the reply
    QEventLoop loop;
    QObject::connect(reply, &QNetworkReply::finished, &loop, &QEventLoop::quit);
    loop.exec();

    // Process the reply
    if (reply->error() == QNetworkReply::NoError) {
        QByteArray response = reply->readAll();
        qDebug() << "Response:" << response;
    } else {
        qDebug() << "Error:" << reply->errorString();
    }

    // Clean up
    reply->deleteLater();
}
```

3. **Set up the main function:**

```cpp
int main(int argc, char *argv[]) {
    QCoreApplication a(argc, argv);

    sendPostRequest();

    return a.exec();
}
```

4. **Compile and run the application:**

Make sure your `.pro` file includes the necessary network module:

```pro
QT += core network
CONFIG += console
CONFIG -= app_bundle
```

Here’s a breakdown of what the function does:

- **QNetworkAccessManager**: Manages the network requests.
- **QNetworkRequest**: Encapsulates information about the request (URL, headers, etc.).
- **QJsonDocument**: Used to create and format the JSON payload.
- **QAuthenticator**: Handles authentication if needed, but here we manually set the Basic Authentication header.
- **QEventLoop**: Waits synchronously for the network reply.

When you run this function, it sends the POST request to the specified server with the JSON payload and Basic Authentication. The response from the server is then printed to the debug output.

