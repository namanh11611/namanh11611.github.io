---
title: "Tất tần tật về Process trong Android"
description: "Process là một khái niệm cơ bản nhưng cũng rất quan trọng trong Android. Khi chúng ta khởi chạy ứng dụng, mặc định tất cả các component như Activity, Service, BroadcastReceiver, ContentProvider sẽ cùng chạy trên một Linux Process."
date: 2024-06-21T00:00:00+07:00
slug: process
image: process.webp
toc: true
categories: [Technical, Android]
tags: [Android, Process]
---

# Khái niệm

**Process** là một khái niệm cơ bản nhưng cũng rất quan trọng trong Android. Khi chúng ta khởi chạy ứng dụng, mặc định tất cả các component như **Activity**, **Service**, **BroadcastReceiver**, **ContentProvider** sẽ cùng chạy trên một Linux Process, trừ khi chúng ta muốn khai báo một Process riêng trong file **AndroidManifest** như sau:

```xml
<activity
    android:process="new_process_name"
    ...>
</activity>
```

Mặc định, process có tên trùng với **app ID** được khai báo trong `build.gradle` file. Cả application và 4 component chính đều có tag `android:process`. Chính vì vậy, nếu bạn khai báo `android:process` cho tag `<application>`, process name đấy sẽ được áp dụng cho toàn bộ các component của application đó.

# Mức độ ưu tiên

Chúng ta không thể tự quản lý Process lifetime. Android sẽ tự tính toán xem **component** nào của các ứng dụng **đang chạy**, **tầm quan trọng** của chúng với user, và **bộ nhớ còn lại** là bao nhiêu để quyết định Process lifetime.

Khi Android không đủ tài nguyên, nó sẽ shut down một Process, và tất nhiên là các component đang chạy trên Process đó cũng sẽ bị destroy theo. Vậy yếu tố nào quyết định Process nào sẽ bị shut down?

Android sẽ ưu tiên chúng dựa vào mức độ quan trọng đối với user. Nó chia thành 4 loại process với mức độ ưu tiên như sau:

## Foreground process

Đây là loại Process có mức độ ưu tiên cao nhất. Nó chứa các component mà user **đang làm gì đó**, ví dụ như:

* **Activity** trên cùng của screen mà user đang tương tác, method `onResume()` đã được gọi.
* **BroadcastReceiver** đang chạy, method `onReceive()` đang thực thi.
* **Service** đang thực thi code trong một trong những callback của nó: `onCreate()`, `onStart()` hoặc `onDestroy()`.

Chỉ có một vài Process như này trong hệ thống, và nó chỉ bị kill khi bộ nhớ thấp đến mức chính những Process này cũng không thể chạy tiếp.

## Visible process

Process này thực hiện các tác vụ mà user **nhận biết được**. Vì vậy, nếu chúng bị kill cũng sẽ ảnh hưởng đến trải nghiệm. Ví dụ như:

* **Activity** đang hiển thị trên screen nhưng không còn trên foreground, method `onPause()` đã được gọi. Ví dụ như có một Activity khác hiển thị dưới dạng dialog che một phần của nó chẳng hạn.
* **Foreground Service** đang chạy, thông qua method `startForeground()`. Khi đó, user cũng có thể thấy được nó.
* Một service nào đó đang chạy tính năng nào đó mà user thấy, ví dụ như live wallpaper hoặc bàn phím.

## Service process

Process này chứa một Service chạy bằng method `startService()`. User không nhìn thấy trực tiếp, chỉ là nó đang thực hiện **các tác vụ mà user quan tâm**, ví dụ như upload hoặc download data dưới background.

Service chạy trong một thời gian dài, ví dụ như hơn 30 phút, có thể sẽ bị giảm mức độ quan trọng để đưa vào cache.

## Cached process

Đây là Process **không còn cần thiết nữa**, hệ thống có thể **thoải mái kill** nó không do dự khi cần thêm tài nguyên như bộ nhớ.

Một hệ thống tốt sẽ có nhiều Cached process để phục vụ cho việc chuyển đổi giữa các app được hiệu quả, và thường xuyên kill các Cached app khi cần thiết.

Android sử dụng **LRU Cache** (Least Recently Used Cache) để quản lý các Cached process, và nó sẽ kill các process ít được sử dụng nhất trong thời gian gần đây.

Tựu chung lại, chúng ta phải nắm được các component như **Activity**, **Service** và **BroadcastReceiver** có sự ảnh hưởng khác nhau như thế nào đến mức độ ưu tiên đó, chọn đúng component để sử dụng trong use-case của mình, tránh process bị kill khi đang thực hiện một tác vụ quan trọng.

# Inter-Process Communication (IPC)

**Inter-Process Communication** hay còn gọi là Giao tiếp liên tiến trình, là cơ chế cho phép các process **giao tiếp** và **đồng bộ hóa** hành động của chúng trong Android.

Mỗi app chạy trong một process riêng biệt, nhưng nhiều app cần giao tiếp với nhau để **chia sẻ dữ liệu** hoặc thực hiện **các tác vụ kết hợp**, vậy nên IPC cung cấp các phương thức để các process giao tiếp với nhau một cách an toàn và hiệu quả.

## Intent

**Intent** là cơ chế chính thống để giao tiếp bất đồng bộ giữa các **Activity** và **BroadcastReceiver**. Chúng ta có thể dùng  `sendBroadcast`, `sendOrderedBroadcast` hoặc explicit intent tuỳ theo nhu cầu.

## Android Interface Definition Language (AIDL)

**AIDL** là tool được sử dụng để định nghĩa interface giữa các ứng dụng Android. AIDL cho phép các ứng dụng giao tiếp với nhau một cách an toàn và hiệu quả, bất kể chúng được viết bằng ngôn ngữ lập trình nào.

## Messenger

**Messenger** là một class trong Android SDK cho phép các ứng dụng giao tiếp với nhau bằng cách **gửi và nhận tin nhắn**. Messenger cung cấp một interface đơn giản và dễ sử dụng để giao tiếp giữa các ứng dụng.

Điểm khác biệt giữa AIDL và Messenger là bạn có thể dùng AIDL cho các **tác vụ đồng thời**, còn Messenger chỉ dùng cho các **tác vụ tuần tự**.

## Broadcast Receiver

**BroadcastReceiver** sẽ xử lý các yêu cầu bất đồng bộ từ Intent. Mặc định, receiver có thể được gọi bởi bất kỳ app nào khác. Nếu bạn có ý định chỉ dùng BroadcastReceiver cho một ứng dụng cụ thể, bạn có thể áp dụng bảo mật cho receiver bằng cách sử dụng tag `<receiver>` trong AndroidManifest. Nó giúp ngăn các ứng dụng không có quyền gửi Intent đến BroadcastReceiver.

# Reference

* https://developer.android.com/guide/components/processes-and-threads
* https://developer.android.com/guide/components/activities/process-lifecycle
* https://developer.android.com/privacy-and-security/security-tips#interprocess-communication
