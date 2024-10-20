---
title: "Kotlin Flow cheat sheet phần 1: Channel"
description: "Trong phần 1, chúng ta sẽ tìm hiểu chi tiết về Channel, cách thức hoạt động, và cách ứng dụng trong các trường hợp thực tế để giao tiếp giữa các coroutine một cách an toàn và hiệu quả."
date: 2024-08-18T04:00:00+07:00
slug: kotlin-flow-cheat-sheet-1
image: cheat_sheet.jpg
toc: true
categories: Technical
tags: [Kotlin, Android, Coroutines, Flow]
---

*Photo by [Ana Cruz](https://unsplash.com/@anacruzbaeza?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/photographie-a-plat-de-papiers-dimprimante-blancs-S0qh0ONK-AE?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)*

Sau khi làm việc với **Kotlin Flows** một thời gian, có thể bạn đã quen với các khái niệm cơ bản. Nhưng nếu chưa bao giờ sử dụng `Channel`, bạn sẽ không biết sự khác nhau giữa `merge`, `combine` và `zip`, hoặc có thể bạn chưa hiểu rõ `SharedFlow` và `StateFlow` cũng như cách sử dụng chúng.

Cheat sheet này hệ thống lại những kiến thức quan trọng mà mình đã góp nhặt được trong quá trình làm việc với **Kotlin Flow**. Nó được thiết kế để trở thành một tài liệu tham khảo hữu ích, giúp bạn giải quyết các tình huống phức tạp.

Trong phần 1, chúng ta sẽ tìm hiểu chi tiết về Channel, cách thức hoạt động, và cách ứng dụng trong các trường hợp thực tế để giao tiếp giữa các coroutine một cách an toàn và hiệu quả.

Bạn có thể đọc toàn bộ serie tại đây:

* [Kotlin Coroutines cheat sheet nâng cao dành cho Android Engineer](../kotlin-coroutines-cheat-sheet)
* Kotlin Flow cheat sheet phần 1: Channel
* [Kotlin Flow cheat sheet phần 2: Flow](../kotlin-flow-cheat-sheet-2)
* [Kotlin Flow cheat sheet phần 3: SharedFlow và StateFlow](../kotlin-flow-cheat-sheet-3)

# Hot streams và Cold streams

## Hot streams

* Ví dụ: `channel`, `Collections` (`List`, `Set`… ).
* **Bắt đầu ngay lập tức:** bắt đầu emit các giá trị bất kể có subscriber hay không.
* **Lưu các phần tử:** chúng không cần phải tính toán lại và tất cả subscriber đều nhận được cùng một chuỗi giá trị.

## Cold streams

* Ví dụ: `Sequence`, `Flow`
* **Bắt đầu theo yêu cầu:** cold streams chỉ bắt đầu emit các giá trị khi subscriber chủ động đăng ký stream đó. Nguồn dữ liệu là lazy.
* **Phát độc lập:** mỗi subscriber nhận được chuỗi giá trị độc lập của riêng mình. Không có phần tử nào được lưu trữ.

# Channel

## Nguyên tắc chính

* Là một **hot stream**.
* Đảm bảo **không có xung đột** (không có vấn đề với trạng thái chia sẻ) và **công bằng** nên rất hữu ích khi các **coroutine** khác nhau cần **liên lạc với nhau**.
* Hỗ trợ **bất kỳ số lượng** sender và receiver.
* Mỗi giá trị gửi tới channel chỉ **được nhận một lần**.
* Nếu có nhiều receiver subscribe cùng lúc, các phần tử sẽ được **phân bổ công bằng** giữa các receiver. (Hàng đợi **FIFO** của receiver).

> Channel có 3 receiver, subscribe theo thứ tự:
> Receiver1, Receiver2, Receiver3.
>
> Tất cả receiver đã subscribe channel.
>
> Channel emit ra 4 giá trị: "A", "B", "C" rồi "D".
>
> Receiver1 nhận được "A" và "D"
>
> Receiver2 nhận được "B"
>
> Receiver3 nhận được "C"

* Chúng có 2 suspend function là `send` và `receive`.
* `receive` bị suspend nếu **không có phần tử nào** trong channel và sẽ đợi một phần tử sẵn sàng để tiếp tục.
* `send` bị suspend nếu channel **đạt đến capacity**.
* Chúng ta cũng có thể sử dụng phiên bản **không bị suspend** là `trySend` và `tryReceive`, chúng trả về một `ChannelResult` (cho chúng ta biết thao tác có thành công hay không).
* Chúng cần được **close thủ công** sau khi chúng ta gửi xong dữ liệu hoặc khi xảy ra Exception: `myChannel.close()`. Nếu không, receive sẽ đợi các phần tử mãi mãi.

## Các loại channel capacity

```kotlin
val myChannel = Channel<Int>(capacity = 3)

// hoặc

val myChannel = produce(capacity = 3) {
    // emit các giá trị ở đây
}
```

* `Channel.UNLIMITED`: buffer không giới hạn và `send` không bao giờ bị suspend.
* `Channel.BUFFERED`: buffer capacity là 64. Giá trị mặc định này có thể được override bằng thuộc tính hệ thống `kotlinx.coroutines.channels.defaultBuffer` trong JVM.
* `Channel.RENDEZVOUS`: (behavior mặc định) buffer capacity là 0. Receiver sẽ chỉ nhận được dữ liệu nếu nó đã subscribe với sender khi dữ liệu được emit.
* `Channel.CONFLATED`: buffer capacity là 1. Mỗi phần tử mới sẽ thay thế phần tử trước đó.
* Giá trị `int`bất kỳ: buffer sẽ có capacity bằng giá trị được set.

## Xử lý lỗi tràn buffer

Các channel có một tham số `onBufferOverflow` kiểm soát những gì xảy ra khi buffer đầy. Có 3 lựa chọn:

* `BufferOverflow.SUSPEND`: (behavior mặc định) tạm dừng phương thức `send` khi buffer đầy.
* `BufferOverflow.DROP_OLDEST`: loại bỏ phần tử cũ nhất khi buffer đầy.
* `BufferOverflow.DROP_LATEST`: loại bỏ phần tử mới nhất khi buffer đầy.

## Tạo Channel tự động close

Coroutine builder `produce` sẽ close channel bất cứ khi nào builder coroutine kết thúc (finish, stop hoặc cancel).

```kotlin
suspend fun myFunction() = coroutineScope {
    val channel = produce {
        // emit các giá trị ở đây và không cần gọi close() khi kết thúc
    }
}
```

## Tự động dọn dẹp nếu một phần tử không thể xử lý

Nếu channel đã bị close, cancel hoặc khi `send`, `receive`, `hastNext` có lỗi

```kotlin
val myChannel = Channel(
    capacity,
    onUnderliveredElement = { /* các tác vụ dọn dẹp ở đây */ }
)
```

## Use case: trigger một refresh

Trong Android, trường hợp sử dụng phổ biến cho các channel là trigger khi một screen được refresh (pull to refresh hoặc button retry). Đoạn code bên dưới trình bày cách fetch data từ API khi chúng ta subscribe flow lần đầu tiên hoặc khi trigger một refresh.

Rất nhiều người sử dụng `SharedFlow` để trigger refresh và nó hoạt động ổn, nhưng đó không phải là giải pháp tốt nhất vì `SharedFlow` được thiết kế để có nhiều receiver.

```kotlin
// Đây là phiên bản đơn giản hóa để minh họa cách chúng ta có thể sử dụng channel. 
// Trong trường hợp sử dụng thực tế, chúng ta sẽ yêu cầu một số logic bổ sung để tránh 
// làm mới nếu dữ liệu đã được tải chẳng hạn. 

interface ApiService {
    suspend fun fetchData(): List<String>
}

class FetchDataUseCase @Inject constructor (
    private val apiService: ApiService
) {
    // tạo một channel có buffer là 1 và sẽ loại bỏ dữ liệu mới nhất
    // vì vậy nếu chúng ta trigger refresh nhiều lần liên tiếp
    // chúng ta sẽ chỉ giữ phần tử đầu tiên.
    private val refreshChannel = Channel<Unit>(
        capacity = 1,
        onBufferOverflow = BufferOverflow.DROP_LATEST
    )

    // viewModel có thể receive flow này để build UI state
    val dataState: Flow<FetchDataState> =
        refreshChannel
            // convert channel thành flow
            .consumeAsFlow()
            // emit một phần tử khi bắt đầu fetch data ngay khi chúng ta subscribe flow
            .onStart { emit(Unit) }
            .map { fetchData() }

    fun refresh() {
        // Chúng ta sử dụng trySend ở đây để không phải tạo suspend function
        // và vì vậy chúng ta không cần scope để gọi nó.
        // Phương thức này có thể được gọi từ viewModel để trigger refresh
        refreshChannel.trySend(Unit)
    }
    private suspend fun fetchData(): FetchDataState =
        try {
            val data = apiService.fetchData()
            FetchDataState.Success(data)
        } catch (e: Exception) {
            FetchDataState.Error(e.message ?: "An error occurred")
        }

    sealed interface FetchDataState {
        data object Loading : FetchDataState
        data class Success(val data: List<String>) : FetchDataState
        data class Error(val message: String) : FetchDataState
    }
}
```

Cảm ơn các bạn đã đọc đến đây, cùng chờ đón những phần tiếp theo nhé.

# Reference

* https://medium.com/@galou.minisini/advanced-kotlin-flow-cheat-sheet-for-android-engineer-cb8157d4f848
