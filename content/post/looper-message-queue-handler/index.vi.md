---
title: "Tất tần tật về Looper, MessageQueue và Handler trong Android"
description: "Trong Android, nếu thực hiện các tác vụ nặng như request network hoặc đọc ghi database trên main thread có thể sẽ gây ra đơ, crash ứng dụng. Chính vì vậy, để ứng dụng hoạt động mượt mà hơn, chúng ta cần chuyển các tác vụ đó xuống background thread, tránh block main thread."
date: 2024-10-10T00:00:00+07:00
slug: looper-message-queue-handler
image: looper_message_queue_handler.jpg
toc: true
categories: Technical
tags: [Looper, MessageQueue, Handler, Thread, Android, Java]
---

*Photo by [Etienne Girardet](https://unsplash.com/@etiennegirardet?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/a-pile-of-black-and-white-wires-and-a-cassette-OA0qcP6GOw0?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)*

Trong Android, nếu thực hiện các tác vụ nặng như **request network** hoặc **đọc ghi database** trên **main thread** có thể sẽ gây ra đơ, crash ứng dụng. Chính vì vậy, để ứng dụng hoạt động mượt mà hơn, chúng ta cần chuyển các tác vụ đó xuống **background thread**, tránh block **main thread**. Ví dụ như khi user click Submit button trên main thread, tác vụ request netword sẽ được thực thi trên background thread, khi có kết quả trả về thì gửi kết quả trở lại main thread. Android đã cung cấp một số công cụ như **Looper**, **MessageQueue** và **Handler** để xử lý việc chạy đồng thời các tác vụ và truyền message giữa các thread.

Ủa... khoan đã! Việc này đã có **Kotlin Coroutines** rồi mà nhỉ? Đúng vậy, bây giờ hầu hết người ta dùng Kotlin Coroutines để xử lý các use case tương tự như trên. Nhưng trong một số dự án đặc thù, làm việc với các tầng Service ở bên dưới, người ta vẫn chủ yếu phải dùng bộ 3 này để giải quyết vấn đề.

Trong bài viết này, mình sẽ đi giải thích vai trò, nhiệm vụ của từng thành phần và cách mà chúng tương tác với nhau. Vì các thành phần này liên quan chéo đến nhau, nên khi mình giải thích một khái niệm, sẽ phải nhắc đến thành phần khác. Vậy nên chỗ nào chưa hiểu, bạn cứ tạm bỏ qua, rồi khi đọc hết bài có thể quay lại đọc sau để hiểu kỹ hơn nhé.

# Looper

**Looper** là một class quản lý **message loop** cho một **thread**, trong đó mỗi thread **chỉ có duy nhất một** Looper. Đào sâu vào code của Android SDK, chúng ta có thể thấy class Looper được khai báo như sau:

```java
public final class Looper {
    // Looper chứa MessageQueue
    final MessageQueue mQueue;
    // Mối quan hệ giữa Looper với Thread là 1-1
    final Thread mThread;
    
    public static void prepare() {
        prepare(true);
    }
    
    private static void prepare(boolean quitAllowed) {
        // Mỗi thread chỉ có duy nhất một Looper
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
    
    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
}
```

Mặc định, thread **không liên kết** với bất kỳ message loop nào. Để tạo một message loop, chúng ta cần gọi đến method `Looper.prepare()` trong đoạn code trên. Sau đó là gọi method `Looper.loop()` để xử lý các message cho đến khi loop dừng lại.

Ví dụ triển khai Looper:

```java
class LooperExampleThread extends Thread {
    public void run() {
        Looper.prepare();
        Looper.loop();
    }
}
```

# MessageQueue

**MessageQueue** là một cấu trúc dữ liệu chứa danh sách các **Message** và **Runnable** objects cần được xử lý. Nó hoạt động theo nguyên tắc **FIFO** (First-In-First-Out). Bạn có thể truy cập MessageQueue của thread hiện tại bằng method `Looper.myQueue()`.

Quay trở lại đoạn code bên trên của Looper, bạn có thể thấy mỗi Looper chỉ có một MessageQueue duy nhất:

```java
public final class Looper {
    // Looper chứa MessageQueue
    final MessageQueue mQueue;
}
```

Đặc điểm của MessageQueue là message không được thêm trực tiếp vào MessageQueue mà **thông qua Handler**. Sau đó, Looper liên tục trích xuất và xử lý messages từ queue.

# Handler

**Handler** là một class được sử dụng để gửi và xử lý **Message** và **Runnable** objects liên kết với **MessageQueue** của một thread. Mỗi Handler liên kết với **duy nhất một thread và message queue của nó**.

Khi bạn khởi tạo một Handler, bạn phải truyền vào trong **constructor** của nó một Looper. Handler có một vài constructor, nhưng mình lấy ví dụ một cái điển hình như sau:

```java
public Handler(@NonNull Looper looper) {}
```

**MessageQueue** mà chúng ta nhắc đến ở trên thuộc về chính **Looper** này, các message cũng sẽ được thực thi trên thread của **Looper** đó luôn.

Một số method thường dùng của Handler bao gồm:

* `post(Runnable)`
* `postAtTime(java.lang.Runnable, long)`
* `postDelayed(Runnable, Object, long)`
* `sendEmptyMessage(int)`
* `sendMessage(Message)`
* `sendMessageAtTime(Message, long)`
* `sendMessageDelayed(Message, long)`

Handler có 2 chức năng chính:

* **Lên lịch thực hiện** message và runnable trong tương lai. Bạn có thể dùng các method `...AtTime` hoặc `...Delayed` như trên.
* Thực hiện tác vụ trên một **thread khác** với thread hiện tại. Như đã nói ở trên, bạn muốn thực hiện trên thread nào thì có thể truyền Looper của nó vào constructor của Handler khi khởi tạo.

# Giao tiếp giữa các thành phần

![Looper, MessageQueue, Handler](communication.webp)

1. Khi một **Message** hoặc **Runnable** được gửi thông qua **Handler**, nó được đưa vào **MessageQueue**.
2. **Looper** liên tục kiểm tra **MessageQueue** để tìm message mới.
3. Khi có message, **Looper** trích xuất nó từ queue và gửi đến **Handler** tương ứng để xử lý.
4. **Handler** xử lý message trong thread mà nó được liên kết.

Bạn có thể xem đoạn code minh họa cho hình ảnh bên trên:

```java
class ProcessingThread extends Thread {
    public Handler mHandler;
    
    public void run() {
        Looper.prepare();
        
        // Looper.myLooper() return Looper object
        // liên kết với thread hiện tại
        mHandler = new Handler(Looper.myLooper()) {
            public void handleMessage(Message msg) {
                // Xử lý các message nhận được
            }
        };
        
        Looper.loop();
    }
}

// ClientThread có reference đến mHandler object
// của ProcessingThread
class ClientThread extends Thread {
    private void sendMessageExample() {
        Message msg = Message.obtain(mHandler, 1);
        msg.obj = "New message";
        mHandler.sendMessage(msg);
    }
    
    private void sendRunnableExample() {
        mHandler.post(new Runnable() {
            @Override
            public void run() {
                // Tác vụ thực thi trên ProcessingThread
            }
        });
    }
}
```

## HandlerThread

Trong thực tế thì ít ai tạo Thread và quản lý Looper thủ công như ví dụ trên. Android đã cung cấp cho chúng ta **HandlerThread**, một loại thread đặc biệt, có chứa một property là Looper. Vì vậy, chúng ta có thể get Looper bằng method `getLooper()` trong HandlerThread.

```java
HandlerThread thread = new HandlerThread("ProcessingThread");
thread.start();
Looper looper = thread.getLooper();
Handler handler = new Handler(looper);
```

## Main Thread

**Main thread** (UI thread) trong Android đã có sẵn một Looper, bạn có thể get nó thông qua method `Looper.getMainLooper()`. Một ví dụ điển hình nhất là chúng ta hay tạo một Handler để delay việc thực hiện một tác vụ trên UI thread như sau:

```java
Handler handler = new Handler(Looper.getMainLooper());
handler.postDelayed(new Runnable() {
    @Override
    public void run() {
        // Tác vụ bị delay
    }
}, 3000);

```

# Kết luận

**Looper**, **MessageQueue** và **Handler** là ba thành phần quan trọng trong hệ thống xử lý bất đồng bộ của Android. Chúng làm việc cùng nhau để đảm bảo việc giao tiếp giữa các thread được thực hiện một cách hiệu quả và an toàn. Hiểu rõ về cách hoạt động của chúng sẽ giúp bạn phát triển các ứng dụng Android xịn xò hơn. Chân thành cảm ơn các bạn đã đọc đến đây!

# Reference

* https://developer.android.com/reference/android/os/Looper
* https://developer.android.com/reference/android/os/MessageQueue
* https://developer.android.com/reference/android/os/Handler
