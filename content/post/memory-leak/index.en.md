---
title: "IT Doctor Diagnoses Common Memory Leak Cases in Android"
description: "In the previous article about Stack and Heap Memory in Java, I mentioned Memory Leaks as a cause of java.lang.OutOfMemoryError. Today, let's dive into specific examples that cause Memory Leaks to learn how to prevent and fix them."
date: 2025-05-11T11:00:00+07:00
slug: memory-leak
image: memory_leak.webp
toc: true
categories: [Technical, Android]
tags: [Memory Leak, Heap, Android, Java]
---

*Photo by [Patrick Hendry](https://unsplash.com/@worldsbetweenlines?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/brown-rocks-on-body-of-water-9wnabOhABno?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)*

In the previous article [**Stack and Heap Memory in Java**](../stack-heap), I mentioned **Memory Leaks** as a cause of `java.lang.OutOfMemoryError`. Today, let's dive into specific examples that cause Memory Leaks to learn how to prevent and fix them. Let's get started and diagnose each case like an IT doctor!

# Static Reference to Context

Sometimes, by mistake, you might declare an `Activity` or `Context` as a static variable. This keeps a reference to the `Activity` or `Context` in that static variable, preventing the **Garbage Collector** from reclaiming memory and leading to a memory leak. You can read more about how the **Garbage Collector** works in [this article](../stack-heap).

```kotlin
class MemoryLeakExample {
    companion object {
        var context: Context? = null  // Static reference to Context
    }

    fun initialize(context: Context) {
        this.context = context
    }
}
```

```kotlin
object MySingleton {
    var context: Context? = null
}
```

To fix this, use `applicationContext` instead of the `Activity` context.

```kotlin
class MemorySafeExample {
    companion object {
        var context: Context? = null
    }

    fun initialize(context: Context) {
        this.context = context.applicationContext  // Use applicationContext instead of Activity context
    }
}
```

Or, if you must store `context` in a singleton/static, use `WeakReference`:

```kotlin
object MySingleton {
    private var weakContext: WeakReference<Context>? = null

    fun setContext(context: Context) {
        weakContext = WeakReference(context.applicationContext)
    }

    fun getContext(): Context? = weakContext?.get()
}
```

# Inner Class (Non-Static)

Inner classes always carry an implicit reference to the outer class. So, if a `Handler`, `Runnable`, etc. in an inner class continues to exist after the `Activity` is destroyed, it can cause a memory leak.

```kotlin
class MainActivity : AppCompatActivity() {
    private val handler = Handler(Looper.getMainLooper())

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        handler.postDelayed({
            // Code that refers to activity
        }, 1000)
    }
}
```

To fix this, use `WeakReference` or a static inner class:

```kotlin
class MainActivity : AppCompatActivity() {
    private val handler = Handler(Looper.getMainLooper())

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val weakActivity = WeakReference(this)
        handler.postDelayed({
            weakActivity.get()?.let {
                // Safe usage of activity
            }
        }, 1000)
    }
}
```

# Forgetting to Unregister Listener

Forgetting to unregister a listener or `BroadcastReceiver` after an `Activity`/`Fragment` is destroyed can prevent the activity from being reclaimed by the **Garbage Collector**.

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var receiver: BroadcastReceiver

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        receiver = BroadcastReceiver { context, intent -> }
        registerReceiver(receiver, IntentFilter("com.example.MY_ACTION"))
    }

    // Forgot to unregister receiver
}
```

Always remember to unregister listeners/receivers in `onDestroy()`:

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var receiver: BroadcastReceiver

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        receiver = BroadcastReceiver { context, intent -> }
        registerReceiver(receiver, IntentFilter("com.example.MY_ACTION"))
    }

    override fun onDestroy() {
        super.onDestroy()
        unregisterReceiver(receiver)  // Unregister when activity is destroyed
    }
}
```

# Long-Running Async Task or Thread

Background tasks like `coroutine`, `thread`, or `Runnable` that continue running after the `Activity` is destroyed and still hold a reference to the old `Activity` are also a common cause of memory leaks.

Some solutions for this case:
* Cancel coroutine/async tasks in `onDestroy()`
* Use `lifecycleScope`/`viewModelScope` to automatically cancel tasks when the lifecycle ends

# Conclusion

Besides remembering and applying the best practices above, you can also use tools like **LeakCanary** or **Memory Profiler** in Android Studio to monitor and debug memory leaks. Understanding and avoiding memory leaks will help your Android app save RAM and run more smoothly.
