---
title: How to Use Custom QML Types in Qt6
layout: note
tags:
  - Cpp
  - Qt6
---

The documentation isn't clear about this and the wizard doesn't set it up for you. In the documentation, it makes it sound like you just need your Custom Type in the same directory as main.qml. Also, this will make the IDE properly highlight the Custom Type as if it's valid. But when you run, you will get an error like

```
Custom Type not found.
```

Here's what you have to do to get Qt 6 to recognize your custom type:

- Add a new **Qt Resource File** to your project if you don't have one. Name it `qml`.
	- Set its prefix to just "/"
- Add any Custom Types to this resource file.
- Also add the `main.qml` file to the resources.
- In `main.cpp`, change the main URL:

Original URL created witth the project:

```
const QUrl url(u"qrc:/MyProjectDirectory/main.qml"_qs);
```

Change to:

```
 const QUrl url(u"qrc:/main.qml"_qs);
```

This will load the main.qml from resources instead of the directory. The QML engine will recognize any Custom Types in that same resource directory.

When creating new Custom QML Types, you can just create them directly inside your qml.qrc resource directory in the project tree. There is no need to create it in the QML directory and add it to Resources as a separate step.

