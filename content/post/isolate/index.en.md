---
title: "Isolate in Flutter: The Savior for Heavy Tasks"
description: "As you may know, all Dart code runs in isolates, starting from the default main isolate. When developing Flutter applications, handling heavy tasks like image processing or complex calculations on this main isolate will cause lag and stutter. The Isolate API was born to solve this problem, allowing us to offload heavy tasks to background workers and restore the app's smoothness."
date: 2025-05-25T00:00:00+07:00
slug: isolate
image: isolate.webp
toc: true
categories: [Technical, Flutter]
tags: [Flutter, Isolate]
---

As you may know, all **Dart** code runs in **isolates**, starting from the default **main isolate**. When developing Flutter applications, handling heavy tasks like image processing or complex calculations on the **main isolate** can cause **lag and stutter**. Although Flutter provides **Future** and **Stream**, they are still processed on the **main isolate**. The **Isolate API** was created to solve this problem, allowing us to offload heavy tasks to **background workers** and restore the app's smoothness. In this article, I will explain how to use **Isolate** and provide best practices for working with it.

# What is an Isolate?

**Dart** is a **single-threaded** language. When you use **async** and **await**, it actually runs **concurrently**. If a time-consuming task runs on the main thread, it will **block the entire UI**. **Isolate** is the solution for **parallelism**.

An **Isolate** is an **independent execution thread** in Dart. Each isolate has its own **Heap** memory, ensuring that no isolate can access another's memory, which helps the app run smoothly without shared state.

Let's jump into code for a clearer picture. Here’s a simple example of creating an Isolate:

```dart
import 'dart:isolate';

void main() {
  Isolate.spawn(isolateMethod, "Hello World from Main Isolate");
  print('Main isolate');
}

void isolateMethod(String message) {
  print('New isolate: $message');
}
```

In the example above, the `main()` function runs on the **main isolate**, often called the **UI isolate**. We create a **new isolate** by calling `Isolate.spawn()`, which executes the `isolateMethod()`. These worker isolates cannot access the UI.

# Communication Between Isolates

To exchange data between isolates, Dart provides two classes: `SendPort` and `ReceivePort`, which allow isolates to communicate via messages. Read the following code; I’ll explain it in detail below.

```dart
void main() async {
  final mainReceivePort = ReceivePort();
  final isolate = await Isolate.spawn(twoWayIsolate, mainReceivePort.sendPort);

  // Receive SendPort from the worker isolate
  mainReceivePort.listen((message) {
    if (message is SendPort) {
      print('Main: SendPort');
      message.send('Hello');
    } else {
      print('Main received: $message'); // Output: Hello to Henry
      isolate.kill();
    }
  });
}

void twoWayIsolate(SendPort mainSendPort) {
  final receivePort = ReceivePort();

  // Send the worker isolate's SendPort back to main
  mainSendPort.send(receivePort.sendPort);

  receivePort.listen((message) {
    if (message is String) {
      print('Worker received: $message'); // Output: Hello
      final newMessage = '$message to Henry';
      mainSendPort.send(newMessage);
    }
  });
}
```

In this example, we set up two-way communication between the main isolate and the worker isolate:

```dart
// Send the worker isolate's SendPort back to main
mainSendPort.send(receivePort.sendPort);
```

As soon as the main isolate receives a `SendPort` message, it sends a greeting to the worker isolate:

```dart
message.send('Hello');
```

The worker isolate responds by sending a greeting back to the main isolate:

```dart
final newMessage = '$message to Henry';
mainSendPort.send(newMessage);
```

As you can see, `SendPort.send()` is responsible for sending messages, while listening for messages is done via `ReceivePort.listen()`.

# Using `compute` for Simple Tasks

Flutter provides the `compute` function to run a function in a new isolate with just one line of code:

```dart
Future<void> fetchData() async {
  final result = await compute(_heavyProcessing, 1000000);
  print('Result: $result');
}

// This function will run in its own isolate
int _heavyProcessing(int iterations) {
  int sum = 0;
  for (int i = 0; i < iterations; i++) {
    sum += i;
  }
  return sum;
}
```

The advantage of `compute` is that it **automatically manages** the isolate, which is **destroyed** as soon as it finishes.

However, there are limitations: `compute` only accepts **one parameter**. If you need more, you must package them into a Map or List. Also, `compute` cannot do two-way communication like `Isolate.spawn` in the previous example.

# Best Practices

## Only use isolates for CPU-intensive tasks

For network requests, you can use a regular HTTP client. Only use isolates for CPU-heavy tasks like image processing.

```dart
// DO
compute(imageProcessing, imageData);

// DON'T
compute(networkRequest, url);
```

## Avoid creating isolates continuously

Creating new isolates repeatedly consumes CPU and memory, and initializing an isolate is relatively slow, taking about 30ms each time. Therefore, using a **Worker Pool** helps reuse existing isolates.

## Use simple data types when passing through ports

Since isolates do not share memory, data passed between isolates must be **serialized/deserialized**. Using complex data types increases conversion time and memory usage when copying data.

```dart
// Recommended
sendPort.send({'id': 1, 'value': 42});

// Not recommended
sendPort.send(MyComplexClass());
```

## Stop isolates when not needed

When using `Isolate.spawn`, it does not automatically release resources when unused. Leaving isolates running in the background wastes CPU/memory and can cause memory leaks. Use `Isolate.kill` to stop them when done.

```dart
isolate.kill(priority: Isolate.immediate);
```

# Conclusion

Isolate is a powerful tool for handling heavy tasks in Flutter without affecting the UI. Although it has some complexity, by using it correctly with `compute` and supporting libraries, you can create smooth apps even when handling complex business logic.

# Reference
* [Concurrency in Dart](https://dart.dev/language/concurrency)
* [Isolates](https://dart.dev/language/isolates)
