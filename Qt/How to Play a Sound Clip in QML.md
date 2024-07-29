---
title: How to Play a Sound Clip in QML
layout: note
tags:
  - Qt6
  - Cpp
---

To play a sound clip once in QML, you can use the `Audio` element from the `QtMultimedia` module. Here's a simple example of how to do this:

1. Ensure you have the QtMultimedia module imported.
2. Define an `Audio` element in your QML file.
3. Use the `play()` method to start the sound.

Here's a sample QML file demonstrating how to play a sound clip once:

```qml
import QtQuick 2.15
import QtQuick.Controls 2.15
import QtMultimedia 5.15

ApplicationWindow {
    visible: true
    width: 640
    height: 480
    title: "Play Sound Once Example"

    Button {
        text: "Play Sound"
        anchors.centerIn: parent
        onClicked: {
            sound.play()
        }
    }

    Audio {
        id: sound
        source: "file:///path/to/your/soundclip.mp3"
        onPlaybackStateChanged: {
            if (playbackState === Audio.StoppedState) {
                console.log("Sound finished playing")
            }
        }
    }
}
```

Replace `"file:///path/to/your/soundclip.mp3"` with the actual path to your sound clip file. When you run this QML application and click the button, the sound clip will play once.