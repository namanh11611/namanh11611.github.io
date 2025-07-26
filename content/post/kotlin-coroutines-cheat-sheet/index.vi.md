---
title: "Kotlin Coroutines cheat sheet nâng cao dành cho Android Engineer"
description: "Cheat sheet này hệ thống lại những kiến thức quan trọng mà mình đã góp nhặt được trong quá trình làm việc với Kotlin Coroutines. Nó được thiết kế để trở thành một tài liệu tham khảo hữu ích, giúp anh em giải quyết các trường hợp phức tạp của coroutine."
date: 2024-08-12T00:00:00+07:00
slug: kotlin-coroutines-cheat-sheet
image: cheat_sheet.webp
toc: true
categories: [Technical, Android]
tags: [Kotlin, Android, Coroutines]
---

*Photo by [Ana Cruz](https://unsplash.com/@anacruzbaeza?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/photographie-a-plat-de-papiers-dimprimante-blancs-S0qh0ONK-AE?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)*

Sau khi làm việc với **Kotlin Coroutines** một thời gian, có thể anh em đã quen với các khái niệm cơ bản như `suspend` function và các hàm `launch`, `async`..., có thể giải quyết các use case đơn giản một cách ngon ơ. Nhưng khi dự án trở nên phức tạp hơn, anh em có thể thường xuyên cần các giải pháp nâng cao hơn và phải nhờ sự trợ giúp đến từ Google hoặc AI.

Cheat sheet này hệ thống lại những kiến thức quan trọng mà mình đã góp nhặt được trong quá trình làm việc với **Kotlin Coroutines**. Nó được thiết kế để trở thành một tài liệu tham khảo hữu ích, giúp anh em giải quyết các trường hợp phức tạp của coroutine.

Bạn có thể đọc toàn bộ serie tại đây:

* Kotlin Coroutines cheat sheet nâng cao dành cho Android Engineer
* [Kotlin Flow cheat sheet phần 1: Channel](../kotlin-flow-cheat-sheet-1)
* [Kotlin Flow cheat sheet phần 2: Flow](../kotlin-flow-cheat-sheet-2)
* [Kotlin Flow cheat sheet phần 3: SharedFlow và StateFlow](../kotlin-flow-cheat-sheet-3)

# Các khái niệm trong Coroutines

[**Coroutine Context**](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html): tập hợp các thành phần khác nhau. Trong đó, các thành phần chính là **Job** và **Dispatcher** của coroutine.

[**Job**](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/): thứ có thể hủy được với vòng đời đạt đến đỉnh khi nó hoàn thành. Mỗi coroutine đều tạo một **Job** của riêng nó (đó là **coroutine context duy nhất** không được kế thừa từ coroutine cha).

**Dispatcher**: cho phép chúng ta quyết định **thread** nào (hoặc pool của thread) mà coroutine sẽ chạy trên đó (khi start và resume). Bạn có thể đọc bài viết chi tiết của mình về [Dispatchers trong Kotlin Coroutines](../kotlin-coroutines-dispatchers)

**Coroutine scope**: xác định thời gian tồn tại và context của coroutine. Nó chịu trách nhiệm quản lý vòng đời của coroutine, bao gồm cả việc hủy và xử lý lỗi.

**Coroutine builder**: các **extension function** của `CoroutineScope`, cho phép chúng ta start một coroutine bất đồng bộ (ví dụ như `launch`, `async`… ).

# Các quy tắc chính của Coroutines

* Bạn cần một `CoroutineScope` để start một coroutine (với function `launch` hoặc `async`). **`viewModelScope`** được sử dụng phổ biến nhất trong Android, nhưng bạn cũng có thể tự xây dựng scope của riêng bạn.
* **Coroutine con** (một coroutine bắt đầu từ một coroutine khác) **kế thừa** coroutine context từ **coroutine cha** (ngoại trừ **Job**).
* **Job** của **coroutine cha** được sử dụng làm **cha** của **Job** của **coroutine con**.
* **Coroutine cha suspend** cho đến khi tất cả các **coroutine con** của nó **kết thúc**.
* Khi một **coroutine cha** bị **hủy** thì tất cả các **coroutine con** của nó cũng bị **hủy**.
* Khi một **coroutine con** bị lỗi vì một Exception chưa được xử lý, nó sẽ **cancel coroutine cha** của nó (trừ khi bạn sử dụng một `SupervisorJob`).
* Bạn không nên sử dụng `GlobalScope`, nó có thể gây memory leak và giữ coroutine tồn tại ngay cả sau khi **Activity** hoặc **Fragment** khởi chạy nó đã bị bỏ qua.
* Bạn không nên truyền **coroutine scope** như một tham số, thay vào đó hãy sử dụng function `coroutineScope`.

# Các function của Coroutine scope

* `coroutineScope`: suspend function, dùng để bắt đầu một scope và trả về giá trị do tham số của function tạo ra.
* `supervisorScope`: tương tự `coroutineScope` nhưng nó override **Job** của context, vì vậy function không bị cancel khi coroutine con throw một Exception.
* `withContext`: tương tự `coroutineScope` nhưng cho phép thực hiện một số thay đổi trong scope (thường được sử dụng để set **Dispatcher**).
* `withTimeout`: tương tự `coroutineScope` nhưng đặt giới hạn thời gian cho phần body và nếu quá lâu sẽ bị hủy. Throw một `TimeoutCancellationException`.
* `withTimeoutOrNull`: tương tự `withTimeout` nhưng sẽ trả về `null` thay vì throw Exception khi hết thời gian.

# Chạy song song

Khi bạn muốn thực hiện hai tác vụ cùng lúc và đợi kết quả của cả hai trước khi trả về kết quả:

## Khi bạn có quyền truy cập vào một scope (ví dụ từ ViewModel)

```kotlin
suspend fun getConfigFromAPI(): UserConfig {
    // thực hiện lệnh gọi API tại đây hoặc bất kỳ suspend fun nào
}

suspend fun getSongsFromAPI(): List<Song> {
    // thực hiện lệnh gọi API tại đây hoặc bất kỳ suspend fun nào
}

fun getConfigAndSongs() {
    // scope có thể là bất kỳ scope nào bạn muốn, trường hợp điển hình sẽ là viewModelScope
    scope.launch {
        val userConfig = async { getConfigFromAPI() }
        val songs = async { getSongsFromAPI() }
        return Pair(userConfig.await(), songs.await())
    }
}
```

Giả sử bạn có API được phân trang và bạn muốn tải xuống tất cả các trang trước khi hiển thị chúng cho người dùng, nhưng bạn muốn tải song song tất cả các trang:

```kotlin
suspend fun getSongsFromAPI(page: Int): List<Song> {
    // thực hiện lệnh gọi API
}
const val totalNumberOfPages = 10

fun getAllSongs() {
    // scope có thể là bất kỳ scope nào bạn muốn, trường hợp điển hình là viewModelScope
    scope.launch {
        val allNews = (0 until totalNumberOfPages)
                  .map { page -> async { getSongsFromAPI(page) } }
                  .flatMap { it.await }
    }
}
```

> Lưu ý về `async`/`await`: coroutine sẽ được bắt đầu ngay lập tức khi nó được gọi. `async` trả về một object thuộc loại `Deferred<T>` (trong ví dụ của chúng ta là `Deferred<List<Song>>`). `Deferred` có suspend function `await` trả về giá trị khi nó sẵn sàng.

## Khi bạn không có quyền truy cập vào một scope (ví dụ từ một repository)

Từ repository hoặc use case của bạn, bạn muốn định nghĩa một coroutine sẽ bắt đầu song song 2 (hoặc nhiều) lệnh gọi. Vấn đề là bạn cần một scope để sử dụng `async` nhưng bạn không ở trong `viewModel` hoặc presenter nên bạn không có quyền truy cập vào scope của mình ở đây (hãy nhớ quy tắc của chúng ta là không nên truyền scope như một tham số).

Từ ví dụ ở trên, chúng ta sửa lại một chút như sau:

```kotlin
suspend fun getConfigAndSongs(): Pair<UserConfig, List<Song> = coroutineScope { 
    val userConfig = async { getConfigFromAPI() } 
    val songs = async { getSongsFromAPI()} 
    Pair(userConfig.await(), songs.await()) 
}
```

# Dọn dẹp khi Coroutine bị cancel

Nếu một coroutine bị hủy thì nó sẽ có trạng thái `cancelling` trước khi chuyển sang `cancelled`. Khi một coroutine bị hủy, chúng ta sẽ có thời gian để thực hiện một số tác vụ dọn dẹp nếu cần thiết (chẳng hạn như dọn dẹp local database hoặc gọi API để cho server biết rằng tác vụ không thành công).

Chúng ta có thể sử dụng `finally` để thực hiện một tác vụ:

```kotlin
viewModelScope.launch {
    try {
        // gọi một số suspend function tại đây
    } finally {
        // thực hiện tác vụ dọn dẹp tại đây
    }
}
```

Nhưng không được phép gọi suspend function trong quá trình dọn dẹp. Nếu bạn cần gọi suspend function, bạn sẽ cần phải làm như sau:

```kotlin
viewModelScope.launch {
    try {
        // gọi một số suspend function tại đây
    } finally {
        withContext(NonCancellable) {
            // thực hiện suspend function dọn dẹp tại đây
        }
    }
}
```

> Lưu ý: Việc cancel sẽ xảy ra tại điểm suspend đầu tiên. Vì vậy việc cancel sẽ không xảy ra nếu chúng không có bất kỳ suspend function nào.

# Dọn dẹp Coroutine khi hoàn thành

Tương tự như việc dọn dẹp khi một coroutine bị hủy, bạn có thể muốn thực hiện một thao tác khi coroutine đạt đến trạng thái cuối cùng (`completed` hoặc `cancelled`).

```kotlin
suspend fun myFunction() = coroutineScope {
    val job = launch { /* suspend function tại đây */ }
    job.invokeOnCompletion { exception: Throwable ->
        // do something here
    }
}
```

# Làm cách nào để KHÔNG cancel Coroutine khi một trong các phần tử con của nó bị lỗi

Bạn có thể sử dụng `SupervisorJob` và nó sẽ bỏ qua tất cả các exception ở con của nó.

## Tạo coroutine scope của bạn

```kotlin
val scope = CoroutineScope(SupervisorJob())
// nếu một coroutine mắc lỗi thì coroutine còn lại sẽ không bị hủy
scope.launch { myFirstCoroutine() }
scope.launch { mySecondCoroutine() }
```

## Sử dụng scope function

```kotlin
suspend fun myFunction() = supervisorScope {
    // nếu một coroutine xảy ra lỗi thì coroutine kia sẽ không bị hủy
    launch { myFirstCoroutine() } 
    launch { mySecondCoroutine() } 
}
```

## Bắt exception

```kotlin
suspend fun myFunction() {
    try {
        coroutineScope {
            launch { myFirstCoroutine() }
        }
    } catch (e: Exception) {
        // xử lý lỗi tại đây
    }
    try {
        coroutineScope {
            launch { mySecondCoroutine() }
        }
    } catch (e: Exception) {
        // xử lý lỗi tại đây
    }
}
```

`CancellationException` không truyền tới coroutine cha, chỉ coroutine hiện tại bị cancel. Có thể kế thừa `CancellationException` để tạo loại exception của riêng bạn, và nó cũng sẽ không truyền tới coroutine cha.

# Định nghĩa tác vụ mặc định trong trường hợp có exception

Chúng ta có thể sử dụng `CoroutineExceptionHandler`. Ví dụ, dùng để tự động đăng xuất người dùng khi server trả về lỗi 401.

```kotlin
val handler = CoroutineExceptionHandler { context, exception ->
    // định nghĩa tác vụ mặc định như hiển thị hộp thoại hoặc thông báo lỗi
}
val scope = CoroutineScope(SupervisorJob() + handler) 
scope.launch { /* gọi suspend function tại đây */ } 
scope.launch { /* gọi suspend function tại đây */ }
```

# Chạy một tác vụ không cần thiết

Nếu bạn muốn chạy một suspend function mà không ảnh hưởng đến các function khác (ví dụ nếu nó gây ra lỗi thì chỉ hàm này sẽ KHÔNG cancel coroutine, nhưng các hàm khác nếu gây ra lỗi thì vẫn sẽ cancel coroutine bình thường). Ví dụ điển hình là các function analytics.

```kotlin
val nonEssentialOperationScope = CoroutineScope(SupervisorJob())
suspend fun getConfigAndSongs(): Pair<UserConfig, List<Song> = coroutineScope {
    val userConfig = async { getConfigFromAPI() }
    val songs = async { getSongsFromAPI()}
    nonEssentialOperationScope.launch { /* tác vụ không cần thiết ở đây */ }
    Pair(userConfig.await(), songs.await())
}
```

Lý tưởng nhất là bạn nên inject `nonEssentialOperationScope` vào class để dễ test hơn.

# Chạy một tác vụ trên single thread để tránh các sự cố đồng bộ

```kotlin
suspend fun myFunction() = withContext(Dispatchers.Default.limiteParallelism(1)) {
    // suspend function tại đây
}
// Cũng có thể sử dụng Dispatchers.IO
```

## Các cách tiếp cận khác để tránh sự cố đồng bộ hóa với multithreading

Bạn có thể sử dụng `AtomicReference` (từ Java)

```kotlin
private val myList = AtomicReference(listOf( /* thêm object vào đây */ ))

suspend fun fetchNewElement() {
    val myNewElement = // fetch phần tử mới tại đây
    myList.getAndSet { it + myNewElement }
}
```

Hoặc với `Mutex`

```kotlin
val mutex = Mutex() 
private var myList = listOf( /* thêm object vào đây */ )

suspend fun fetchNewElement() {
    mutex.withLock {
        val myNewElement = // fetch phần tử mới tại đây
        myList += myNewElement
    }
}
```

# Tránh gửi lại một coroutine đến cùng một dispatcher

Tránh chi phí không cần thiết khi chuyển đổi dispatcher nếu chúng ta đã sử dụng `Dispatcher.Main`:

```kotlin
// điều này sẽ chỉ dispatch nếu cần thiết
suspend fun myFunction() = withContext(Dispatcher.Main.immediate) {
    // suspend fun tại đây
}
```

Hiện tại chỉ `Dispatchers.Main` hỗ trợ `immediate` dispatching.

Cảm ơn bạn đã đọc đến đây. Nếu bạn có kiến thức hay ho hoặc tip về Kotlin Coroutines, đừng ngần ngại comment chia sẻ với mình nhé!

# Reference

* https://medium.com/@galou.minisini/advanced-kotlin-coroutine-cheat-sheet-for-android-engineer-15e0d180fc1f
