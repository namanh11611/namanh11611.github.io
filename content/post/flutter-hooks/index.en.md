---
title: "Flutter Hooks - Write More Concise and Efficient Code"
description: "When searching for the keyword Hooks on Google, you'll find many results related to React. Indeed, as mentioned in the introduction, the flutter_hooks library by Remi Rousselet was built inspired by React."
date: 2024-04-30T00:00:00+07:00
slug: flutter-hooks
image: hooks.webp
toc: true
categories: [Technical, Flutter]
tags: [Flutter, Hooks, Riverpod]
---

# What is Flutter Hooks?

When searching for the keyword **Hooks** on Google, you'll find many results related to **React**. Indeed, as mentioned in the introduction, the [**flutter_hooks**](https://pub.dev/packages/flutter_hooks) library by [**Remi Rousselet**](https://github.com/rrousselGit) was built inspired by React.

**Hooks** are objects that help manage the life-cycle of a `Widget`. Their sole purpose is to increase the ability to share source code between `Widgets` by eliminating duplicated code.

You might wonder: *"Wait, doesn't `StatefulWidget` in Flutter already have `initState` and `dispose` methods to handle life-cycle management? Why do we need Hooks?"* That's correct, but it's very difficult to reuse code in these two methods. **Hooks** were created to solve that problem.

# Usage Guide

A simple example of Hooks is as follows:

```dart
class MyHookWidget extends HookWidget {
  const MyHookWidget({super.key});

  @override
  Widget build(BuildContext context) {
    final counter = useState(0);

    return Scaffold(
      body: Center(
        child: Text('Counter: ${counter.value}'),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => counter.value++,
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

You can see that our `Widget` now extends `HookWidget` instead of `StatefulWidget` or `StatelessWidget`. In the `build` method, there's a new keyword `useState`, which is one of the **hooks** in Flutter Hooks. We'll explore some basic hooks below.

Now, when you click the `FloatingActionButton`, the text value will change as the `counter` variable increases, similar to how `StatefulWidget` works, right?

## useEffect hook

This is similar to the `useEffect` hook in React, used to perform side effects **synchronously** during rendering. The effect can return a function, which will be called when the **effect is recalled** or the **widget is disposed**.

By default, the effect is called every build, unless you pass a **key parameter**. In that case, the effect is only called when the key changes.

Side effects can include stream subscriptions, opening a **WebSocket connection**, or making an **HTTP request**. You can cancel them when the widget is disposed.

```dart
useEffect(() {
  performSideEffect();
  return () {
    cleanUp();
  };
}, [key]);
```

## useState hook

This is the most basic hook when you want to convert a `StatelessWidget` to a `StatefulWidget`. When called in the `build` method, it maintains state between widget rebuilds.

```dart
final counter = useState(0);
```

In this example, we pass an initial value of `0`. The `counter` variable is an instance of `ValueNotifier`. The state is stored in the `value` property of `ValueNotifier`. Whenever the `value` changes, the `useState` hook will rebuild the widget to display the new value.

## useMemoized hook

This is a useful method when you need to create a complex object and want to return the same object across multiple child widget rebuilds. `useMemoized` helps cache that object; the value is only computed on the first call, and subsequent calls return the previously stored value.

```dart
final complexObject = useMemoized(
  () => createComplexObject(),
  []
);
```

## useRef hook

Creates an object containing a mutable property. However, changing the property of the object does not trigger an effect. It's suitable for use-cases where you need to share state between `build` method calls, but want to avoid unnecessary rebuilds.

```dart
final textController = useTextEditingController();

/// Using useState() in this case
/// would cause the widget to rebuild on every character input
// final name = useState('');
final name = useRef('');
useEffect(() {
  textController.addListener(() {
    name.value = textController.text;
  });
  return null;
}, []);
```

## useCallback hook

Caches the instance of an entire function, if that function is recalled.

```dart
final cachedFun = useCallback(() {
  Statements
}, []);
```

## useContext hook

Keeps the `BuildContext` of the `HookWidget`, so you don't have to pass the `context` parameter through methods.

```dart
Widget createSizedBox() {
  return SizedBox(
    height: MediaQuery.sizeOf(useContext()).height / 10
  );
}
```

## useValueChanged hook

Monitors a value and triggers a callback whenever its value changes. Returning to the initial example, let's modify it a bit:

```dart
final count = useState(0);
final newCount = useState(0);
useValueChanged(count.value, (oldValue, oldResult) {
  print('oldValue = $oldValue, oldResult = $oldResult');
  newCount.value += 5;
  return newCount.value;
});
```

When you click the `FloatingActionButton`, the output will be printed as follows:

```text
oldValue = 0, oldResult = null
oldValue = 1, oldResult = 5
oldValue = 2, oldResult = 10
...
```

As you can see, whenever the value of `count` changes, `newCount` also changes accordingly.

## useStream hook

Helps subscribe to a `Stream` and returns its current state.

```dart
final snapshot = useStream(backend.stream);
```

## useAnimationController hook

Creates an `AnimationController` and disposes it automatically.

```dart
final controller = useAnimationController(
  duration: Duration(seconds: 1)
);
```

# Benefits of Hooks

1. **State Management**: Hooks simplify local state management, ensuring widgets only handle logic relevant to themselves.
2. **Reduce duplicate code**: Custom hooks allow you to reuse widget logic, significantly reducing duplicate code and improving code quality.
3. **Improve Hot Reload**: With hooks, hot reload is less likely to fail because the hook's state is preserved.
4. **Convenience**: Some built-in hooks like `useAnimationController`, `useFuture`, and `useStream` are methods that can be applied to many common use-cases.
5. **Simplify life-cycle**: Managing complex life-cycles becomes much simpler with Flutter Hooks. They provide a solution for easily managing state, side-effects, and stream subscriptions.
6. **Flexible custom hooks**: Besides the built-in hooks, Flutter Hooks allows you to create custom hooks, making your code more flexible.

You may already know that **Remi Rousselet** is also the author of two famous **State Management** libraries: [**Provider**](https://pub.dev/packages/provider) and [**Riverpod**](https://pub.dev/packages/flutter_riverpod). He has combined Riverpod and Hooks in a library called [**Hooks Riverpod**](https://pub.dev/packages/hooks_riverpod). You can harness the power of both with this library. Riverpod is for "global" application state, while hooks are for local widget state.

# Conclusion

I find **Flutter Hooks** quite promising, but not many projects have adopted it yet. Have you ever used it? If you see any pros or cons, please share with me!
