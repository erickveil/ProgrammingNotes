---
title: Swift Socket Client
layout: note
tags:
  - swift
  - iOS
  - tcp
  - client
  - socket
---
# Basic Client

To create a simple TCP socket client in Swift that sends "Hello World" to a server and logs the response, you can use the `Stream` class which provides basic socket functionalities. Below is a step-by-step guide on how to achieve this:

1. **Create a new Swift file for your TCP client**: You can name it `TCPClient.swift`.

2. **Import the necessary module**:
   ```swift
   import Foundation
   ```

3. **Define the TCPClient class**:
   Here you'll create a class that manages the connection, sending, and receiving of data.

   ```swift
   class TCPClient {
       var inputStream: InputStream?
       var outputStream: OutputStream?
       
       let serverAddress: String
       let serverPort: UInt32
       
       init(address: String, port: UInt32) {
           self.serverAddress = address
           self.serverPort = port
       }
       
       func connect() {
           var readStream: Unmanaged<CFReadStream>?
           var writeStream: Unmanaged<CFWriteStream>?
           
           CFStreamCreatePairWithSocketToHost(kCFAllocatorDefault,
                                              self.serverAddress as CFString,
                                              self.serverPort,
                                              &readStream,
                                              &writeStream)
           
           inputStream = readStream!.takeRetainedValue()
           outputStream = writeStream!.takeRetainedValue()
           
           inputStream!.open()
           outputStream!.open()
       }
       
       func send(message: String) {
           let data = message.data(using: .utf8)!
           _ = data.withUnsafeBytes {
               outputStream?.write($0.bindMemory(to: UInt8.self).baseAddress!, maxLength: data.count)
           }
       }
       
       func read() -> String? {
           let buffer = UnsafeMutablePointer<UInt8>.allocate(capacity: 1024)
           defer {
               buffer.deallocate()
           }
           
           if let n = inputStream?.read(buffer, maxLength: 1024) {
               if n > 0 {
                   return String(bytesNoCopy: buffer, length: n, encoding: .utf8, freeWhenDone: true)
               }
           }
           return nil
       }
       
       func disconnect() {
           inputStream?.close()
           outputStream?.close()
           inputStream = nil
           outputStream = nil
       }
   }
   ```

4. **Usage**:
   You can now use this client in your app to connect to your server, send a message, and receive a response. Here’s how you might do it from a view controller or other parts of your app:

   ```swift
   let client = TCPClient(address: "192.168.1.100", port: 50502)
   client.connect()
   client.send(message: "Hello World")
   if let response = client.read() {
       print("Received: \(response)")
   }
   client.disconnect()
   ```

Make sure to replace `"192.168.1.100"` with the IP address of your network where the server is running. This code assumes your server is capable of handling the incoming connection and responding appropriately. Since network operations can block the main thread, consider running this code in a background thread or using asynchronous methods depending on your application's architecture.

# Handling Errors

To handle and log errors in TCP communication within your Swift application, you can enhance the error handling capabilities of the `TCPClient` class. Here’s how you can modify the existing code to capture and log specific TCP errors:

1. **Modify the `connect`, `send`, `read`, and `disconnect` methods** to handle exceptions and errors more robustly.

2. **Add error handling to the stream events**:
   You can utilize the stream delegate method to handle errors more effectively. This involves conforming to the `StreamDelegate` protocol and managing different stream events.

Here’s an updated version of your `TCPClient` class with error handling:

```swift
import Foundation

class TCPClient: NSObject, StreamDelegate {
    private var inputStream: InputStream?
    private var outputStream: OutputStream?
    
    let serverAddress: String
    let serverPort: UInt32
    
    init(address: String, port: UInt32) {
        self.serverAddress = address
        self.serverPort = port
    }
    
    func connect() {
        var readStream: Unmanaged<CFReadStream>?
        var writeStream: Unmanaged<CFWriteStream>?
        
        CFStreamCreatePairWithSocketToHost(kCFAllocatorDefault,
                                           self.serverAddress as CFString,
                                           self.serverPort,
                                           &readStream,
                                           &writeStream)
        
        inputStream = readStream!.takeRetainedValue()
        outputStream = writeStream!.takeRetainedValue()
        
        inputStream!.delegate = self
        outputStream!.delegate = self
        
        inputStream!.schedule(in: .current, forMode: .common)
        outputStream!.schedule(in: .current, forMode: .common)
        
        inputStream!.open()
        outputStream!.open()
    }
    
    func send(message: String) {
        let data = message.data(using: .utf8)!
        _ = data.withUnsafeBytes {
            outputStream?.write($0.bindMemory(to: UInt8.self).baseAddress!, maxLength: data.count)
        }
    }
    
    func read() -> String? {
        let buffer = UnsafeMutablePointer<UInt8>.allocate(capacity: 1024)
        defer {
            buffer.deallocate()
        }
        
        if let n = inputStream?.read(buffer, maxLength: 1024) {
            if n > 0 {
                return String(bytesNoCopy: buffer, length: n, encoding: .utf8, freeWhenDone: true)
            } else if n == 0 {
                print("End of stream reached")
            } else {
                print("Error occurred while reading from stream")
            }
        }
        return nil
    }
    
    func stream(_ aStream: Stream, handle eventCode: Stream.Event) {
        switch eventCode {
        case .errorOccurred:
            if let error = aStream.streamError {
                print("Stream error: \(error.localizedDescription)")
            }
        case .endEncountered:
            disconnect()
            print("Stream ended")
        default:
            break
        }
    }
    
    func disconnect() {
        inputStream?.close()
        outputStream?.close()
        inputStream?.remove(from: .current, forMode: .common)
        outputStream?.remove(from: .current, forMode: .common)
        inputStream = nil
        outputStream = nil
    }
}
```

