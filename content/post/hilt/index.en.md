---
title: "Khi nào nên dùng Inject, Provides, Binds trong Hilt"
description: "Bài viết này mình sẽ đi thẳng vào giải thích cách dùng các annotation @Inject, @Provides và @Binds."
date: 2022-10-02T01:33:00+07:00
image: hilt.jpeg
categories: Technical
tags: [Android, Kotlin, Dependency Injection]
---

Bài viết này mình sẽ đi thẳng vào giải thích cách dùng các annotation `@Inject`, `@Provides` và `@Binds`. Vậy nên bỏ qua phần giải thích về Dependency Injection hay giới thiệu về `Hilt`. Coi như mọi người đã biết cách dùng nó rồi nhé. Chiến thôi!!!
# Tổng quan
Chúng ta có 3 annotation thường dùng để inject các object trong Hilt:
- `@Inject`: annotation dùng ở constructor của class
- `@Provides`: annotation dùng ở Module
- `@Binds`: một annotation khác cũng dùng ở Module

Câu hỏi đặt ra là vậy khi nào thì dùng những thằng này nhỉ?

# Inject
Chúng ta dùng `@Inject` annotation ở tất cả các constructor mà mình cần inject object, từ `ViewModel`, `Repository` đến `DataSource`. Ví dụ:
```kotlin
class ProfileRepository @Inject constructor(
    private val profileDataSource: ProfileDataSource
) {
    fun doSomething() {}
}
```
Việc này giúp chúng ta dễ dàng inject tiếp `ProfileRepository` vào các class khác, ví dụ như `ViewModel` hoặc `UseCase`. Tuy nhiên thì chúng ta lại chỉ có thể sử dụng annotation này để annotate constructor của những class mà mình tự define.
# Provides
Vậy thì để khắc phục điểm yếu trên, inject object của những class mà mình không define (Ví dụ như `Retrofit`, `OkHttpClient` hoặc `Room` database), chúng ta cùng đến với `@Provides`. Trước tiên, chúng ta cần tạo một `@Module` để chứa các dependency với annotation `@Provides`. Ví dụ:
```kotlin
@Module
class NetworkModule {
    @Provides
    fun providesApiService(): ApiService =
        Retrofit.Builder()
            .addConverterFactory(GsonConverterFactory.create())
            .baseUrl(BASE_URL)
            .build()
            .create(ApiService::class.java)
}
```
Vì `Retrofit` khởi tạo object không phải code của chúng ta define, thêm nữa lại còn khởi tạo theo kiểu Builder pattern, nên chúng ta không thể dùng `@Inject` annotation mà bắt buộc phải dùng `@Provides`. Bây giờ, chúng ta đã có thể inject object của interface `ApiService` ở bất cứ đâu.
# Binds
Đối với interface, chúng ta không thể dùng annotation `@Inject`, vì nó không có constructor function. Tuy nhiên, nếu bạn có một interface mà chỉ có duy nhất một implementation (một class implement interface đó), thì bạn có thể dùng `@Binds` để inject interface đó. Việc inject interface thay vì class là một good practice, giúp dễ dàng test hơn.

Quay trở lại với `ProfileRepository` ở phần `@Inject`, chúng ta sẽ biến nó thành một interface, và tạo một class implement interface đó. Ví dụ:
```kotlin
interface ProfileRepository {
}

class ProfileRepositoryImpl @Inject constructor(
    private val profileDataSource: ProfileDataSource
) : ProfileRepository {
}

@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {
    @Singleton
    @Binds
    abstract fun bindProfileRepository(profileRepository: ProfileRepositoryImpl): ProfileRepository
}

class RegisterUseCase @Inject constructor(
    private val profileRepository: ProfileRepository
)
```
Ưu điểm của việc dùng `@Binds` thay cho `@Provides` là nó giúp giảm lượng code được generate, như là Module Factory class. Ở đây bạn có thể thấy mình vẫn dùng `@Inject`, bởi vì constructor function của `ProfileRepositoryImpl` vẫn cần một số parameter.
# Tổng kết
Vậy tóm gọn lại là
- Dùng `@Inject` cho code của bạn
- Dùng `@Provides` cho code của bên thứ 3
- Dùng `@Binds` cho inject interface, giúp giảm lượng code không cần thiết

**Reference**
- https://developer.android.com/training/dependency-injection/hilt-android
- https://dagger.dev/hilt
- https://www.valueof.io/blog/inject-provides-binds-dependencies-dagger-hilt
