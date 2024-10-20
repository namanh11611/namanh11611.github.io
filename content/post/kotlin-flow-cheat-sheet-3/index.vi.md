---
title: "Kotlin Flow cheat sheet phần 3: SharedFlow và StateFlow"
description: "SharedFlow và StateFlow là hai loại flow đặc biệt trong Kotlin Flow, cung cấp các tính năng mạnh mẽ cho việc chia sẻ trạng thái và sự kiện giữa các thành phần khác nhau trong ứng dụng. Trong phần cuối của loạt bài viết này, chúng ta sẽ tìm hiểu sâu về cách sử dụng SharedFlow và StateFlow, những lợi ích của chúng, và cách tích hợp vào ứng dụng Android của bạn để xử lý luồng dữ liệu một cách hiệu quả và mượt mà hơn."
date: 2024-08-18T06:00:00+07:00
slug: kotlin-flow-cheat-sheet-3
image: cheat_sheet.jpg
toc: true
categories: Technical
tags: [Kotlin, Android, Coroutines, Flow]
---

*Photo by [Ana Cruz](https://unsplash.com/@anacruzbaeza?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/photographie-a-plat-de-papiers-dimprimante-blancs-S0qh0ONK-AE?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)*

Tiếp nối serie **Kotlin cheat sheet**, chúng ta cùng đi đến với cheat sheet cuối cùng dành cho **SharedFlow** và **StateFlow**.

**SharedFlow** và **StateFlow** là hai loại flow đặc biệt trong Kotlin Flow, cung cấp các tính năng mạnh mẽ cho việc chia sẻ trạng thái và sự kiện giữa các thành phần khác nhau trong ứng dụng. Trong phần cuối của loạt bài viết này, chúng ta sẽ tìm hiểu sâu về cách sử dụng SharedFlow và StateFlow, những lợi ích của chúng, và cách tích hợp vào ứng dụng Android của bạn để xử lý luồng dữ liệu một cách hiệu quả và mượt mà hơn.

Bạn có thể đọc toàn bộ serie tại đây:

* [Kotlin Coroutines cheat sheet nâng cao dành cho Android Engineer](../kotlin-coroutines-cheat-sheet)
* [Kotlin Flow cheat sheet phần 1: Channel](../kotlin-flow-cheat-sheet-1)
* [Kotlin Flow cheat sheet phần 2: Flow](../kotlin-flow-cheat-sheet-2)
* Kotlin Flow cheat sheet phần 3: SharedFlow và StateFlow

# SharedFlow

## Nguyên tắc chính

* Là một **hot stream**.
* Có thể có nhiều receiver và tất cả chúng sẽ nhận được cùng một giá trị.
* Hữu ích khi bạn cần truyền các giá trị tới nhiều consumer hoặc muốn chia sẻ trạng thái/sự kiện giữa các phần khác nhau trong ứng dụng của mình.
* Không bao giờ hoàn thành cho đến khi chúng ta close toàn bộ scope.
* Có phiên bản có thể thay đổi `MutableSharedFlow` cho phép chúng ta cập nhật state bằng cách emit các giá trị mới với suspend function `emit`.
* Chúng ta cũng có thể sử dụng phiên bản non suspend `tryEmit`.
* Hỗ trợ cấu hình replay và tràn buffer.
* Tất cả các phương thức của shared flow đều thread-safe và có thể được gọi một cách an toàn từ các coroutine đồng thời mà không cần đồng bộ hóa bên ngoài.

## Các tham số cấu hình

Kotlin đang cung cấp cho chúng ta một phương thức hữu ích để tạo `MutableSharedFlow` và xác định cách chúng ta muốn buffer hoạt động:

```kotlin
public fun <T> MutableSharedFlow(
    // số lượng giá trị được replayed cho subscriber mới
    replay: Int = 0,
    // số lượng giá trị được lưu vào buffer ngoài `replay`
    extraBufferCapacity: Int = 0,
    // hành động khi tràn buffer
    // Các giá trị: SUSPEND, DROP_OLDEST, DROP_LATEST
    onBufferOverflow: BufferOverflow = BufferOverflow.SUSPEND
): MutableSharedFlow<T>
```

## shareIn

* Biến đổi `Flow` thành `SharedFlow`.
* Hữu ích khi chúng ta muốn biến một flow thành nhiều flow
* Yêu cầu coroutine scope làm tham số đầu tiên (scope) để bắt đầu coroutine và collect phần tử của flow.
* Tham số thứ hai `started` xác định thời điểm SharedFlow sẽ bắt đầu listen giá trị do flow emit. Nó lấy một object `SharingStarted`.
* Tham số thứ ba, `replay`, (mặc định là 0) xác định số lượng giá trị được replay cho subscriber mới.

### `SharingStarted` option

* `SharingStarted.Eagerly`: bắt đầu listen các phần tử ngay lập tức và không bao giờ dừng lại cho đến khi scope bị cancel.
* `SharingStarted.Lazily`: bắt đầu listen khi subscriber đầu tiên xuất hiện và không bao giờ dừng cho đến khi scope bị cancel.
* `SharingStarted.WhileSubscribed()`: bắt đầu listen khi subscriber đầu tiên xuất hiện và dừng ngay khi subscriber cuối cùng biến mất. Chúng ta config delay (tính bằng mili giây) giữa thời điểm subscriber cuối cùng biến mất và thời điểm dừng coroutine bằng tham số `stopTimeoutMillis`.

Lưu ý về `WhileSubscribed`: nếu bạn mở một Intent mới từ màn hình của mình, chẳng hạn như ứng dụng máy ảnh, màn hình của bạn sẽ bị tạm dừng và do đó SharedFlow của bạn sẽ không còn subscriber nữa và sẽ ngừng emit. Khi quay lại màn hình ban đầu, bạn sẽ subscribe lại màn hình của mình và có thể chạy lại tác vụ bên trong flow của mình. Điều này có thể gây ra sự cố hoặc trigger lại một tác vụ không cần thiết.

Lưu ý về `SharingStarted.Eagerly` và `SharingStarted.Lazily`: nếu bạn đang sử dụng `ViewModelScope` hoặc `LifecycleScope` thì `SharedFlow` sẽ ngừng gửi các phần tử khi màn hình bị destroy.

## Biến flow thành SharedFlow

```kotlin
// từ một viewModel hoặc một class có lifeCycleScope
myFlow.shareIn(
    scope = viewModelScope
    started = SharingStarted.Lazily
)

// từ một class không có lifeCycleScope (repository hoặc use case)

suspend fun myFunction() = coroutineScope {
    myFlow.shareIn(
        scope = this,
        started = SharingStarted.Lazily
    )
}
```

## Use case: Observe database thay đổi từ nhiều vị trí

Nếu bạn sử dụng **Room** cho cơ sở dữ liệu của mình thì bạn có thể đã biết rằng nó đã hỗ trợ Flow. Vì vậy, bạn có thể observe các thay đổi trong database của mình và nhận dữ liệu mới ngay khi có. Nhưng việc đọc dữ liệu từ disk có thể khá nặng. Nếu cần nhận dữ liệu ở nhiều màn hình, bạn có thể sử dụng `SharedFlow` để tránh phải fetch dữ liệu cho mọi màn hình.

Trong ví dụ này, mình sẽ trình bày cách để fetch một `UserSettings` một lần nhưng vẫn nhận được update trên nhiều màn hình:

```kotlin
// DAO đơn giản để fetch dữ liệu từ Room
@Dao
interface UserSettingsDao {
    // fetch tất cả user settings từ database và emit một flow
    @Query("SELECT * FROM user_settings")
    fun getAll(): Flow<List<UserSettings>>
}

class UserSettingsRepository @Inject constructor(
    private val dao: UserSettingsDao
) {
    // Chúng ta chỉ đọc từ DB một lần và tất cả receiver sẽ nhận được
    // data được tính toán ở đây.
    suspend fun getAll(): SharedFlow<List<UserSettings>> = coroutineScope {
        dao.getAll.shareIn(
            // truyền xuống scope
            scope = this,
            // chỉ bắt đầu emit khi chúng ta có receiver
            started = SharingStarted.Lazily,
            // replay phần tử mới nhất khi một receiver mới subscribe nó
            replay = 1
        )
    }
}
```

# StateFlow

## Nguyên tắc chính

* Hoạt động tương tự như a `SharedFlow` với tham số `replay` được đặt thành 1.
* Luôn chỉ lưu trữ một giá trị.
* Giá trị được lưu trữ có thể được truy cập bằng thuộc tính `value`.
* Chúng ta cần đặt giá trị ban đầu trong constructor.
* Sự thay thế hiện đại cho `LiveData`.
* Sẽ không emit phần tử mới nếu nó bằng phần tử trước đó.

## Thiết lập và đọc một giá trị

```kotlin
val state = MutableStateFlow("A") // giá trị ban đầu là A
state.value = "B"  // đặt giá trị thành B
state.value = "B"  // giá trị này sẽ không emit phần tử mới vì giá trị đã là B 
val myValue = state.value // đọc giá trị từ state, ở đây là "B"
```

## stateIn

* Chuyển đổi một flow thành một `StateFlow`.
* Cần xác định scope.
* Có 2 loại, một loại suspend và một loại không suspend

### stateIn suspend

* suspend cho đến khi phần tử đầu tiên của flow được emit và giá trị mới được tính toán

```kotlin
suspend fun myFunction() = coroutineScope {
    myFlow.stateIn(this)
}
```

### stateIn not suspend

* Yêu cầu một giá trị ban đầu trong tham số `initialValue` của nó.
* Tham số thứ hai của nó là `started` và mong đợi một phần tử `SharingStarted`.

```kotlin
myFlow.stateIn(
    scope = viewModelScope,
    started = SharingStarted.Lazily,
    initValue = "A"
)
```

## Use case: Emit data từ viewModel sang view

Đoạn code về cách chuyển flow thành `StateFlow` để emit state từ view model sang view mà đang observe:

```kotlin
class MyViewModel @Inject constructor(
    private val fetchDataUseCase: FetchDataUseCase
) : ViewModel() {
    val myState: StateFlow<MyState> =
        fetchDataUseCase.dataState
            .map {
                when (it) {
                    is FetchDataUseCase.FetchDataState.Loading -> MyState.Loading
                    is FetchDataUseCase.FetchDataState.Success -> MyState.Success(it.data)
                    is FetchDataUseCase.FetchDataState.Error -> MyState.Error(it.message)
                }
            }
            // chuyển flow thành state flow
            .stateIn(
                // đặt scope thành viewModel vì vậy chúng ta sẽ stop
                // listening khi viewModel bị destroy
                scope = viewModelScope,
                started = SharingStarted.WhileSubscribed(5_000),
                initialValue = MyState.Loading
            )
    
    sealed interface MyState {
        data object Loading : MyState
        data class Success(val data: List<String>) : MyState
        data class Error(val message: String) : MyState
    }
}

@Composable
fun MyScreen(viewModel = MyViewModel()) {
    val state = viewModel.myState.collectAsStateWithLifecycle()
    when (state) {
        is MyState.Loading -> // show loading view
        is MyState.Success -> // show success view
        is MyState.Error -> // show error view
    }
}
```

Cảm ơn bạn đã đồng hành cùng mình đến hết serie Kotlin cheat sheet này. Hy vọng những kiến thức hữu ích này sẽ giúp bạn tự tin hơn khi làm việc với Kotlin Coroutines và Flow.

# Reference

* https://medium.com/@galou.minisini/advanced-kotlin-flow-cheat-sheet-for-android-engineer-cb8157d4f848
