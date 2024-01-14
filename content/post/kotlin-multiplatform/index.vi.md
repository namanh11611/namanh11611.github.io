---
title: "Kotlin Multiplatform - Kẻ ngáng đường Flutter, React Native?"
description: "Phải chia sẻ thật với chư vị huynh đệ rằng ấn tượng ban đầu của tại hạ với Kotlin Multiplatform không tốt lắm."
date: 2023-05-07T13:37:00+07:00
image: kotlin.jpeg
categories: Technical
tags: [Android, iOS, Kotlin]
---

## Lời mở đầu
Lúc mới nghe đến **Kotlin Multiplatform** (từ nay xin được viết tắt là **KM**), tại hạ trộm nghĩ *"Công nghệ quái gì mà chỉ share được mỗi logic code? Muốn code nhanh thì người ta làm luôn bằng Flutter cho rồi, performance có kém gì đâu"*. Thế rồi dòng đời xô đẩy, tại hạ được tham dự vào một project đang áp dụng KM, quả nhiên đã được mở mang tầm mắt về những ưu cũng như nhược điểm của nó. Nay mượn chén rượu dưới đêm trăng, xin được lạm bàn mấy lời, hầu chuyện chư vị huynh đệ.
## Kotlin Multiplatform là cái gì?
KM giúp việc phát triển các cross-platform projects trở nên nhẹ tựa lông hồng. Nó giúp giảm thời gian viết và maintain code nhưng vẫn giữ được những ưu điểm của native cho từng platform. Kotlin app có thể chạy trên Android, iOS, macOS, Windows, Linux, watchOS...

![](https://images.viblo.asia/1eea8f44-e16d-4702-8266-30d4d82aec2a.png)

Mặc dù KM vẫn đang trong giai đoạn Beta nhưng nó cũng đã khá ổn định và được áp dụng ở nhiều công ty như 9gag, Netflix, Philips, Baidu, VMWare, Quizlet, Memrise...

Trở lại với câu hỏi đầu bài, vậy thì KM có những ưu điểm gì so với các cross-platform framework khác?

![](https://images.viblo.asia/b22cf0ae-b2b4-4b3a-9eab-c8618f0afa66.png)

Thực sự thì KM phù hợp với những project có **logic code phức tạp**. Chúng ta đều biết rằng viết càng nhiều code thì càng sinh ra nhiều bug, chỉ có một cách duy nhất để không gây ra bug đó là [No Code](https://github.com/kelseyhightower/nocode). Vậy nên thay vì việc Android dev viết logic code cho Android, iOS dev viết logic code cho iOS, thì giờ đây chỉ cần viết logic code 1 lần cho KM cũng đã giúp giảm một nửa lượng bug phát sinh rồi.

Ví dụ với một project cần 10 Android dev và 10 iOS dev, nếu áp dụng KM thì bây giờ chỉ cần 5 Android dev, 5 iOS dev và 5 dev code KM (có thể là Android dev luôn). Như vậy là project đã giảm được 25% resource rồi. So với Flutter, đương nhiên là chúng ta vẫn tốn resource hơn, nhưng đánh đổi lại là native performance.
## Use Cases
### Android và iOS app
Chúng ta có thể share business logic code giữa các nền tảng để build một cross-platform mobile app. Đây cũng chính là use case chính của KM tại thời điểm này. Ví dụ các bạn có thể viết code từ **ViewModel** đến **Data layer** trong KM, còn bên phía Android và iOS app chỉ đơn thuần là build UI bằng **Jetpack Compose** và **SwiftUI**.
### Full-stack web app
Có thể các bạn không biết nhưng bây giờ Kotlin đã có thể dùng để viết Front-end web app. Như vậy hoàn toàn có thể build một full-stack web app với server dùng Kotlin/JVM và web client dùng Kotlin/JS. Do đó, chúng ta cũng có thể dùng KM để tái sử dụng logic code cho cả server và client.
### Multiplatform libraries
Nếu Kotlin đã có thể build được cho cả Android, iOS, và web thì tại sao chúng ta không viết một library mà có thể dùng cho cả 3 platform? Với những project mà các bạn cần build cho cả app và web, hãy thử dùng KM để viết common code, publish nó như một library, và ở app hay web, bạn có thể import nó như một dependency.
### Common code cho mobile và web app
Thậm chí, bạn không cần tạo library mà có thể viết trực tiếp code để share giữa Android, iOS, desktop và web app. Cách làm này giúp giảm khối lượng công việc cho anh em mobile và front-end web dev, vì chúng ta chỉ phải viết logic code một lần duy nhất. Đồng thời nó cũng giúp giảm bug, giảm thời gian viết test.
## Cách hoạt động
Vậy làm thế nào mà KM có thể shared code cho tất cả các platform?
![](https://images.viblo.asia/64059043-ba35-4f8c-ad14-bf0fec7df6c8.png)
- Common Kotlin là phần code bao gồm core libraries, có thể chạy trên tất cả platform.
- Với KM libraries, chúng ta có thể tái sử dụng **multiplatform logic trong common** và **platform-specific code**.
- **Platform-specific code** (Kotlin/JVM, Kotlin/JS, Kotlin/Native) bao gồm extensions cho Kotlin, platform-specific libraries và tools.
- Với từng platform, chúng ta có thể tận dụng **platform native code** (JVM, JS và Native).

Ví dụ, để viết một function generate UUID cho cả Android và iOS, chúng ta có thể khai báo một function với keyword `expect` trong common module như sau:
```kotlin
// Common
expect fun randomUUID(): String
```
Sau đó, với mỗi platform, chúng ta lại viết các function implement với keyword `actual` như sau:
```kotlin
// Android
import java.util.*

actual fun randomUUID() = UUID.randomUUID().toString()
```
```swift
// iOS
import platform.Foundation.NSUUID

actual fun randomUUID(): String = NSUUID().UUIDString()
```
## Bóc trần công nghệ
Project của mình apply KM cho Android và iOS app. Mình đặt ra câu hỏi là tại sao Kotlin có thể integrate được với iOS app viết bằng Swift?

![](https://images.viblo.asia/6a6a746d-dc3e-4f60-8a43-49c3496c3ce9.png)

Và câu trả lời của Jet Brains là họ dùng **Kotlin/Native** để compile **Kotlin code** sang **native binaries**, do đó có thể chạy Kotlin code mà không cần virtual machine. Nó bao gồm backend dựa trên [LLVM](https://llvm.org) cho Kotlin compiler và native implementation của Kotlin standard library.

Kotlin/Native được thiết kế để compile cho các platform mà không chạy được virtual machine như là embedded devices hoặc iOS.

Còn bên phía Android, Kotlin code được compile sang **JVM bytecode** bằng **Kotlin/JVM**.
## Lời kết
Trong tương lai, Jet Brains không chỉ dừng lại với Kotlin Multiplatform mà đang có kế hoạch phát triển [Compose Multiplatform](https://www.jetbrains.com/lp/compose-multiplatform/), nghĩa là thay vì share logic code thì có thể share cả UI như các cross-platform framework khác (Flutter, React Native). Đấy là cả một chặng đường dài, chư vị huynh đệ hãy cùng ngồi xuống đây ăn miếng thịt to, uống bát rượu lớn rồi cùng chờ xem hồi sau sẽ rõ.
## Reference
* https://kotlinlang.org/docs/multiplatform.html
* https://kotlinlang.org/docs/multiplatform-mobile-faq.html#what-is-kotlin-native-and-how-does-it-relate-to-kotlin-multiplatform
