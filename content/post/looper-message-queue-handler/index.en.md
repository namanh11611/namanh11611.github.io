---
title: "All About Looper, MessageQueue, and Handler in Android"
description: "In Android, performing heavy tasks like network requests or database operations on the main thread can cause the app to freeze or crash. To ensure smooth performance, such tasks need to be offloaded to a background thread to avoid blocking the main thread."
date: 2024-10-10T00:00:00+07:00
slug: looper-message-queue-handler
image: looper_message_queue_handler.jpg
toc: true
categories: Technical
tags: [Looper, MessageQueue, Handler, Thread, Android, Java]
---

*Photo by [Etienne Girardet](https://unsplash.com/@etiennegirardet?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/a-pile-of-black-and-white-wires-and-a-cassette-OA0qcP6GOw0?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)*

In Android, performing heavy tasks like **network requests** or **database operations** on the **main thread** can cause the app to freeze or crash. To ensure smoother app performance, these tasks should be executed on a **background thread** to avoid blocking the **main thread**. For instance, when a user clicks the Submit button on the main thread, the network request will be processed on a background thread, and the result will be sent back to the main thread. Android provides tools such as **Looper**, **MessageQueue**, and **Handler** to manage concurrent tasks and handle message passing between threads.

Wait a second! Isn’t Kotlin Coroutines already solving these problems? That’s true; nowadays, Kotlin Coroutines are widely used for such use cases. However, in certain projects, for example Android Automotive, the trio of Looper, MessageQueue, and Handler is still essential.

In this article, I’ll explain the role and responsibilities of each component and how they interact with one another. Since these components are closely intertwined, you might need to refer back to earlier sections to fully grasp the concepts. Take your time!

# Looper

**Looper** is a class that manages the **message loop** for a **thread**, with each thread having **exactly one** Looper. If we dive into the Android SDK code, we can see the Looper class defined as follows:

```java
public final class Looper {
    // Looper contains the MessageQueue
    final MessageQueue mQueue;
    // The relationship between Looper and Thread is one-to-one
    final Thread mThread;
    
    public static void prepare() {
        prepare(true);
    }
    
    private static void prepare(boolean quitAllowed) {
        // Each thread can have only one Looper
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
    
    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
}
```

By default, threads are **not associated** with any message loop. To create a message loop, we need to call the `Looper.prepare()` method, as shown above. Then, we invoke the `Looper.loop()` method to process messages until the loop stops.

Here’s an example of a Looper implementation:

```java
class LooperExampleThread extends Thread {
    public void run() {
        Looper.prepare();
        Looper.loop();
    }
}
```

# MessageQueue

**MessageQueue** is a data structure that holds a list of **Message** and **Runnable** objects to be processed. It operates on a **FIFO** (First-In-First-Out) basis. You can access the MessageQueue of the current thread using the `Looper.myQueue()` method.

In the Looper code above, you’ll notice that each Looper has exactly one MessageQueue:

```java
public final class Looper {
    // Looper contains the MessageQueue
    final MessageQueue mQueue;
}
```

Messages are not added directly to the MessageQueue. Instead, they are added **through a Handler**. The Looper continuously extracts and processes messages from the queue.

# Handler

**Handler** is a class used to send and process **Message** and **Runnable** objects linked to a thread’s **MessageQueue**. Each Handler is associated with **a specific thread and its MessageQueue**.

When creating a Handler, you must pass a Looper to its **constructor**. Here’s an example of a typical constructor:

```java
public Handler(@NonNull Looper looper) {}
```

The **MessageQueue** we mentioned earlier belongs to the **Looper** passed here, and messages will be executed on the same thread as that **Looper**.

Some commonly used methods of Handler include:

* `post(Runnable)`
* `postAtTime(java.lang.Runnable, long)`
* `postDelayed(Runnable, Object, long)`
* `sendEmptyMessage(int)`
* `sendMessage(Message)`
* `sendMessageAtTime(Message, long)`
* `sendMessageDelayed(Message, long)`

Handler serves two main purposes:

* **Scheduling tasks** to run in the future. You can use methods like `...AtTime` or `...Delayed`.
* Executing tasks on a **different thread** from the current one. As mentioned earlier, you can specify the desired thread by passing its Looper when initializing the Handler.

# Communication Between Components

![Looper, MessageQueue, Handler](communication.webp)

* When a **Message** or **Runnable** is sent via a **Handler**, it is added to the **MessageQueue**.
* The **Looper** continuously checks the **MessageQueue** for new messages.
* Upon finding a message, the **Looper** extracts it from the queue and forwards it to the corresponding **Handler** for processing.
* The **Handler** processes the message on the thread it is associated with.

Here’s an illustrative code example:

```java
class ProcessingThread extends Thread {
    public Handler mHandler;
    
    public void run() {
        Looper.prepare();
        
        // Looper.myLooper() return the Looper object
        // associated with the current thread
        mHandler = new Handler(Looper.myLooper()) {
            public void handleMessage(Message msg) {
                // Process received messages
            }
        };
        
        Looper.loop();
    }
}

// ClientThread has a reference to the mHandler object
// of ProcessingThread
class ClientThread extends Thread {
    private void sendMessageExample() {
        Message msg = Message.obtain(mHandler, 1);
        msg.obj = "New message";
        mHandler.sendMessage(msg);
    }
    
    private void sendRunnableExample() {
        mHandler.post(new Runnable() {
            @Override
            public void run() {
                // Task executed on ProcessingThread
            }
        });
    }
}
```

## HandlerThread

In practice, developers rarely create and manage threads and Loopers manually. Android provides **HandlerThread**, a special type of thread with a built-in Looper property. You can retrieve its Looper using the `getLooper()` method.

```java
HandlerThread thread = new HandlerThread("ProcessingThread");
thread.start();
Looper looper = thread.getLooper();
Handler handler = new Handler(looper);
```

## Main Thread

The **Main thread** (UI thread) in Android already has a built-in Looper, which can be accessed via `Looper.getMainLooper()`. A common example is creating a Handler to delay a task on the UI thread, as follows:

```java
Handler handler = new Handler(Looper.getMainLooper());
handler.postDelayed(new Runnable() {
    @Override
    public void run() {
        // Delayed task
    }
}, 3000);

```

# Conclusion

**Looper**, **MessageQueue**, and **Handler** are essential components of Android’s asynchronous processing system. They work together to facilitate efficient and safe inter-thread communication. Understanding their operations can help you build robust Android applications. Thank you for reading!

# Reference

* https://developer.android.com/reference/android/os/Looper
* https://developer.android.com/reference/android/os/MessageQueue
* https://developer.android.com/reference/android/os/Handler
