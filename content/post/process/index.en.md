---
title: "Everything About Process in Android"
description: "Processes are a fundamental but essential concept in Android. When we launch an application, by default, all components like Activity, Service, BroadcastReceiver, and ContentProvider run within a single Linux Process."
date: 2024-06-21T00:00:00+07:00
slug: process
image: process.jpg
toc: true
categories: Technical
tags: [Android, Process]
---

# Introduction

**Process** is a fundamental but essential concept in Android. When we launch an application, by default, all components like **Activity**, **Service**, **BroadcastReceiver**, and **ContentProvider** run within a single Linux Process unless we specify a separate process in the **AndroidManifest** file, as shown below:

```xml
<activity
    android:process="new_process_name"
    ...>
</activity>
```

By default, the process name matches the **app ID** declared in the `build.gradle` file. Both the application and the four main components have an `android:process` tag. Therefore, if you declare `android:process` for the `<application>` tag, that process name will apply to all components of that application.

# Priority Levels

We cannot manage Process lifetime directly. Android automatically calculates which **components** of running applications are active, their **importance** to the user, and the **remaining memory** to decide the Process lifetime.

When Android runs out of resources, it shuts down a Process, and naturally, the components running on that Process are destroyed as well. What determines which Process gets shut down?

Android prioritizes Processes based on their importance to the user. It classifies Processes into four priority levels:

## Foreground Process

This is the highest priority Process. It contains components the user is **actively interacting with**, such as:

* **Activity** at the top of the screen that the user is engaging with, where the `onResume()` method has been called.
* **BroadcastReceiver** running, with its `onReceive()` method currently executing.
* **Service** executing code in one of its callbacks: `onCreate()`, `onStart()`, or `onDestroy()`.

Only a few Processes like this exist in the system, and they are only killed when memory is so low that even these cannot continue running.

## Visible Process

This Process performs tasks that the user is **aware of**. If killed, it would impact the user experience. Examples include:

* **Activity** displayed on the screen but not in the foreground, where the `onPause()` method has been called. For example, an Activity partially covered by a dialog.
* **Foreground Service** running via the `startForeground()` method, making it visible to the user.
* A service running a feature visible to the user, such as a live wallpaper or keyboard.

## Service Process

This Process contains a Service running via the `startService()` method. The user does not see it directly but is aware of the **tasks it performs**, such as uploading or downloading data in the background.

A long-running Service (e.g., more than 30 minutes) may have its importance reduced to a cached state.

## Cached Process

These Processes are **no longer necessary**, and the system can **safely kill** them without hesitation when more resources are required.

An efficient system will have many Cached Processes to facilitate smooth app transitions and frequently kill Cached apps when needed.

Android uses **LRU Cache** (Least Recently Used Cache) to manage Cached Processes, prioritizing the removal of Processes least recently used.

In summary, understanding how components like **Activity**, **Service**, and **BroadcastReceiver** impact priority levels is crucial. Select the appropriate component for your use case to avoid a Process being killed during important tasks.

# Inter-Process Communication (IPC)

**Inter-Process Communication**, or IPC, is a mechanism that allows Processes to **communicate** and **synchronize their actions** in Android.

Each app runs in a separate Process, but many apps need to communicate with each other to **share data** or perform **collaborative tasks**, making IPC essential for safe and efficient inter-Process communication.

## Intent

**Intent** is the standard mechanism for asynchronous communication between **Activities** and **BroadcastReceivers**. Depending on the need, you can use `sendBroadcast`, `sendOrderedBroadcast`, or explicit intents.

## Android Interface Definition Language (AIDL)

**AIDL** is a tool for defining interfaces between Android applications. It enables apps to communicate safely and efficiently, regardless of the programming languages they are written in.

## Messenger

**Messenger** is a class in the Android SDK that allows applications to **send and receive messages**. It provides a simple interface for inter-application communication.

The main difference between AIDL and Messenger is that AIDL supports **concurrent tasks**, while Messenger is limited to **sequential tasks**.

## Broadcast Receiver

**BroadcastReceiver** handles asynchronous requests from Intents. By default, any app can call the receiver. If you intend to use BroadcastReceiver for a specific application, you can secure it by using the `<receiver>` tag in the AndroidManifest. This prevents unauthorized apps from sending Intents to the BroadcastReceiver.

# Reference

* https://developer.android.com/guide/components/processes-and-threads
* https://developer.android.com/guide/components/activities/process-lifecycle
* https://developer.android.com/privacy-and-security/security-tips#interprocess-communication
