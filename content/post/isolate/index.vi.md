---
title: "Isolate trong Flutter: Kẻ cứu cánh cho những tác vụ nặng"
description: "Như anh em đã biết, tất cả Dart code chạy trong các isolate, bắt đầu từ main isolate mặc định. Khi phát triển các ứng dụng Flutter, việc xử lý các tác vụ nặng như xử lý ảnh, tính toán phức tạp... trên main isolate này sẽ gây ra hiện tượng giật lag. Và Isolate API sinh ra chính là để giải quyết vấn đề này, nó sẽ giúp chúng ta đưa những tác vụ nặng đó xuống background worker, giúp ứng dụng trở lại với sự mượt mà vốn có."
date: 2025-05-25T00:00:00+07:00
slug: isolate
image: isolate.webp
toc: true
categories: [Technical, Flutter]
tags: [Flutter, Isolate]
---

Như anh em đã biết, tất cả **Dart** code chạy trong các **isolate**, bắt đầu từ **main isolate** mặc định. Khi phát triển các ứng dụng Flutter, việc xử lý các tác vụ nặng như xử lý ảnh, tính toán phức tạp... trên **main isolate** này sẽ gây ra hiện tượng **giật lag**. Mặc dù Flutter đã cung cấp cho chúng ta **Future** hay **Stream**, nhưng về bản chất chúng vẫn được xử lý trên **main isolate**. Và **Isolate API** sinh ra chính là để giải quyết vấn đề này, nó sẽ giúp chúng ta đưa những tác vụ nặng đó xuống **background worker**, giúp ứng dụng trở lại với sự mượt mà vốn có. Trong bài viết này, mình sẽ giải thích cách sử dụng **Isolate** và lần lượt cung cấp các best practice trong quá trình sử dụng.

# Isolate là gì?

**Dart** là ngôn ngữ **single-thread**, khi bạn sử dụng **async**, **await**, thực chất là nó đang chạy **concurrency** (đồng thời). Khi một tác vụ tốn thời gian chạy trên main thread, nó sẽ **block toàn bộ UI**. Và **Isolate** là nhân tố giúp chúng ta giải quyết bài toán **parallelism** (song song).

**Isolate** là một **luồng thực thi độc lập** trong Dart. Mỗi isolate có bộ nhớ **Heap** riêng, đảm bảo rằng không isolate nào có thể truy cập vào bộ nhớ của isolate khác, giúp cho ứng dụng chạy mượt mà mà không có trạng thái chia sẻ.

Bắt tay ngay vào code cho dễ hình dung, chúng ta sẽ đến với ví dụ đầu tiên là tạo ra một Isolate đơn giản:

```dart
import 'dart:isolate';

void main() {
  Isolate.spawn(isolateMethod, "Hello World from Main Isolate");
  print('Main isolate');
}

void isolateMethod(String message) {
  print('New isolate: $message');
}
```

Trong ví dụ trên, hàm `main()` sẽ chạy trên **main isolate**, thường được gọi là **UI isolate**. Chúng ta tạo một **isolate mới** bằng cách gọi phương thức `Isolate.spawn()`, nó sẽ thực thi hàm `isolateMethod()`. Các isolate phụ như này sẽ không truy cập được UI.

# Giao tiếp giữa các Isolate

Khi cần trao đổi dữ liệu giữa các isolate, Dart đã cung cấp cho chúng ta 2 class là `SendPort` và `ReceivePort` để giúp các Isolate giao tiếp với nhau thông qua message. Các bạn hãy đọc qua đoạn code sau, mình sẽ giải thích chi tiết bên dưới.

```dart
void main() async {
  final mainReceivePort = ReceivePort();
  final isolate = await Isolate.spawn(twoWayIsolate, mainReceivePort.sendPort);

  // Nhận SendPort từ isolate phụ
  mainReceivePort.listen((message) {
    if (message is SendPort) {
      print('Main: SendPort');
      message.send('Hello');
    } else {
      print('Main received: $message'); // Output: Hello to Henry
      isolate.kill();
    }
  });
}

void twoWayIsolate(SendPort mainSendPort) {
  final receivePort = ReceivePort();

  // Gửi SendPort của isolate phụ về main
  mainSendPort.send(receivePort.sendPort);

  receivePort.listen((message) {
    if (message is String) {
      print('Worker received: $message'); // Output: Hello
      final newMessage = '$message to Henry';
      mainSendPort.send(newMessage);
    }
  });
}
```

