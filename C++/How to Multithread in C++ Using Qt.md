---
title: How to Multithread in C++ Using Qt
layout: note
tags:
  - Cpp
  - Qt6
---


My number one advice for multithreading, is this:

Don't.

Don't do it unless you have a really, really good reason to. Especially in Qt, where the event system is almost always a better alternative.

But, alas, we're here because the interviewers keep obsessing over this topic, and they do that because their legacy code has got it. Someone has to maintain it, and ChatGPT isn't going to do it for you.

# Overview

You will need to use signals and slots to control the thread. The thread takes an entire class to be assigned to it, and you signal its start and stop from your main thread.

It's best to have one thread, your main thread (the one you usually work in with a single-threaded application) to manage all other threads. Keep multithreading as simple as possible and don't go creating thread trees if you can at all help it.

# Create Worker and Thread

So let's do some simple work. We're going to make a class that counts from 0 to N in the console. We'll run it in the main thread, and also run it in another thread we create so that it's running twice at the same time.

```
class Counter : public QObject
{
    Q_OBJECT
public:
    explicit Counter(int maxCount, QObject *parent = nullptr);
    int _maxCount;
public slots:
    void doCounting();
};
```

We extend QObject so that we can make use of its `moveToThread()` method. We could do this without any inheritance, but we would have to get clever and not explicitly identify which thread we're running the `Counter` object's methods on. This is confusing for future programmers, and may cause some unexpected problems in more complicated threading programs.

The `doCounting()` method will be where all the work happens.

```
void Counter::doCounting()
{
    for (int i = 0; i < _maxCount; ++i) {
        QString time = QTime::currentTime().toString("hh:mm:ss:zzz");
        qDebug() << "Time: " << time << " | "
                 << "Thread: " << QThread::currentThreadId() << " | "
                 << "Count: " << i;
        QThread::sleep(1);
    }
}
```

- We will mark each output with the time so we can watch when things get written to output.
- We use `QThread::currentThreadId` to get the thread's unique ID. 
	- This works differently on each platform you build it on, and you should only really use this for debugging - don't go trying to get clever with it. On Windows, this will output a hexadecimal value. This will help us to differentiate which thread is doing the output.
- The we use `QThread::sleep` to put a little pause in there. You can always use this to pause, even in your main thread, but remember that it's a blocking sleep. Blocking here is just to simulate something that takes a long time to run.

# Using the Worker Class

Now in main, we can just use the worker class like normal:

```
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    int maxCount = 10;
    Counter mainCounter(maxCount);
    mainCounter.doCounting();
    return a.exec();
}
```

There's a single thread program that works like you're used to. To get another thread, we are going to define another worker, and then give it a thread that we will then start:

```
QThread thread;
Counter threadCounter(maxCount);
```

If you extended QObject to create your worker, at this point you'd have to call `QObject::moveToThread()` on it.  This will move the worker to the other thread, and also tip off any future programmers that this will be working off of the main thread.

In the previous Counter, we were able to just call `Counter::doCounting()` to get it started. But once we start this other thread, we can't control it from here. So we will connect a slot to fire up when the thread starts.

```
 Counter threadCounter(maxCount);
    threadCounter.moveToThread(&thread);
    QObject::connect(&thread, &QThread::started,
                     &threadCounter, &Counter::doCounting);
    thread.start();
```

Now when `QThread::start()` is called, it will automatically emit its `started()` signal. We have that signal connected to our `Counter::doCounting()` as a slot via the Lambda so that it starts when the thread starts.

Also, let's tip ourselves off on which thread ID will be the main thread:

```
qDebug() << "Main thread ID: " << QThread::currentThreadId();
```

This gives a valid thread ID when called from `main()` because it is always run in a thread itself — the main thread. Therefore calling `QThread::currentThreadId()` from `main()` will return the ID of the main thread.

# Running the Program

The full program can be found [here](https://gitlab.com/erickveil/basic-multithreading-with-c-and-qt/-/tree/inheritQObject?ref_type=heads). It's intended to be built with Qt 6.4 on Windows. 

The output will be something like this:

```
Main thread ID:  0x7d0
Time:  "15:50:52:909"  |  Thread:  0x7d0  |  Count:  0
Time:  "15:50:52:925"  |  Thread:  0x2c70  |  Count:  0
Time:  "15:50:53:932"  |  Thread:  0x7d0  |  Count:  1
Time:  "15:50:53:932"  |  Thread:  0x2c70  |  Count:  1
Time:  "15:50:54:947"  |  Thread:  0x2c70  |  Count:  2
Time:  "15:50:54:947"  |  Thread:  0x7d0  |  Count:  2
Time:  "15:50:55:959"  |  Thread:  0x2c70  |  Count:  3
Time:  "15:50:55:959"  |  Thread:  0x7d0  |  Count:  3
Time:  "15:50:56:971"  |  Thread:  0x7d0  |  Count:  4
Time:  "15:50:56:971"  |  Thread:  0x2c70  |  Count:  4
Time:  "15:50:57:976"  |  Thread:  0x2c70  |  Count:  5
Time:  "15:50:57:976"  |  Thread:  0x7d0  |  Count:  5
Time:  "15:50:58:991"  |  Thread:  0x7d0  |  Count:  6
Time:  "15:50:58:991"  |  Thread:  0x2c70  |  Count:  6
Time:  "15:51:00:005"  |  Thread:  0x2c70  |  Count:  7
Time:  "15:51:00:005"  |  Thread:  0x7d0  |  Count:  7
Time:  "15:51:01:007"  |  Thread:  0x2c70  |  Count:  8
Time:  "15:51:01:007"  |  Thread:  0x7d0  |  Count:  8
Time:  "15:51:02:013"  |  Thread:  0x7d0  |  Count:  9
Time:  "15:51:02:013"  |  Thread:  0x2c70  |  Count:  9
```

You can see the threads output whenever they can and it's not always coordinated between the two threads. Even the time stamps show this to be pretty simultaneous —within the same millisecond— but at least its in chronological order.

We can also see that the method is indeed being called on different threads by looking at the Thread ID. I'd say it's a great success!

# That's It!

That's the basics. In other articles, I'll talk about communication between threads and safely controlling reading and writing and executing things in a way that doesn't cause problems.

# References

https://doc.qt.io/qt-6/thread-basics.html
https://wiki.qt.io/QThreads_general_usage
https://doc.qt.io/qt-6/threads-technologies.html
https://doc.qt.io/qt-6/threads-synchronizing.html
https://doc.qt.io/qt-6/qthreadpool.html
https://doc.qt.io/qt-6/examples-threadandconcurrent.html
https://doc.qt.io/qt-6/threads.html