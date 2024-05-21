---
title: Websockets
layout: note
tags:
  - Linux
  - Cpp
  - Qt6
  - network
---

# Listen on the Command Line

You can use a command-line utility called `websocat` to interact with a WebSocket server from your Linux terminal. `websocat` is a versatile WebSocket client that can send and receive messages via WebSockets directly from the command line.

Here’s how you can install and use `websocat` on a Linux system:

### Using `websocat`

To connect to your WebSocket server and start interacting with it, you can use the following command:

```sh
websocat ws://your.websocket.server:port
```

Replace `ws://your.websocket.server:port` with the actual WebSocket URL of your server.

Here are a few examples of how you might use `websocat`:

#### Connect to a WebSocket server and send a message
```sh
echo "Your message here" | websocat ws://your.websocket.server:port
```

#### Connect to a WebSocket server and interact manually
```sh
websocat ws://your.websocket.server:port
```

The port is optional and will default to 80, like any URL.

This command opens an interactive session where you can type messages and see the server's responses. If the server is broadcasting messages, you can see all of them as they appear.

### Example Usage

Assuming your WebSocket server is running on `ws://localhost:8080`, you can connect to it like this:

```sh
websocat ws://localhost:8080
```

This will open an interactive session. To send a message, simply type it and press Enter. You will see responses from the server in the same terminal.

#### For more advanced usage

`websocat` has many advanced features, such as connecting two WebSockets together, using SSL/TLS, etc. You can find more information and examples in the [websocat GitHub repository](https://github.com/vi/websocat).

With `websocat`, you should be able to interact with your WebSocket server effectively from the command line.

# Interacting with a JSON server with `jq`

Let's say we have a server that broadcasts a status message over and over.
### View the Messages

According to the documentation, you should be prepared to handle various messages, but you are primarily interested in messages starting with `42` and containing a `status-update` or `value-changed`.

Here’s how you can filter and process these messages:
#### Filter Messages
To filter and process messages of interest, you might need to use a script or command that processes incoming data. Here’s an example using `websocat` with `jq` to filter messages:

1. **Install `jq` if you don't have it:**
   ```sh
   sudo apt-get install jq
   ```

2. **Filter and Display Relevant Messages:**
   ```sh
   websocat ws://your.server.ip/socket.io | grep '42["status-update' | jq .
   ```

This command connects to the WebSocket server, filters messages that start with `42["status-update`, and processes the JSON data using `jq`.

### Handling the `status-update` Messages

You might want to write a small script to handle the JSON data more precisely. Here’s a bash script example to get you started:

```bash
#!/bin/bash

# Connect to the WebSocket server and filter messages
websocat ws://your.server.ip/socket.io | while read -r line; do
  if [[ $line == 42* ]]; then
    # Extract JSON object from the message
    json=$(echo $line | sed 's/^42\[\"status-update\",//;s/\]$//')
    # Use jq to process the JSON object
    echo $json | jq .
  fi
done
```

Save this script as `amplifier_monitor.sh`, make it executable, and run it:

```sh
chmod +x amplifier_monitor.sh
./amplifier_monitor.sh
```

### Example Outputs
Based on the example messages provided in the documentation, you should see outputs like:

```json
{
  "payload": {}
}
```

or

```json
{
  "version": "1.0.0",
  "payload": {
    "type": 99999
  }
}
```

or

```json
{
  "version": "1.0.0",
  "payload": {
    "type": 99999
  },
  "a": "b"
}
```

# Reading Websockets in C++ and Qt

To establish a connection to the WebSocket and handle incoming JSON messages in C++/Qt, you can use the `QWebSocket` class provided by the Qt WebSockets module. Below is a step-by-step guide on how to do this:

### Include Qt WebSockets Module

Make sure you have the Qt WebSockets module installed. Add it to your `.pro` file:

```plaintext
QT += websockets
```

### Create a WebSocket Client Class

Create a class to handle the WebSocket connection and process the incoming messages.

#### websocketclient.h

```cpp
#ifndef WEBSOCKETCLIENT_H
#define WEBSOCKETCLIENT_H

#include <QObject>
#include <QWebSocket>
#include <QJsonDocument>
#include <QJsonObject>

class WebSocketClient : public QObject {
    Q_OBJECT
public:
    explicit WebSocketClient(const QUrl &url, QObject *parent = nullptr);

signals:
    void jsonReceived(const QJsonObject &json);

private slots:
    void onConnected();
    void onTextMessageReceived(const QString &message);

private:
    QWebSocket m_webSocket;
    QUrl m_url;
    QJsonObject m_latestJson;
};

#endif // WEBSOCKETCLIENT_H
```

#### websocketclient.cpp

```cpp
#include "websocketclient.h"
#include <QDebug>

WebSocketClient::WebSocketClient(const QUrl &url, QObject *parent)
    : QObject(parent), m_url(url) {
    connect(&m_webSocket, &QWebSocket::connected, this, &WebSocketClient::onConnected);
    connect(&m_webSocket, &QWebSocket::disconnected, []() {
        qDebug() << "WebSocket disconnected.";
    });
    m_webSocket.open(QUrl(url));
}

void WebSocketClient::onConnected() {
    qDebug() << "WebSocket connected.";
    connect(&m_webSocket, &QWebSocket::textMessageReceived,
            this, &WebSocketClient::onTextMessageReceived);
}

void WebSocketClient::onTextMessageReceived(const QString &message) {
    // Check if the message starts with "42["
    if (message.startsWith("42[")) {
        // Extract the JSON part
        QString jsonPart = message.mid(2);
        
        // Parse the JSON
        QJsonDocument doc = QJsonDocument::fromJson(jsonPart.toUtf8());
        if (!doc.isNull() && doc.isArray()) {
            QJsonArray jsonArray = doc.array();
            if (jsonArray.size() == 2 && jsonArray[0].isString()) {
                QString type = jsonArray[0].toString();
                QJsonObject jsonObject = jsonArray[1].toObject();
                
                // Process the JSON object
                if (type == "status-update" || type == "value-changed") {
                    m_latestJson = jsonObject;
                    emit jsonReceived(jsonObject);
                }
            }
        }
    }
}
```

### Use the WebSocket Client in Your Application

In your main application, create an instance of the `WebSocketClient` and connect the `jsonReceived` signal to a slot that will handle the new JSON data.

#### main.cpp

```cpp
#include <QCoreApplication>
#include "websocketclient.h"

int main(int argc, char *argv[]) {
    QCoreApplication a(argc, argv);

    QUrl url(QStringLiteral("ws://your.server.ip/socket.io"));
    WebSocketClient client(url);

    QObject::connect(&client, &WebSocketClient::jsonReceived, [](const QJsonObject &json) {
        qDebug() << "New JSON received:" << QJsonDocument(json).toJson(QJsonDocument::Indented);
    });

    return a.exec();
}
```

### Explanation

1. **`WebSocketClient` Constructor**: Initializes the WebSocket and connects its signals to the appropriate slots.
2. **`onConnected` Slot**: Connected to the `QWebSocket::connected` signal and sets up a connection to handle incoming text messages.
3. **`onTextMessageReceived` Slot**: Checks if the message starts with `42[` and extracts the JSON part. If the JSON is valid and contains the expected structure, it updates the `m_latestJson` member variable and emits the `jsonReceived` signal.
4. **Main Application**: Sets up the `WebSocketClient` and connects the `jsonReceived` signal to a lambda function that prints the new JSON data.

### Note

Replace `ws://your.server.ip/socket.io` with the actual URL of your WebSocket server.

With this setup, your Qt application will connect to the WebSocket server, listen for incoming messages, and process them as described.

