---
title: "Dispatchers trong Kotlin Coroutines"
description: "Hiểu một cách đơn giản, Dispatcher sẽ quyết định xem Coroutines được thực thi trên thread nào. Có thể là main thread, background thread, hoặc nó đẩy Coroutines vào thread pool."
date: 2023-05-01T00:22:00+07:00
image: dispatchers.jpeg
categories: Technical
tags: [Kotlin, Android]
---

## Dispatcher là cái gì?
Hiểu một cách đơn giản, Dispatcher sẽ quyết định xem Coroutines được thực thi trên thread nào. Có thể là main thread, background thread, hoặc nó đẩy Coroutines vào thread pool.

Có 4 loại Dispatchers:
* `Dispatchers.Default`
* `Dispatchers.Main`
* `Dispatchers.IO`
* `Dispatchers.Unconfined`

Hoặc bạn có thể tự tạo Dispatchers cho riêng mình bằng function `newSingleThreadContext()` hoặc `newFixedThreadPoolContext()`.

Các function để build Coroutines như `launch` và `async` đều có một param là `CoroutinesContext` để chúng ta truyền `Dispatchers` vào, vì mấy thằng `Dispatchers` ở trên đều extends từ `CoroutinesContext`.
```kotlin
launch(Dispatchers.Default) {
    println("I'm working in thread ${Thread.currentThread().name}")
}
```
Còn khi mà chúng ta gọi `launch` với `async` mà không truyền param `CoroutinesContext`, nó sẽ kế thừa context của `CoroutineScope` mà nó được khởi chạy.
```kotlin
fun main() = runBlocking<Unit> {
    launch {
        println("I'm working in thread ${Thread.currentThread().name}")
    }
}
```
Ở trong ví dụ này, nó sẽ kế thừa context của `runBlocking` nên sẽ thực thi trên main thread.

Bây giờ, chúng ta sẽ đi tìm hiểu chi tiết từng loại Dispatchers nhé.
## `Dispatchers.Default`
`Dispatchers.Default` được dùng mặc định bởi các builder function `launch` và `async` nếu như chúng ta không gán Dispatchers nào khác cho nó. Default Dispatchers dùng một **shared background threads pool**. Vì vậy chúng ta có thể dùng `Dispatchers.Default` cho những công việc **tốn nhiều CPU**. Ví dụ:
* Các tác vụ nặng như tính toán ma trận
* Sort, filter hoặc search một cái list siêu to khổng lồ trên bộ nhớ
* Các tác vụ với Bitmap trên bộ nhớ
* Parse JSON trên bộ nhớ

Mặc định, số lượng thread nhiều nhất được `Dispatchers.Default` dùng sẽ **bằng với số CPU core**, nhưng ít nhất là 2.
## `Dispatchers.Main`
Bạn nghe tên là đoán được rồi đúng không? Chính xác, `Dispatchers.Main` sẽ thực thi trên **main thread**, nó phù hợp với các tác vụ **tương tác với UI**.

Thông thường thì `Dispatchers.Main` sẽ là **single thread**.
## `Dispatchers.IO`
Nghe nhạc hiệu đoán được chương trình tiếp này. `Dispatchers.IO` dùng một **shared pool gồm các thread được tạo theo nhu cầu**. Nó giúp giảm tải các tác vụ blocking IO. Vì vậy nó phù hợp với các tác vụ liên quan đến **disk và network**. Ví dụ:
* Gọi API
* Download file từ server
* Move 1 file từ folder này đến folder khác trên disk
* Đọc ghi file
* Query database
* Loading Shared Preferences

Số lượng thread được sử dụng bởi `Dispatchers.IO` được giới hạn bởi **64 hoặc số lượng core** (tuỳ xem số nào lớn hơn).
## `Dispatchers.Unconfined`
Mình gọi `Dispatchers.Unconfined` là con ngựa bất kham, vì mình sẽ không biết nó được thực thi trên thread nào.

Ban đầu, khi khởi chạy, Coroutines sẽ được thực thi trên chính thread gọi nó. Nhưng sau khi bị suspend, và resume, nó sẽ thực thi trên một thread khác, được quyết định bởi suspend functions được gọi. `Dispatchers.Unconfined` phù hợp với những công việc không tiêu tốn CPU và không update UI. Nhưng bản thân Kotlin document cũng nhấn mạnh là:
> The Unconfined dispatcher should not normally be used in code.
> > **Tạm dịch:** Bình thường không ai dùng dispatcher Unconfined trong code cả.
## `newSingleThreadContext`
Function này sẽ tạo một thread mới cho chúng ta tha hồ vùng vẫy. Nhưng thật sự thì việc tạo một thread mới tốn rất nhiều resource, và bạn phải tự gọi function `close` để giải phóng nó khi không dùng nữa. Vậy nên trong thực tế, mình khuyến nghị các bạn không nên dùng cách này.

Ngoài ra còn có `newFixedThreadPoolContext` để tạo một thread pool với size cố định.
## So sánh với RxJava, RxAndroid
Chúng ta có thể thấy Dispatchers tương tự như Schedulers trong RxJava.
| Coroutines | RxJava/RxAndroid |
| -------- | -------- |
| `Dispatchers.Default` | `Schedulers.computation()` |
| `Dispatchers.Main` | `AndroidSchedulers.mainThread()` |
| `Dispatchers.IO` | `Schedulers.io()` |

## Lời kết
Tựu chung lại, Dispatchers là một khái niệm quan trọng trong Coroutines, vậy nên các bạn cần nắm chắc về nó để có thể chọn một Dispatchers phù hợp cho từng function của mình.

## Reference
* https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html
* https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-default.html
* https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-main.html
* https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-i-o.html
* https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-unconfined.html
* https://amitshekhar.me/blog/dispatchers-in-kotlin-coroutines
