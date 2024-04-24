---
title: Swift Reading and Writing Files
layout: note
tags:
  - swift
  - "#iOS"
  - files
---

# Reading

To load the contents of a JSON file into a string in a Swift iOS app, you'll want to use the `Bundle` class to locate the file within your project's resources, and then read the contents into a string. Here’s a basic way to do it:

1. First, make sure your JSON file (let's call it `data.json`) is added to your project's bundle. You can add it by dragging the file into the Xcode project navigator and making sure it's included in the "Copy Bundle Resources" section of your target's Build Phases.

2. Write a function or include the following code in your app to read the file and convert its contents into a string:

```swift
func loadJsonFile() -> String? {
    guard let path = Bundle.main.path(forResource: "data", ofType: "json") else {
        print("JSON file not found")
        return nil
    }
    do {
        let data = try Data(contentsOf: URL(fileURLWithPath: path))
        let jsonString = String(data: data, encoding: .utf8)
        return jsonString
    } catch {
        print("Error reading JSON file: \(error)")
        return nil
    }
}
```

3. You can call `loadJsonFile()` from anywhere in your app where you need the JSON string. If the function returns `nil`, it means the file was not found or there was an error reading it.

This function checks if the JSON file exists in the app bundle, reads it into a `Data` object, and then attempts to convert this data into a string. If there's any problem in finding the file or reading it, the function will return `nil` and print an error message to the console. Make sure to handle these cases in your app to avoid crashes or unexpected behavior.

# Writing

Here we will write a JSON string to a file in the document directory of your iOS app, which is a suitable place for storing user-generated or application-generated files.

1. **Define the JSON String**: Ensure you have your JSON string ready. For example:
   ```swift
   let jsonString = "{\"key\":\"value\"}"
   ```

2. **Get the Path to the Document Directory**: You need to find the path to the document directory where you can save your file. Here’s how you can get the path and define the file name:

   ```swift
   guard let documentDirectory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first else {
       print("Document directory not found")
       return
   }
   let filePath = documentDirectory.appendingPathComponent("data.json")
   ```

3. **Write the JSON String to the File**: Convert your JSON string to `Data` and then write this data to the file system:

   ```swift
   do {
       try jsonString.data(using: .utf8)?.write(to: filePath, options: .atomic)
       print("JSON string was written to the file successfully")
   } catch {
       print("Failed to write JSON string to the file: \(error)")
   }
   ```

In this code:
- We first get the path to the document directory. This is a directory where your app can freely write and read files.
- We append `"data.json"` to the path to define where the file will be located.
- We attempt to convert the JSON string to `Data` using UTF-8 encoding.
- We write this data to the specified file path using the `write(to:options:)` method of the `Data` class. The option `.atomic` ensures that the file is first written to a temporary file and then renamed to the target file name, which is a safer way to handle file writing.

This will create a file `data.json` in the document directory with the contents of your JSON string. You can adjust the file path and handling as needed for your specific app requirements.

# Writing to the Bundle?

The code for writing to a file saves the JSON string to the document directory, not back to the original file located in your app's bundle. Files in the iOS app bundle are read-only; they cannot be modified after the app has been compiled and installed on a device. This is a security measure to ensure that app resources are not tampered with after distribution.

If you want to manage read and write operations for the same data within your app, you should initially copy the file from the bundle to a writable location, such as the document directory, when your app first runs. From there, you can read from and write to the file as needed.

Here's how you can copy a file from the bundle to the document directory on the first run and then use it for read/write operations:

1. **Check and Copy the File from the Bundle to the Document Directory**:
   ```swift
   func setupDataFile() {
       let fileManager = FileManager.default
       guard let documentDirectory = fileManager.urls(for: .documentDirectory, in: .userDomainMask).first else {
           print("Document directory not found")
           return
       }
       let destinationPath = documentDirectory.appendingPathComponent("data.json")

       // Check if the file already exists to avoid overwriting it each time the app runs
       if !fileManager.fileExists(atPath: destinationPath.path) {
           if let bundlePath = Bundle.main.path(forResource: "data", ofType: "json") {
               do {
                   try fileManager.copyItem(atPath: bundlePath, toPath: destinationPath.path)
                   print("File copied successfully")
               } catch {
                   print("Error copying file: \(error)")
               }
           } else {
               print("JSON file not found in bundle")
           }
       }
   }
   ```

2. **Read and Write to the File in the Document Directory**:
   - Use the earlier provided methods to read from and write to the file at `destinationPath` in the document directory.

By doing this, you create a writable copy of your initial JSON file when the app is first launched. Subsequent reads and writes are then performed on this copy in the document directory. This approach allows your app to maintain a modifiable data set across sessions.

