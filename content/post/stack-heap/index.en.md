---
title: "Bộ nhớ Stack và Heap trong Java"
description: "Quản lý bộ nhớ là một yếu tố quan trọng trong lập trình, biết cách tối ưu bộ nhớ sẽ giúp ứng dụng của chúng ta hoạt động mượt mà, không bị lag hoặc crash. Trong bài viết này, chúng ta sẽ tìm hiểu về vai trò, chức năng và cách hoạt động của từng loại bộ nhớ."
date: 2024-01-04T00:00:00+07:00
slug: stack-heap
image: stack-heap.jpeg
toc: true
categories: Technical
tags: [Stack, Heap, Memory, Java]
---

# Bộ nhớ Stack với Heap là cái chi chi?
Quản lý bộ nhớ là một yếu tố quan trọng trong lập trình, biết cách tối ưu bộ nhớ sẽ giúp ứng dụng của chúng ta hoạt động mượt mà, không bị lag hoặc crash. JVM (Java Virtual Machine) chia bộ nhớ ra thành 2 phần: **Stack** và **Heap** (các bạn đừng nhầm với cấu trúc dữ liệu Stack và Heap nhé). Trong bài viết này, chúng ta sẽ tìm hiểu về vai trò, chức năng và cách hoạt động của từng loại bộ nhớ.

# Bộ nhớ Stack
## Cách hoạt động
**Stack** lưu trữ các biến có kiểu dữ liệu **primitive** (int, float, char, boolean...), **biến cục bộ** và **thông tin về các method được gọi**. Nó hoạt động theo cơ chế **LIFO** (**L**ast **I**n **F**irst **O**ut). Nghĩa là những method nào được gọi sau sẽ được Stack cấp phát cho một frame, chứa thông tin về tham số, biến cục bộ, và Stack sẽ giải phóng frame đó khi method được return.

Ví dụ với một đoạn code như sau:
```java
public static void main(String[] args) {
    doSomething();
}

private static void doSomething() {
    long id = 123456789L;
    doSomethingElse();
}

private static void doSomethingElse() {
    int age = 23;
}
```

Trong Stack sẽ lưu các thông tin theo cấu trúc sau:

