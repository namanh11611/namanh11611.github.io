---
title: "When to use Inject, Provides, Binds in Hilt"
description: "In this article, I'll go straight to explaining how to use the @Inject, @Provides, and @Binds annotations."
date: 2022-10-02T01:33:00+07:00
slug: hilt
image: hilt.webp
categories: [Technical, Android]
tags: [Android, Kotlin, Dependency Injection, Hilt]
---

This article will go straight to explaining how to use the `@Inject`, `@Provides`, and `@Binds` annotations. So I'll skip the explanation of Dependency Injection or an introduction to `Hilt`. Let's assume you already know how to use it. Let's go!

# Overview

There are 3 commonly used annotations to inject objects in Hilt:

- `@Inject`: annotation used on the class constructor
- `@Provides`: annotation used in a Module
- `@Binds`: another annotation also used in a Module

So, when should you use each of these?

# Inject

We use the `@Inject` annotation on any constructor where we want to inject an object, from `ViewModel`, `Repository` to `DataSource`. For example:

```kotlin
class ProfileRepository @Inject constructor(
    private val profileDataSource: ProfileDataSource
) {
    fun doSomething() {}
}
```

This makes it easy to inject `ProfileRepository` into other classes, such as a `ViewModel` or `UseCase`. However, you can only use this annotation on constructors of classes you define yourself.

# Provides

To overcome the above limitation—injecting objects of classes you don't define (such as `Retrofit`, `OkHttpClient`, or a `Room` database)—we use `@Provides`. First, you need to create a `@Module` to hold dependencies with the `@Provides` annotation. For example:

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

Since `Retrofit` objects are not defined by your code and are created using the Builder pattern, you can't use the `@Inject` annotation and must use `@Provides`. Now, you can inject the `ApiService` interface object anywhere.

# Binds

For interfaces, you can't use the `@Inject` annotation because they don't have constructors. However, if you have an interface with only one implementation (a class that implements that interface), you can use `@Binds` to inject that interface. Injecting interfaces instead of classes is a good practice and makes testing easier.

Back to the `ProfileRepository` in the `@Inject` section, let's turn it into an interface and create a class that implements it. For example:

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

The advantage of using `@Binds` instead of `@Provides` is that it reduces the amount of generated code, such as Module Factory classes. Here, you can see I still use `@Inject` because the constructor of `ProfileRepositoryImpl` still needs some parameters.

# Summary

So, to summarize:

- Use `@Inject` for your own code
- Use `@Provides` for third-party code
- Use `@Binds` to inject interfaces, reducing unnecessary code

**Reference**

- https://developer.android.com/training/dependency-injection/hilt-android
- https://dagger.dev/hilt
- https://www.valueof.io/blog/inject-provides-binds-dependencies-dagger-hilt
