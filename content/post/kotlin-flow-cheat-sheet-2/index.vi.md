---
title: "Kotlin Flow cheat sheet phần 2: Flow"
description: "Kotlin Flow là một API mạnh mẽ giúp quản lý luồng dữ liệu bất đồng bộ một cách rõ ràng và dễ dàng. Trong phần 2 này, chúng ta sẽ khám phá Flow từ cơ bản đến nâng cao, tìm hiểu cách tạo, chuyển đổi và thu thập các luồng dữ liệu, cũng như các best practice để áp dụng trong dự án Android của bạn."
date: 2024-08-18T05:00:00+07:00
slug: kotlin-flow-cheat-sheet-2
image: cheat_sheet.jpg
toc: true
categories: Technical
tags: [Kotlin, Android, Coroutines, Flow]
---

*Photo by [Ana Cruz](https://unsplash.com/@anacruzbaeza?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/photographie-a-plat-de-papiers-dimprimante-blancs-S0qh0ONK-AE?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)*

Tiếp nối serie Kotlin cheat sheet, chúng ta cùng đi đến với cheat sheet tiếp theo dành cho **Flow**.

**Kotlin Flow** là một API mạnh mẽ giúp quản lý luồng dữ liệu bất đồng bộ một cách rõ ràng và dễ dàng. Trong phần 2 này, chúng ta sẽ khám phá Flow từ cơ bản đến nâng cao, tìm hiểu cách tạo, chuyển đổi và thu thập các luồng dữ liệu, cũng như các best practice để áp dụng trong dự án Android của bạn.

Bạn có thể đọc toàn bộ serie tại đây:

* [Kotlin Coroutines cheat sheet nâng cao dành cho Android Engineer](../kotlin-coroutines-cheat-sheet)
* [Kotlin Flow cheat sheet phần 1: Channel](../kotlin-flow-cheat-sheet-1)
* Kotlin Flow cheat sheet phần 2: Flow
* [Kotlin Flow cheat sheet phần 3: SharedFlow và StateFlow](../kotlin-flow-cheat-sheet-3)

# Flow

## Nguyên tắc chính

* Là một **cold stream**.
* Hỗ trợ sẵn structured concurrency.
* Tác vụ cuối cùng của flow được gọi là tác vụ **terminal** (`collect`, `first`… ).
* Một flow có thể có các tác vụ trung gian để sửa đổi flow (`map`, `onEach`, `flatMapLastest`… ).
* Terminal operation là suspend và yêu cầu một scope.
* Các Exception chưa được bắt sẽ ngay lập tức cancel một flow và `collect` sẽ throw lại Exception đó.
* Theo mặc định, context của flow sẽ lấy từ context mà `collect` được gọi.

## Kết hợp các flow với nhau

`merge`, `combine` và `zip` là các hàm trung gian cho phép chúng ta kết hợp 2 (hoặc nhiều) flow thành 1. Vậy điểm khác biệt chính giữa 3 hàm đó là gì?

### merge

* **Không sửa đổi** bất kỳ phần tử nào.
* Các phần tử được **emit ngay khi chúng được tạo ra**, chúng ta không đợi flow khác để tạo ra giá trị.
* Sử dụng nó khi bạn có **nhiều nguồn event sẽ tạo ra cùng một action**.

> flowA emit: 1
>
> flowB emit: 2
>
> flowA emit: 3
>
> merge(flowA, flowB) tạo ra 1, 2, 3

### zip

* **Kết hợp** các phần tử từ các flow khác nhau để **tạo ra giá trị mới**.
* Chúng ta cần một hàm để **xác định** cách các phần tử được **kết hợp** với nhau.
* Chúng ta cần **đợi mỗi flow emit một giá trị** để có thể tạo cặp.
* Các phần tử chỉ có thể là **một phần của một cặp**.
* Các phần tử còn lại **không có cặp sẽ bị mất**.

> flowA emit: 1
>
> flowB emit: 2
>
> flowA emit: 4
>
> flowA.zip(flowB) {fA, fB -> fA + fB } tạo ra 3 (1+2 = 3, còn 4 từ flowA sẽ bị loại bỏ)

### combine

* **Kết hợp** các phần tử từ các flow khác nhau để **tạo ra giá trị mới**.
* Chúng ta cần một hàm **xác định** cách các phần tử được **kết hợp** với nhau.
* Chúng ta cần **đợi flow chậm hơn** emit giá trị lần đầu tiên trước khi tạo phần tử mới.
* Khi một flow tạo ra một phần tử mới, nó sẽ **thay thế phần tử trước đó** và **một giá trị mới sẽ được emit ngay lập tức** (chúng ta không đợi mỗi flow emit một phần tử mới).

> flowA emit: 1
>
> flowB emit: 2
>
> flowA emit: 3
>
> flowA.combines(flowB) { fA, fB -> fA + fB } tạo ra 3 (1+2 = 3) rồi 5 (3+2 = 5, trong đó phần tử 3 đã thay thế cho 1 trước đó)

## Sự khác biệt giữa fold và scan

Cả `fold` và `scan` **kết hợp tất cả các giá trị** do một flow emit thành **một phần tử** bằng cách áp dụng thao tác kết hợp các giá trị lại với nhau.

* `fold` là một tác vụ **terminal**. Nó suspend cho đến khi flow hoàn thành và tạo ra giá trị cuối cùng
* `scan` là một tác vụ trung gian và tạo ra tất cả các giá trị trung gian

```kotlin
val myflow = flowOf(1, 2, 3, 4)
myFlow.fold(0) { acc, newElement -> acc + newElement } // tạo ra 10

 myFlow.scan(0) { acc, newElement -> acc + newElement } 
// tạo ra 1, 3 (1+2), 6 (3+3), 10 (6+4)
```

## flatMapConcat, flatMapMerge và flatMapLatest

* Chúng đều là những tác vụ trung gian
* Chúng biến đổi các phần tử được emit bởi flow ban đầu bằng cách áp dụng một flow khác lên phần tử đó và trả về một flow khác

```kotlin
myFlowA.flatMapConcat { fA -> myFlowB(fA) } // giá trị trả về do flow B tạo ra
```

### flatMapConcat

* Chuyển đổi từng giá trị được emit thành một flow và nối các flow kết quả **một cách tuần tự**.
* Emit hoàn toàn các giá trị từ inner flow đầu tiên trước khi bắt đầu flow tiếp theo.
* Use Case: khi bạn cần xử lý các flow bên trong **theo thứ tự**, không bị chồng chéo.

### flatMapMerge

* Chuyển đổi từng giá trị được emit thành một flow và hợp nhất các flow kết quả **một cách đồng thời**.
* Emit các giá trị từ tất cả các inner flow khi chúng có sẵn, có khả năng không theo thứ tự.
* Use Case: khi bạn muốn xử lý đồng thời các flow bên trong và **không quan tâm đến thứ tự** của các giá trị được emit.

### flatMapLatest

* Chuyển đổi từng giá trị được emit thành một flow, **hủy các flow trước đó** khi một giá trị mới đã được emit, và **emit các giá trị từ flow mới nhất**.
* Chỉ flow mới nhất được hoạt động và các giá trị của nó được emit. Các flow trước đó bị hủy bỏ.
* Use Case: khi bạn chỉ quan tâm đến **giá trị mới nhất** và muốn hủy các thao tác trước đó.

```kotlin
data class User(val id: Int, val name: String)
data class UserDetails(val userId: Int, val address: String)

fun fetchUserData(): Flow<User> = flow {
    emit(User(1, "Alice"))
    delay(500)
    emit(User(2, "Bob"))
    delay(500)
    emit(User(3, "Charlie"))
}

fun fetchUserDetails(userId: Int): Flow<UserDetails> = flow {
    delay(1000) // Giả lập network delay
    emit(UserDetails(userId, "$userId's address"))
}

// flatMapConcat
fetchUserData()
    .flatMapConcat { user ->
        fetchUserDetails(user.id)
    }
    .collect { userDetails ->
        println("flatMapConcat: ${userDetails}")
    }
// Mỗi thông tin user được trả về tuần tự.
// flatMapConcat: UserDetails(userId=1, address=1's address)
// flatMapConcat: UserDetails(userId=2, address=2's address)
// flatMapConcat: UserDetails(userId=3, address=3's address)

// flatMapMerge
fetchUserData()
    .flatMapMerge { user ->
        fetchUserDetails(user.id)
    }
    .collect { userDetails ->
        println("flatMapMerge: ${userDetails}")
    }
// Thông tin user có thể bị xen kẽ do trả về đồng thời.
// flatMapMerge: UserDetails(userId=1, address=1's address)
// flatMapMerge: UserDetails(userId=2, address=2's address)
// flatMapMerge: UserDetails(userId=3, address=3's address)

// flatMapLatest
fetchUserData()
    .flatMapLatest { user ->
        fetchUserDetails(user.id)
    }
    .collect { userDetails ->
        println("flatMapLatest: ${userDetails}")
    }
// Chỉ thông tin của user cuối cùng được trả về
// do user mới sẽ cancel fetch trước đó.
// flatMapLatest: UserDetails(userId=3, address=3's address)
```

## Chuyển đổi function thành Flow

```kotlin
val function = suspend {
    // đây là biểu thức lambda suspend
    // định nghĩa hàm ở đây
}

function.asFlow()
```

Hoặc

```kotlin
suspend fun myFunction(): Flow<T> {
    // định nghĩa hàm ở đây
}

::myFunction.asFlow()
```

## Tạo flow tạo ra các phần tử trước khi chúng ta subscribe

Hàm `channelFlow` tạo ra sự kết hợp giữa flow và channel. Nó tạo ra một hot stream data nhưng cũng implement Flow interface.

```kotlin
val myChannelFlow = channelFlow {
    val myData = // fetch dữ liệu tại đây
    send(myData)
} 

suspend fun fetchData() {
    myData.first()
}
```

## Sửa đổi context của Flow

```kotlin
myFlow.flowOn(Dispatchers.IO)

// Hoặc

myFlow.flowOn(CoroutineName( "NewName" ))
```

## Tránh lồng nhau khi khởi chạy flow

```kotlin
// thay vì
viewModelScope.launch {
    myFlow.collect()
}

// làm như này
myFlow.launchIn(viewModelScope)
```

Cảm ơn các bạn đã đọc đến đây, cùng chờ đón những phần tiếp theo nhé.

# Reference

* https://medium.com/@galou.minisini/advanced-kotlin-flow-cheat-sheet-for-android-engineer-cb8157d4f848