![](https://images.viblo.asia/1b40f39b-74e9-4596-aaa7-e042a78b5ec7.png)

Stack được dùng để thực thi một thread, chính vì vậy JVM sẽ tạo một **stack riêng biệt cho mỗi thread**. Mặc định, nếu chúng ta không khai báo kích thước của Stack, JVM sẽ tạo với kích thước tuỳ thuộc vào **hệ điều hành và kiến trúc máy tính** (thông thường là **1MB**). Tuy nhiên, chúng ta có thể dùng flag `-Xss` để tuỳ chỉnh kích thước của Stack (không vượt quá max size, thường là 1GB).
```
java -Xss1048576 // 1.048.576 byte
java -Xss1024k // 1.024 KB
java -Xss1m // 1 MB
```

## StackOverflow
Một lỗi kinh điển liên quan đến Stack là **StackOverflow**. Nó xảy ra khi lượng data được lưu vào Stack vượt quá giới hạn của nó.

Ví dụ khi chúng ta gọi đệ quy nhưng không có điều kiện dừng:

```java
void callRecursion() {
    callRecursion();
}
```
Bạn có thể tưởng tượng các method liên tục được nạp vào Stack, đến một lúc nào đó nó sẽ vượt quá kích thước 1MB nhỏ bé kia. Vì vậy, đoạn code trên sẽ ném ra lỗi `java.lang.StackOverflowError`.

Một số cách để tránh StackOverflow error:
* Đảm bảo các hàm đệ quy có điều kiện dừng hoặc không bị gọi quá nhiều lần
* Quản lý các thread cẩn trọng
* Tránh sử dụng biến local có kích thước quá lớn
* Tránh dependencies vòng tròn

# Bộ nhớ Heap
## Cách hoạt động
**Heap** lưu trữ các biến có kiểu dữ liệu **Object** hoặc **array**. Nó sử dụng cơ chế **cấp phát và giải phóng bộ nhớ động**. Heap khá linh hoạt, có thể mở rộng hoặc thu hẹp theo nhu cầu trong quá trình thực thi chương trình.

Ví dụ, khi chúng ta sử dụng từ khoá `new` để tạo một object `Student`:
```java
private static void doSomething() {
    long id = 123456789L;
    Student student = new Student();
    doSomethingElse();
}

private static void doSomethingElse() {
    int age = 23;
    String str = "Hello World";
}
```
Bộ nhớ sẽ tìm một vùng nhớ trống ngẫu nhiên trong Heap để cấp phát và lưu thông tin của object `student`. Ở bên Stack sẽ có một **biến tham chiếu**, trỏ sang thông tin của object `student` bên Heap. Còn đối với String, Heap có một cấu trúc dữ liệu đặc biệt là String pool để lưu trữ chúng.

![](https://images.viblo.asia/e4bc1a75-0fa2-4366-a1e2-c74f66a3326a.png)

Heap được tạo ra khi JVM khởi chạy và nó được sử dụng miễn là ứng dụng còn chạy. Khác với Stack, Heap được chia sẻ giữa toàn bộ các thread.

Mặc định, kích thước khi khởi tạo của Heap là **256MB** và kích thước lớn nhất là **4068MB**. Chúng ta cũng có thể thay đổi thông số này bằng flag `-Xms` (kích thước khởi tạo) và `-Xmx` (kích thước lớn nhất).
```
// Initial heap size = 512MB, Maximum heap size = 1024MB
java -Xms512m -Xmx1024m
```

## Garbage Collection
Trong Java, **Garbage Collection** có trách nhiệm thu hồi bộ nhớ từ các biến trong bộ nhớ Heap mà **không còn được tham chiếu** đến nữa. Quá trình này được thực hiện một cách tự động. Trong ví dụ trên, khi chúng ta không còn dùng đến object `student` nữa, Garbage Collection sẽ tự động thu hồi vùng nhớ đã cấp phát trước đó cho nó.

Chính nhờ cơ chế này, Heap cho phép cấp phát và giải phóng vùng nhớ với các biến có kích thước lớn và cấu trúc phức tạp ở runtime. Nếu chúng ta tạo ra quá nhiều biến trong Heap nhưng code lởm nên làm cho Garbage Collection không thể thu hồi vùng nhớ hiệu quả sẽ gây ra **memory leak**.

> **Garbage Collection** hoạt động như một bạn quản lý trong nhà hàng. Khi có khách đến, các bạn nhân viên sẽ mời khách ngồi vào một bàn trống, mang bát đũa, menu ra cho khách (cấp phát bộ nhớ). Bạn quản lý sẽ thường xuyên đi vòng vòng kiểm tra, nếu thấy bàn nào khách đã ăn xong đi về thì sẽ gọi nhân viên ra lau dọn sạch sẽ để sẵn sàng đón khách mới (giải phóng bộ nhớ).

## OutOfMemory
Khi Heap bị đầy và chúng ta không thể cấp phát bộ nhớ cho object mới, nó sẽ ném ra lỗi `java.lang.OutOfMemoryError`.

Giải pháp của chúng ta là phân tích code, dùng các công cụ profiling để phát hiện xem memory leak xảy ra ở đâu, xoá tham chiếu tới object khi không cần dùng đến nó nữa và để cho Garbage Collection làm việc của nó. Một số cách tối ưu bộ nhớ:
* Tránh tạo các object không cần thiết
* Tái sử dụng object nếu có thể
* Chọn cấu trúc dữ liệu phù hợp
* Ưu tiên sử dụng biến cục bộ thay vì biến toàn cục

# So sánh

| | Bộ nhớ Stack | Bộ nhớ Heap |
| -------- | -------- | -------- |
| Lưu trữ | primitive, biến cục bộ, method | Object, array |
| Tốc độ truy cập | Nhanh | Chậm |
| Kích thước | Nhỏ, Cố định | Lớn, Linh động |
| Phạm vi sử dụng | Thread tương ứng với Stack | Toàn bộ các thread |
| Thứ tự cấp phát | LIFO | Ngẫu nhiên |
| Thời gian tồn tại của biến | Từ lúc call method đến lúc return | Từ lúc tạo đến lúc bị Garbage Collection giải phóng |

# Kết luận
Trong bài viết này, còn nhiều khái niệm liên quan đến bộ nhớ nhưng mình chưa thể truyền tải hết. Hy vọng bạn đã hiểu một cách căn bản về cách bộ nhớ Stack và Heap hoạt động, từ đó có thêm kinh nghiệm tối ưu hiệu năng của ứng dụng.