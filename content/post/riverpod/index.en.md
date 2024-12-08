---
title: "Using Code Generation Providers in Flutter Riverpod to Simplify Life"
description: "Have you ever struggled to choose among the 7 types of providers in Riverpod for different use cases in your project? Well, now, Remi Rousselet introduces a new way to use Riverpod with code generation, making developers' lives a bit easier."
date: 2024-10-11T00:00:00+07:00
slug: riverpod
image: riverpod.jpg
toc: true
categories: Technical
tags: [Flutter, Riverpod, Provider, State Management, Code Generation]
---

*Photo by [Dillon Shook](https://unsplash.com/@dillonjshook?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/person-sitting-on-sofa-resting-its-feet-on-top-of-coffee-table-while-using-laptop-3iPKIXVXv_U?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)*

Have you ever struggled to choose one of the seven types of **providers** in **Riverpod** for specific use cases in your project? For instance, the [documentation](https://riverpod.dev/docs/concepts/providers#different-types-of-providers) explains that both **NotifierProvider** and **StateNotifierProvider** are used when:

> A complex state object that is immutable except through an interface.

On the other hand, **ChangeNotifierProvider** are not recommended for scalable applications.

What is going on??? The author really knows how to make developers feel confused…

But now, **Remi Rousselet** has introduced a new way to use **Riverpod** with **code generation**, making developers' lives a bit easier.

# Syntax

In simple terms, **code generation in Riverpod** allows us to declare providers using the `@riverpod` annotation, and most of the code is automatically generated using Dart's built-in `build_runner` tool.

Instead of defining providers as before:

```dart
final fetchUserProvider = FutureProvider.autoDispose.family<User, int>((ref, userId) async {
    final json = await http.get('api/user/$userId');
    return User.fromJson(json);
});
```

Now, you only need to write:

```dart
@riverpod
Future<User> fetchUser(FetchUserRef ref, {required int userId}) async {
    final json = await http.get('api/user/$userId');
    return User.fromJson(json);
}
```

Instead of deliberating over which of the seven providers to use, you can now use the following table to quickly choose the appropriate provider for your use case:

|  | Functional <br> (Can’t perform side-effects <br> using public methods) | Class-Based <br> (Can perform side-effects <br> using public methods) |
| -------- | -------- | -------- |
| Sync | ![Functional Sync](riverpod_function_sync.png) | ![Class-Based Sync](riverpod_class_sync.png) |
| Async - Future | ![Functional Async - Future](riverpod_function_future.png) | ![Class-Based Async - Future](riverpod_class_future.png) |
| Async - Stream | ![Functional Async - Stream](riverpod_function_stream.png) | ![Class-Based Async - Stream](riverpod_class_stream.png) |

## keepAlive

In this new approach, all providers are **auto-dispose** by default, meaning their state is destroyed when no listeners observe them. This is the opposite of the old approach (where you had to explicitly add `autoDispose`, and the default was no dispose).

To disable auto-dispose and keep your provider alive, use:

```dart
@Riverpod(keepAlive: true)
```

## Parameters

As seen in the earlier example, adding parameters to a provider is now as straightforward as adding parameters to a regular function. There’s no need to use `family` like in the old approach.

For functional providers, parameters are added directly to the function:

```dart
@riverpod
String example(
    ExampleRef ref,
    int param1, {
    String param2 = 'foo',
}) {
    return 'Hello $param1 & param2';
}
```

For class-based providers, parameters are added to the `build` method:

```dart
@riverpod
class Example extends _$Example {
    @override
    String build(
        int param1, {
        String param2 = 'foo',
    }) {
        return 'Hello $param1 & param2';
    }

    // Add methods to mutate the state
}
```

# Advantages

Currently, using code generation or the old approach is optional. If you’re considering why you should switch, here are some reasons provided by the author:

* Better syntax, more readable/flexible, and with a reduced learning curve.
  * No need to worry about the type of provider. Write your logic, and Riverpod will pick the most suitable provider for you.
  * The syntax no longer looks like we're defining a "dirty global variable". Instead we are defining a custom function/class.
  * Passing parameters to providers is now unrestricted. Instead of being limited to using .family and passing a single positional parameter, you can now pass any parameter. This includes named parameters, optional ones, and even default values.
* Stateful hot-reload of the code written in Riverpod.
* Better debugging, through the generation of extra metadata that the debugger then picks up.
* Some Riverpod features will be available only with code generation.

# Disadvantages

However, when applied to real-world projects, there are some drawbacks to consider.

Currently, as code generation is relatively new, few projects have adopted it, making it hard to find reference source code. For the most part, you’ll rely on Riverpod’s documentation during development.

In this AI-driven era, developers often use tools to generate code. With limited adoption, most tools generate Riverpod code in the old style. But don’t worry! Android Studio has the [Flutter Riverpod Snippets](https://plugins.jetbrains.com/plugin/14641-flutter-riverpod-snippets) plugin, which helps you write code faster. Just type riverpod, and it will suggest the four main provider types.

These drawbacks are temporary. As code generation becomes more popular, these issues will be resolved. So there’s no need to worry too much.

# Reference

* https://riverpod.dev/docs/concepts/about_code_generation
