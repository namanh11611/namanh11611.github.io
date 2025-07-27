---
title: "Dispatchers in Kotlin Coroutines"
description: "Simply put, a Dispatcher decides which thread a Coroutine will run on. It could be the main thread, a background thread, or it could push the Coroutine into a thread pool."
date: 2023-05-01T00:22:00+07:00
slug: kotlin-coroutines-dispatchers
image: dispatchers.webp
categories: [Technical, Android]
tags: [Kotlin, Android, Coroutines, Dispatchers]
---

# What is a Dispatcher?

Simply put, a Dispatcher decides which thread a Coroutine will run on. It could be the main thread, a background thread, or it could push the Coroutine into a thread pool.

There are 4 types of Dispatchers:

* `Dispatchers.Default`
* `Dispatchers.Main`
* `Dispatchers.IO`
* `Dispatchers.Unconfined`

Or you can create your own Dispatcher using `newSingleThreadContext()` or `newFixedThreadPoolContext()`.

Coroutine builder functions like `launch` and `async` have a `CoroutinesContext` parameter so you can pass in a `Dispatcher`, since all the above Dispatchers extend from `CoroutinesContext`.

```kotlin
launch(Dispatchers.Default) {
    println("I'm working in thread ${Thread.currentThread().name}")
}
```

If you call `launch` or `async` without passing a `CoroutinesContext`, it will inherit the context of the `CoroutineScope` it was launched in.

```kotlin
fun main() = runBlocking<Unit> {
    launch {
        println("I'm working in thread ${Thread.currentThread().name}")
    }
}
```

In this example, it inherits the context of `runBlocking`, so it runs on the main thread.

Now, let's look at each Dispatcher in detail.

# `Dispatchers.Default`

`Dispatchers.Default` is used by default by builder functions like `launch` and `async` if you don't assign another Dispatcher. Default uses a **shared background threads pool**. So you can use `Dispatchers.Default` for **CPU-intensive work**. For example:

* Heavy tasks like matrix calculations
* Sorting, filtering, or searching a huge list in memory
* Bitmap operations in memory
* Parsing JSON in memory

By default, the maximum number of threads used by `Dispatchers.Default` is **equal to the number of CPU cores**, but at least 2.

# `Dispatchers.Main`

You can guess from the name, right? Exactly, `Dispatchers.Main` runs on the **main thread**, suitable for **UI-related tasks**.

Usually, `Dispatchers.Main` is a **single thread**.

# `Dispatchers.IO`

Again, the name says it all. `Dispatchers.IO` uses a **shared pool of threads created as needed**. It helps offload blocking IO tasks. So it's suitable for **disk and network** operations. For example:

* Calling APIs
* Downloading files from a server
* Moving files between folders on disk
* Reading and writing files
* Querying databases
* Loading Shared Preferences

The number of threads used by `Dispatchers.IO` is limited to **64 or the number of cores** (whichever is greater).

# `Dispatchers.Unconfined`

I call `Dispatchers.Unconfined` the wild horse, because you never know which thread it will run on.

Initially, when launched, the Coroutine runs on the thread that called it. But after being suspended and resumed, it may run on a different thread, depending on the suspend functions called. `Dispatchers.Unconfined` is suitable for work that doesn't consume CPU and doesn't update the UI. But the Kotlin documentation emphasizes:

> The Unconfined dispatcher should not normally be used in code.

# `newSingleThreadContext`

This function creates a new thread for you to play with. But creating a new thread is resource-intensive, and you have to call `close` to release it when done. So in practice, I don't recommend using this.

There's also `newFixedThreadPoolContext` to create a thread pool with a fixed size.

# Comparison with RxJava, RxAndroid

You can see that Dispatchers are similar to Schedulers in RxJava.

| Coroutines            | RxJava/RxAndroid                 |
|-----------------------|----------------------------------|
| `Dispatchers.Default` | `Schedulers.computation()`       |
| `Dispatchers.Main`    | `AndroidSchedulers.mainThread()` |
| `Dispatchers.IO`      | `Schedulers.io()`                |

# Conclusion

In summary, Dispatchers are an important concept in Coroutines, so you need to understand them well to choose the right Dispatcher for each function.

# Reference

* https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html
* https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-default.html
* https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-main.html
* https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-i-o.html
* https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-unconfined.html
* https://amitshekhar.me/blog/dispatchers-in-kotlin-coroutines