Trong ví dụ này, chúng ta sẽ thiết lập giao tiếp 2 chiều giữa main isolate và isolate phụ thông qua đoạn code:

```dart
// Gửi SendPort của isolate phụ về main
mainSendPort.send(receivePort.sendPort);
```

Ngay sau khi nhận được message là `SendPort`, main isolate đã gửi lời chào đến isolate phụ:

```dart
message.send('Hello');
```

Isolate phụ cũng đã đáp lễ bằng cách gửi lời chào ngược lại đến main isolate:

```dart
final newMessage = '$message to Henry';
mainSendPort.send(newMessage);
```

Các bạn có thể thấy, `SendPort.send()` sẽ là thằng chịu trách nhiệm gửi message, còn việc lắng nghe message sẽ được thực hiện qua `ReceivePort.listen()`.

# Sử dụng `compute` cho tác vụ đơn giản

Flutter cung cấp hàm `compute` để chạy hàm trong isolate mới với chỉ với 1 dòng code đơn giản như sau:

```dart
Future<void> fetchData() async {
  final result = await compute(_heavyProcessing, 1000000);
  print('Kết quả: $result');
}

// Hàm này sẽ chạy trong isolate riêng
int _heavyProcessing(int iterations) {
  int sum = 0;
  for (int i = 0; i < iterations; i++) {
    sum += i;
  }
  return sum;
}
```

Ưu điểm của `compute` là **tự động quản lý** isolate, nó sẽ được **hủy** ngay sau khi hoàn thành.

Tuy nhiên, có ưu thì cũng phải có nhược, giới hạn của `compute` là chỉ truyền được **1 tham số**, nếu muốn nhiều tham số hơn thì bạn phải đóng gói nó vào Map hoặc List. Ngoài ra, `compute` cũng không thể giao tiếp 2 chiều như `Isolate.spawn` trong ví dụ trên.

# Best Practices

## Chỉ dùng isolate cho các tác vụ tốn nhiều CPU

Với các network request, chúng ta có thể dùng http client bình thường. Chỉ nên dùng nó trong các trường hợp tốn nhiều CPU như xử lý ảnh.

```dart
// Nên
compute(imageProcessing, imageData);

// Không nên
compute(networkRequest, url);
```

## Tránh tạo isolate liên tục

Việc tạo isolate mới liên tục gây tiêu tốn CPU và bộ nhớ, đồng thời việc khởi tạo isolate cũng khá chậm, mất khoảng 30ms mỗi lần. Vậy nên, việc sử dụng **Worker Pool** sẽ giúp tái sử dụng các isolate đã tạo.

## Sử dụng kiểu dữ liệu đơn giản khi truyền qua port

Do các isolate không chia sẻ bộ nhớ nên dữ liệu truyền giữa các isolate phải được **serialized**/**deserialized**. Nếu sử dụng kiểu dữ liệu phức tạp sẽ làm tăng thời gian chuyển đổi và tốn bộ nhớ khi copy dữ liệu.

```dart
// Nên
sendPort.send({'id': 1, 'value': 42});

// Không nên
sendPort.send(MyComplexClass());
```

## Dừng isolate khi không cần

Khi sử dụng `Isolate.spawn`, nó sẽ không tự động giải phóng khi không dùng nữa. Việc để isolate chạy ngầm sẽ gây tốn CPU/bộ nhớ và có thể là cả memory leak. Vậy nên chúng ta có thể sử dụng hàm `Isolate.kill` để dừng khi hoàn thành.

```dart
isolate.kill(priority: Isolate.immediate);
```

# Kết Luận

Isolate là công cụ mạnh mẽ để xử lý các tác vụ nặng trong Flutter mà không ảnh hưởng đến UI. Tuy có độ phức tạp nhất định, nhưng khi sử dụng đúng cách bằng việc kết hợp với `compute` và các thư viện hỗ trợ, bạn có thể tạo ra ứng dụng mượt mà ngay cả khi xử lý nghiệp vụ phức tạp.

# Reference
* [Concurrency in Dart](https://dart.dev/language/concurrency)
* [Isolates](https://dart.dev/language/isolates)
