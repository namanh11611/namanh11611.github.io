---
title: "Bác sỹ IT bắt bệnh những trường hợp gây Memory Leak trong Android"
description: "Trong bài viết trước Bộ nhớ Stack và Heap trong Java, mình đã nhắc đến Memory Leaks như là một phần gây ra lỗi java.lang.OutOfMemoryError. Vậy thì hôm nay chúng ta sẽ dành thời gian tìm hiểu những ví dụ cụ thể gây ra Memory Leaks để biết cách phòng bệnh và chữa bệnh."
date: 2025-05-11T11:00:00+07:00
slug: memory-leak
image: memory_leak.webp
toc: true
categories: [Technical, Android]
tags: [Memory Leak, Heap, Android, Java]
---

*Photo by [Patrick Hendry](https://unsplash.com/@worldsbetweenlines?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/brown-rocks-on-body-of-water-9wnabOhABno?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)*

Trong bài viết trước [**Bộ nhớ Stack và Heap trong Java**](../stack-heap), mình đã nhắc đến **Memory Leaks** như là một phần gây ra lỗi `java.lang.OutOfMemoryError`. Vậy thì hôm nay chúng ta sẽ dành thời gian tìm hiểu những ví dụ cụ thể gây ra Memory Leaks để biết cách phòng bệnh và chữa bệnh. Nào, xin mời các bác sỹ IT bắt tay vào hội chẩn từng ca bệnh một.

# Static Reference đến Context

Trong quá trình code, đôi khi do sơ suất mà bạn khai báo `Activity` hoặc `Context` như một biến static. Việc này sẽ khiến chúng ta giữ một reference đến `Activity` hoặc `Context` trong biến static đó, làm cho **Garbage Collector** không thể thu hồi bộ nhớ và dẫn đến memory leak. Bạn có thể đọc lại về cơ chế hoạt động của **Garbage Collector** trong [bài viết này](../stack-heap).

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

Để khắc phục, bạn có thể sử dụng `applicationContext` thay vì `context` của `Activity`.

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

Hoặc nếu bắt buộc phải lưu `context` trong singleton/static, bạn có thể sử dụng `WeakReference`.

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

Inner class luôn mang theo reference ngầm đến outer class. Do đó, nếu `Handler`, `Runnable`... trong inner class tiếp tục tồn tại sau khi `Activity` bị hủy thì sẽ dẫn đến memory leak.

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

Cách khắc phục cũng tương tự như phần trên, đó là chúng ta có thể dùng `WeakReference` hoặc static inner class.

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

# Quên unregister listener

Việc quên unregister listener hoặc `BroadcastReceiver` sau khi `Activity`/`Fragment` huỷ có thể làm cho activity không được **Garbage Collector** thu hồi.

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

Vì vậy bạn hãy nhớ luôn luôn unregister listener/receiver trong `onDestroy()`

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

# Long-Running Async Task hoặc Thread

Các background task như `coroutine`, `thread` hoặc `Runnable` tiếp tục chạy sau khi `Activity` bị huỷ và còn giữ tham chiếu đến `Activity` cũ cũng là nguyên nhân thường gặp gây memory leak.

Chúng ta có một số giải pháp cho trường hợp này như:
* Hãy cancel coroutine/async task trong onDestroy()
* Dùng lifecycleScope/viewModelScope để tự động cancel task khi lifecycle kết thúc

# Lời kết

Ngoài việc ghi nhớ và áp dụng những best practice ở trên, bạn cũng có thể sử dụng công cụ hỗ trợ như **LeakCanary** hoặc **Memory Profiler** trong Android Studio để theo dõi và debug memory leak. Việc hiểu đúng và tránh memory leak sẽ giúp app Android của bạn tiết kiệm RAM và chạy ổn định hơn.
