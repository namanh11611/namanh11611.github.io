---
title: "Flutter Hooks - viết code ngắn gọn và hiệu quả hơn"
description: "Khi tìm kiếm từ khoá Hooks trên Google, chúng ta sẽ thấy rất nhiều kết quả liên quan đến React. Quả thật, như trong phần giới thiệu, thư viện flutter_hooks được tác giả Remi Rousselet build dựa trên niềm cảm hứng từ React."
date: 2024-04-30T00:00:00+07:00
slug: flutter-hooks
image: hooks.jpeg
toc: true
categories: Technical
tags: [Flutter, Hooks, Riverpod]
---

# Flutter Hooks là cái chi chi?

Khi tìm kiếm từ khoá **Hooks** trên Google, chúng ta sẽ thấy rất nhiều kết quả liên quan đến **React**. Quả thật, như trong phần giới thiệu, thư viện [**flutter_hooks**](https://pub.dev/packages/flutter_hooks) được tác giả [**Remi Rousselet**](https://github.com/rrousselGit) build dựa trên niềm cảm hứng từ React.

**Hooks** là một loại object giúp quản lý life-cycle của `Widget`. Mục tiêu duy nhất của nó là giúp tăng khả năng chia sẻ source code giữa các `Widget` bằng cách loại bỏ các phần code trùng lặp.

Các bạn có thể sẽ thắc mắc: *"Ủa, `StatefulWidget` trong Flutter đã có method `initState` và `dispose` để lo việc quản lý life-cycle rồi cơ mà? Cần gì Hook nữa?"*. Chính xác, thế nhưng chúng ta rất khó để tái sử dụng code trong 2 method này. Và **Hooks** sinh ra để giải quyết vấn đề đó.

# Hướng dẫn sử dụng

Một ví dụ đơn giản về Hooks như sau:

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

Bạn có thể thấy `Widget` của chúng ta thay vì extend `StatefulWidget` hay `StatelessWidget`, thì bây giờ nó phải extend một `HookWidget`. Trong method `build` có một từ khoá lạ lạ là `useState`, đấy chính là một trong những **hook** của Flutter Hooks, phần sau chúng ta sẽ cùng đi tìm hiểu một số hook cơ bản của nó.

Giờ đây, khi bạn click vào `FloatingActionButton`, giá trị của text sẽ thay đổi với biến `counter` tăng dần, nó tương tự như cách mà `StatefulWidget` hoạt động phải không nào?

## useEffect hook

Nó gần giống với `useEffect` hook của React, được sử dụng để thực hiện các side effect **synchronously** trong quá trình render. Effect có thể return một function, function này sẽ được gọi khi **effect được gọi lại** hoặc **widget bị dispose**.

Mặc định, effect được gọi lại mỗi lần build, trừ khi chúng ta truyền **param key**. Khi đó, effect chỉ được gọi lại khi key thay đổi.

Side effect có thể bao gồm stream subscription, mở một **WebSocket connection** hoặc thực hiện một **HTTP request**. Chúng ta có thể cancel chúng khi widget bị dispose.

```dart
useEffect(() {
  performSideEffect();
  return () {
    cleanUp();
  };
}, [key]);
```

## useState hook

Đây là loại hook cơ bản nhất khi bạn muốn chuyển một `StatelessWidget` sang `StatefulWidget`. Khi được gọi trong method `build`, nó sẽ giữ state giữa những lần widget rebuild.

```dart
final counter = useState(0);
```

Trong ví dụ này, chúng ta truyền cho nó một giá trị khởi tạo là `0`. Biến `counter` chính là một instance của `ValueNotifier`. State được lưu trong thuộc tính `value` của `ValueNotifier`. Mỗi khi giá trị của `value` bị thay đổi, `useState` hook sẽ rebuild widget để hiển thị giá trị mới.

## useMemoized hook

Đây là một method hữu ích khi bạn cần tạo một object phức tạp và muốn trả về cùng một object đó qua nhiều lần rebuild widget con. `useMemoized` sẽ giúp chúng ta cache object đó, giá trị chỉ được tính toán trong lần gọi đầu tiên, còn những lần sau, nó sẽ trả về giá trị đã được lưu trước đó.

```dart
final complexObject = useMemoized(
  () => createComplexObject(),
  []
);
```

## useRef hook

Tạo một object chứa một thuộc tính có thể thay đổi. Tuy nhiên thì việc thay đổi thuộc tính của object sẽ không có effect. Nó phù hợp cho các use-case mà chúng ta cần share state giữa những lần `build` method được gọi, nhưng tránh được việc rebuild không cần thiết.

```dart
final textController = useTextEditingController();

/// Sử dụng useState() trong trường hợp này
/// sẽ làm widget rebuild mỗi lần nhập một ký tự
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

Cache instance của toàn bộ một function, nếu như chính function đó được gọi lại.

```dart
final cachedFun = useCallback(() {
  Statements
}, []);
```

## useContext hook

Giữ `BuildContext` của `HookWidget`, giúp chúng ta không phải truyền param `context` qua các method.

```dart
Widget createSizedBox() {
  return SizedBox(
    height: MediaQuery.sizeOf(useContext()).height / 10
  );
}
```

## useValueChanged hook

Theo dõi một giá trị và trigger một callback bất cứ khi nào giá trị của nó thay đổi. Quay trở lại với ví dụ ban đầu, chúng ta sửa một chút như sau:

```dart
final count = useState(0);
final newCount = useState(0);
useValueChanged(count.value, (oldValue, oldResult) {
  print('oldValue = $oldValue, oldResult = $oldResult');
  newCount.value += 5;
  return newCount.value;
});
```

Khi click `FloatingActionButton`, kết quả sẽ được in ra lần lượt như sau:

```text
oldValue = 0, oldResult = null
oldValue = 1, oldResult = 5
oldValue = 2, oldResult = 10
...
```

Bạn có thể thấy, mỗi khi giá trị của `count` thay đổi, `newCount` cũng sẽ được thay đổi theo.

## useStream hook

Giúp subscribe một `Stream` và return state hiện tại nó.

```dart
final snapshot = useStream(backend.stream);
```

## useAnimationController hook

Tạo một `AnimationController` và nó sẽ bị dispose một cách tự động.

```dart
final controller = useAnimationController(
  duration: Duration(seconds: 1)
);
```

# Lợi ích của Hooks

1. **State Management**: Hooks giúp đơn giản hóa việc quản lý local state, đảm bảo widget chỉ xử lý logic liên quan đến nó.
2. **Giảm duplicate code**: Custom hooks cho phép bạn tái sử dụng logic của widget, giảm đáng kể việc duplicate code và cải thiện chất lượng code.
3. **Cải thiện Hot Reload**: Với hooks, hot reload mặc định ít bị lỗi hơn vì trạng thái của hook được giữ nguyên.
4. **Tiện lợi**: Một số built-in hooks như `useAnimationController`, `useFuture` và `useStream` là những method có thể áp dụng cho nhiều use-case phổ biến.
5. **Đơn giản hóa life-cycle**: Việc quản lý các life-cycle phức tạp trở nên đơn giản hơn rất nhiều với Flutter Hooks. Chúng cung cấp một giải pháp để dễ dàng quản lý state, side-effects và stream subscriptions.
6. **Custom hooks linh hoạt**: Ngoài các hook đã có sẵn, Flutter Hooks cho phép bạn tạo các hooks tùy chỉnh, giúp việc viết code linh hoạt hơn.

Có thể bạn đã biết **Remi Rousselet** còn là tác giả của 2 thư viện **State Management** nổi tiếng là [**Provider**](https://pub.dev/packages/provider) và [**Riverpod**](https://pub.dev/packages/flutter_riverpod). Trong đó thì anh ấy đã kết hợp Riverpod và Hook trong một thư viện là [**Hooks Riverpod**](https://pub.dev/packages/hooks_riverpod). Bạn có thể nhân hai sức mạnh với thư viện này. Riverpod là dành cho "global" application state, còn hooks dành cho local widget state.

# Lời kết

Mình thấy **Flutter Hooks** khá tiềm năng, nhưng vẫn thấy chưa nhiều dự án áp dụng. Không biết các bạn đã từng áp dụng nó chưa? Nếu thấy nó có ưu hoặc nhược điểm gì, hãy chia sẻ với mình nhé.