In this updated class:
- **Stream Delegate**: The class now conforms to `StreamDelegate`, and the delegate methods handle different stream events, including errors and end-of-stream.
- **Error Handling**: Errors during read and write operations, as well as stream errors, are logged through the delegate method. This includes logging the description of the error.

You can further modify the `print` statements to log to a file or a logging framework depending on how you handle logging in the rest of your app. Implementing detailed logging is crucial for diagnosing issues in network communications, especially in production environments.

# Adding Callbacks for Flexibility

To add callback functionality to your `TCPClient` class in Swift, you can use closures. This approach will allow the caller to specify custom behavior for successful receptions of messages and errors. Here's how you can modify the `TCPClient` class to include optional callback parameters:

1. **Define the callback type signatures**: Define types for your success and error callbacks.
2. **Add the callbacks as optional parameters to your class**: These parameters will store the provided callbacks.
3. **Invoke the callbacks at appropriate points within the class methods**: Trigger these callbacks based on network events such as successful message receipt or an error.

Here's the modified version of your `TCPClient` with callback support:

```swift
import Foundation

class TCPClient: NSObject, StreamDelegate {
    private var inputStream: InputStream?
    private var outputStream: OutputStream?
    
    private let serverAddress: String
    private let serverPort: UInt32

    // Callback definitions
    var onSuccess: ((String) -> Void)?
    var onError: ((Error) -> Void)?
    
    init(address: String, port: UInt32, onSuccess: ((String) -> Void)? = nil, onError: ((Error) -> Void)? = nil) {
        self.serverAddress = address
        self.serverPort = port
        self.onSuccess = onSuccess
        self.onError = onError
    }
    
    func connect() {
        var readStream: Unmanaged<CFReadStream>?
        var writeStream: Unmanaged<CFWriteStream>?
        
        CFStreamCreatePairWithSocketToHost(kCFAllocatorDefault,
                                           self.serverAddress as CFString,
                                           self.serverPort,
                                           &readStream,
                                           &writeStream)
        
        inputStream = readStream!.takeRetainedValue()
        outputStream = writeStream!.takeRetainedValue()
        
        inputStream!.delegate = self
        outputStream!.delegate = self
        
        inputStream!.schedule(in: .current, forMode: .common)
        outputStream!.schedule(in: .current, forMode: .common)
        
        inputStream!.open()
        outputStream!.open()
    }
    
    func send(message: String) {
        let data = message.data(using: .utf8)!
        _ = data.withUnsafeBytes {
            outputStream?.write($0.bindMemory(to: UInt8.self).baseAddress!, maxLength: data.count)
        }
    }
    
    func read() -> String? {
        let buffer = UnsafeMutablePointer<UInt8>.allocate(capacity: 1024)
        defer {
            buffer.deallocate()
        }
        
        if let n = inputStream?.read(buffer, maxLength: 1024) {
            if n > 0 {
                let message = String(bytesNoCopy: buffer, length: n, encoding: .utf8, freeWhenDone: true)
                onSuccess?(message ?? "")
                return message
            } else if n == 0 {
                print("End of stream reached")
            } else {
                print("Error occurred while reading from stream")
            }
        }
        return nil
    }
    
    func stream(_ aStream: Stream, handle eventCode: Stream.Event) {
        switch eventCode {
        case .errorOccurred:
            if let error = aStream.streamError {
                onError?(error)
                print("Stream error: \(error.localizedDescription)")
            }
        case .endEncountered:
            disconnect()
            print("Stream ended")
        default:
            break
        }
    }
    
    func disconnect() {
        inputStream?.close()
        outputStream?.close()
        inputStream?.remove(from: .current, forMode: .common)
        outputStream?.remove(from: .current, forMode: .common)
        inputStream = nil
        outputStream = nil
    }
}
```

**Using the TCPClient with Callbacks:**

Here's how you might use this client in your app:

```swift
let client = TCPClient(address: "192.168.1.100", port: 50502,
                       onSuccess: { message in
                           print("Received successful message: \(message)")
                       },
                       onError: { error in
                           print("An error occurred: \(error.localizedDescription)")
                       })

client.connect()
client.send(message: "Hello World")
```

With this setup, your TCPClient now supports optional callbacks for both successful message receptions and errors. If no callbacks are provided, it will default to logging to the console as before. This makes your client more flexible and suitable for a variety of application contexts.

