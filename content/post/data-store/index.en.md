---
title: "DataStore - mảnh ghép hoàn hảo cho bức tranh Kotlin Coroutines"
description: "DataStore được tạo ra chính là để thay thế SharedPreferencs lưu trữ những dữ liệu đơn giản."
date: 2023-05-14T15:41:00+07:00
image: datastore.jpeg
categories: Technical
tags: [Kotlin, Android]
---

# Concept
Trước hết, chúng ta cần hiểu **DataStore** sinh ra với mục đích là gì.

Hiện tại, trong ứng dụng Android, chúng ta có 5 cách để lưu trữ dữ liệu, trong đó **SharedPreferences** là cách dùng để lưu những dữ liệu đơn giản nhất. Nó chỉ gồm **key** và **value**, trong đó value có thể là integer, string...

![](https://images.viblo.asia/2cdc1cea-a8d5-444e-9003-5f88178503b0.png)

Khi lần đầu mở app, nó sẽ đọc toàn bộ giá trị trong file xml của SharedPrefrences và lưu vào RAM. Quá trình đọc file này lại diễn ra trên **UI Thread**, nếu chúng ta có rất rất nhiều giá trị khiến cho thời gian thực hiện tác vụ vượt quá 5 giây, nó sẽ gây ra lỗi **ANR** (Application Not Responding).

Và **DataStore** được tạo ra chính là để thay thế **SharedPreferencs**.

> DataStore là giải pháp lưu trữ dữ liệu theo dạng **cặp key-value** hoặc **typed objects với protocol buffers**.

Tất nhiên, DataStore vẫn chỉ dành để lưu những dữ liệu có cấu trúc đơn giản. Nó sử dụng Coroutines và Flow để lưu data một cách bất đồng bộ và nhất quán.

DataStore gồm 2 loại **Preferences DataStore** và **Proto DataStore**, chúng ta cùng nhìn qua bảng so sánh sau:

| Preferences DataStore | Proto DataStore |
| -------- | -------- |
| Lưu và truy cập data bằng key | Lưu instance của một loại custom data |
| Không yêu cầu định nghĩa trước loại data  | Phải định nghĩa trước loại data bằng protocol buffers |
| Không có type safety | Có type safety |

# Preferences DataStore
## Create
Để sử dụng Preferences DataStore, chúng ta cần tạo một instance `DataStore<Preferences>` bằng [property delegate](https://viblo.asia/p/design-pattern-delegation-trong-kotlin-cach-de-nho-nguoi-khac-lam-bai-tap-ve-nha-vyDZO3exZwj#_delegated-properties-4) với keyword `preferencesDataStore`.
```kotlin
// At the top level of your kotlin file
val Context.dataStore: DataStore<Preferences>
        by preferencesDataStore(name = "settings")
```

## Read
Trước hết, chúng ta có 7 function tương ứng với 7 loại data:
* `intPreferencesKey()`
* `longPreferencesKey()`
* `doublePreferencesKey()`
* `floatPreferencesKey()`
* `booleanPreferencesKey()`
* `stringPreferencesKey()`
* `stringSetPreferencesKey()`

Khi đọc data, chúng ta cần dùng function tương ứng với giá trị mà chúng ta cần lưu. Ví dụ để lưu một biến `counter` dạng số nguyên để đếm số lần user mở app, chúng ta có thể dùng cách sau:
```kotlin
val OPEN_APP_COUNTER = intPreferencesKey("open_app_counter")
val openAppCounterFlow: Flow<Int> = context.dataStore.data
  	.map { preferences ->
    	// No type safety.
    	preferences[OPEN_APP_COUNTER] ?: 0
    }
```
Điểm khác biệt với SharedPreferences chính là ở đây, data được trả về dưới dạng Flow. Giờ đây, các layer phía trên như Repository có thể observe data một cách thống nhất, không cần quan tâm nó đến từ DataStore, Room database hay Server, bởi vì tất cả đều được return dưới dạng Flow.
## Write
Để ghi dữ liệu, chúng ta dùng function `edit`, cũng khá giống với SharedPreferences.
```kotlin
context.dataStore.edit { settings ->
    val openAppCounterValue = settings[OPEN_APP_COUNTER] ?: 0
    settings[OPEN_APP_COUNTER] = openAppCounterValue + 1
}
```

# Proto DataStore
Trước khi tìm hiểu về Proto DataStore, chúng ta cần dạo qua một vòng về protocol buffers.
## Protocol buffers
Đây là một một kiểu định dạng dữ liệu mà không phụ thuộc vào ngôn ngữ lập trình hay platform. Nó giống như JSON nhưng nhỏ và nhanh hơn nhiều lần. Protocol buffers cũng được giới thiệu là định dạng dữ liệu được sử dụng phổ biến nhất tại Google.
* Nó dùng để lưu các dữ liệu nhỏ gọn
* Phân tích cú pháp nhanh
* Hỗ trợ nhiều ngôn ngữ lập trình như C++, C#, Dart, Go, Java, Kotlin, Python
* Tối ưu hoá chức năng thông qua các class được generate tự động

Ví dụ một `message` về thông tin user gồm tên, id và email:
```protobuf
message UserProfile {
  optional string name = 1;
  optional int32 id = 2;
  optional string email = 3;
}
```

Để so sánh về hiệu năng so của Protocol buffers so với JSON, chúng ta thử gọi 500 `GET` requests từ một app Spring Boot này tới app Spring Boot khác với 2 môi trường có nén và không nén data. Và đây là kết quả:
![](https://images.viblo.asia/b71d22aa-c9c7-42e1-be9f-d23879a9c0e4.png)

Chúng ta có thể thấy Protocol buffer **nhanh hơn từ 5 đến 6 lần** so với JSON.
## Create
Để sử dụng Proto DataStore, chúng ta phải định nghĩa loại data bằng một file proto `settings.pb` trong folder `app/src/main/proto/` như sau:
```protobuf
syntax = "proto3";
option java_package = "com.example.application";
option java_multiple_files = true;
message Settings {
  	int32 open_app_counter = 1;
}
```
Sau đó, tiếp tục khai báo một object implement class `Serializer<T>` với `T` là kiểu dữ liệu đã được định nghĩa trong proto file.
```kotlin
object SettingsSerializer : Serializer<Settings> {
  override val defaultValue: Settings = Settings.getDefaultInstance()

  override suspend fun readFrom(input: InputStream): Settings {
    try {
      return Settings.parseFrom(input)
    } catch (exception: InvalidProtocolBufferException) {
      throw CorruptionException("Cannot read proto.", exception)
    }
  }

  override suspend fun writeTo(
    t: Settings,
    output: OutputStream
  ) = t.writeTo(output)
}
```
Và cuối cùng là sử dụng [property delegate](https://viblo.asia/p/design-pattern-delegation-trong-kotlin-cach-de-nho-nguoi-khac-lam-bai-tap-ve-nha-vyDZO3exZwj#_delegated-properties-4) với keyword `dataStore` để tạo một instance của `DataStore<T>`.
```kotlin
val Context.settingsDataStore: DataStore<Settings> by dataStore(
  fileName = "settings.pb",
  serializer = SettingsSerializer
)
```
## Read
Tương tự như Preferences DataStore, chúng ta cũng dùng `DataStore.data` để trả về một Flow.
```kotlin
val openAppCounterFlow: Flow<Int> = context.settingDataStore.data
  .map { settings ->
    // The openAppCounter is generated from the proto schema.
    settings.openAppCounter
  }
```
## Write
Để ghi data vào Proto DataStore, chúng ta có function `updateData()`.
```kotlin
context.settingsDataStore.updateData { currentSettings ->
  currentSettings.toBuilder()
    .setExampleCounter(currentSettings.exampleCounter + 1)
    .build()
}
```
# So sánh với SharedPreferences
![](https://images.viblo.asia/4d39fcd9-db5f-424d-bf88-be99f58cc8eb.png)
## Migrate from SharedPreferences to Preferences DataStore
Để migrate, chúng ta truyền `SharedPreferencesMigration` vào param `produceMigrations`. DataStore sẽ tự động migrate cho chúng ta.
```kotlin
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(
  name = DATA_STORE_NAME
  produceMigrations = { context ->
    listOf(SharedPreferencesMigration(
      context,
      SHARED_PREFERENCES_NAME
    ))
  }
)
```
## Migrate from SharedPreferences to Proto DataStore
Trước tiên, chúng ta cần khai báo `UserProfile` và `UserProfileSerializer` tương tự như các bước ở trên. Sau đó viết một mapping function để migrate từ cặp key-value trong SharedPreferences sang loại dữ liệu trong Proto DataStore.
```kotlin
val Context.dataStore: DataStore<UserProfile> by dataStore(
  fileName = "settings.pb",
  serializer = UserProfileSerializer,
  produceMigrations = { context ->
    listOf(
      SharedPreferencesMigration(
        context,
        "settings_pref"
      ) { prefs: SharedPreferencesView, user: UserProfile ->
        user.toBuilder()
            .setName(prefs.getString(NAME_KEY))
            .setId(prefs.getInt(ID_KEY))
            .setEmail(prefs.getString(EMAIL_KEY))
            .build()
      }
    )
  }
)
```
# References
* https://developer.android.com/topic/libraries/architecture/datastore
* https://protobuf.dev/programming-guides/proto3
* https://android-developers.googleblog.com/2020/09/prefer-storing-data-with-jetpack.html
* https://stackoverflow.com/questions/9986734/which-android-data-storage-technique-to-use
* https://auth0.com/blog/beating-json-performance-with-protobuf
* https://proandroiddev.com/is-jetpack-datastore-a-replacement-for-sharedpreferences-efe92d02fcb3
* https://kinya.hashnode.dev/migrating-sharedpreferences-to-datastore-ckxzlvda101by8rs1c8bg4wdx
* https://amitshekhar.me/blog/jetpack-datastore-preferences
