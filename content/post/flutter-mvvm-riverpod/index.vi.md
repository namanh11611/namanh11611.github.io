---
title: "Flutter MVVM Riverpod Starter: Tạo App Siêu Tốc cho Vibe Coder"
description: "Dạo gần đây mình dành thời gian build một số app như Habit Tree và Speakie, hoặc thỉnh thoảng làm một số dự án outsource cho khách hàng. Mỗi lần bắt đầu một dự án mới là mình phải mất cả ngày để set up architecture, routing, auth, state management... cho app. Vậy nên, xuất phát nhu cầu thực tế, mình đã nghĩ đến việc build một base project Flutter gọn nhẹ để có thể bắt đầu nhanh nhất có thể."
date: 2025-05-11T00:00:00+07:00
slug: flutter-mvvm-riverpod
image: fmr.webp
toc: true
categories: [Technical, Flutter]
tags: [Flutter, MVVM, Riverpod]
---

*Photo by [7](https://unsplash.com/@4000km?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/a-silver-and-red-train-traveling-down-train-tracks-u6QssbF_9JM?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)*

Dạo gần đây mình dành thời gian build một số app như [**Habit Tree**](https://habittree.xyz) và [**Speakie**](https://speakie.xyz) (bạn có thể xem đầy đủ tại [**Areser**](https://areser.net)), hoặc thỉnh thoảng làm một số dự án outsource cho khách hàng. Mỗi lần bắt đầu một dự án mới là mình phải mất cả ngày để set up architecture, routing, auth, state management... cho app. Vậy nên, xuất phát nhu cầu thực tế, mình đã nghĩ đến việc build một **base project Flutter** gọn nhẹ để có thể bắt đầu nhanh nhất có thể. Và đó chính là lý do mà **Flutter MVVM Riverpod Starter** ra đời.

> Bạn có thể xem ngay tại link: https://github.com/namanh11611/flutter_mvvm_riverpod

# 🚀 Tại sao lại chọn starter này?

Mục tiêu của mình là tạo ra một template nhẹ, nhưng đủ sức mạnh cho **indie hacker** và **solo developer**. Chính vì vậy mà mình đã không chọn **Clean Architecture**, nó quá cồng kềnh với đối tượng mà mình nhắm tới. Hãy thử nghĩ xem, với dự án mình làm một mình hoặc team nhỏ chỉ 2-3 người, **99.99%** là team sẽ không cần đến **Abtract Repository** để cho multiple implementation. **Andrea** - một Flutter developer nổi tiếng - cũng đã nhắc đến vấn đề này trong bài viết [Flutter App Architecture: The Repository Pattern](https://codewithandrea.com/articles/flutter-repository-pattern/#repositories-may-not-need-an-abstract-class).

Sau khi cân đo đong đếm, mình chọn kiến trúc **MVVM** (Model - View - ViewModel), chỉ đơn giản là dùng ViewModel để tách biệt logic ra khỏi UI.

Mình cũng chọn **Riverpod**, một state management giúp quản lý state linh hoạt. Mình biết tỷ lệ các bạn dùng **BloC** sẽ đông hơn, mình không đánh giá cái nào tốt hơn cái nào, chỉ đơn giản là mình thấy **Riverpod** viết code ngắn gọn hơn **BloC**.

Tiếp theo là phần Backend, mình biết đa số solo developer sẽ tích hợp **Firebase**, nhưng nó có một vấn đề là ... **ĐẮT** (khi dự án của bạn bắt đầu scale). Vậy nên mình đã chọn **Supabase**, cũng tương tự như **Firebase** nhưng RẺ hơn. Mình đã tích hợp sẵn code setup và một số function authentication.

Nếu anh em muốn kiếm tiền từ app thì cũng sẽ không thể thiếu được **RevenueCat**, giúp quản lý in-app purchase và subscription.

Ngoài ra còn Dark/Light Theme, Localization, routing, local storage, analytics, crashlytics... tất cả đều đã sẵn sàng.

# 📚 Thư viện sử dụng

| Mục đích       | Thư viện                                                   |
|----------------|------------------------------------------------------------|
| State          | `flutter_riverpod`, `riverpod_annotation`                  |
| Auth & Backend | `supabase_flutter`, `google_sign_in`, `sign_in_with_apple` |
| Navigation     | `go_router`                                                |
| UI/UX          | `google_fonts`, `flutter_svg`, `shimmer`, `lottie`         |
| Storage        | `shared_preferences`, `sqflite`                            |
| HTTP           | `dio`, `connectivity_plus`                                 |
| Utils          | `uuid`, `envied`, `easy_localization`                      |
| Monetization   | `purchases_flutter`, `in_app_purchase`                     |
| Analytics      | `firebase_analytics`, `firebase_crashlytics`               |

# 🏗 Kiến trúc project

Về kiến trúc project, mình chọn Feature-first (layer folder bên trong feature folder). Với mỗi feature như `authentication`, `onboarding`... mình sẽ tạo 1 folder bên trong `features`. Sau đó bên trong folder đó, mình lại tạo các layer folder như `model`, `repository`, `ui`.

```bash
lib/
├── constants/         # Hằng số, config chung
├── environment/       # Biến môi trường
├── extensions/        # Extension/helper methods
├── features/          # Module chức năng theo folder
│   ├── authentication/
│       ├── model/
│       ├── repository/
│       ├── ui/
│   ├── onboarding/
│   ├── home/
│   ├── profile/
│   ├── premium/
├── routing/           # Cấu hình route với go_router
├── theme/             # Config giao diện
└── utils/             # Tiện ích chung
```

Bạn có thể thấy không có layer như `domain` hay `data`.

Thường khi dùng Clean Architecture, `domain` của anh em sẽ có `model`, `abtract repository`, `use case`, nhưng giờ đây với việc không cần đến `abtract repository` và `use case`, mình có thể tối giản nó chỉ còn `model`.

Còn với `data`, với những feature chỉ cần truy cập tới remote hoặc local data, mình sẽ gọi hàm trực tiếp trong `repository`. Còn những feature cần cả 2 loại data, mình sẽ cân nhắc tạo `data source` nếu cần thiết.

# 🎉 Lời kết

Phong trào **vibe coding** ngày càng lên cao, vậy nhưng mình vẫn muốn **vide coding một cách có kiểm soát**. Với base project này, mình có thể hướng dẫn AI code theo cách mà mình muốn. Nếu nó có đi chệch đường ray thì mình cũng nhanh chóng nắn chỉnh lại được.

Như đã nói từ đầu, project này dành cho anh em **indie hacker** và **solo developer** muốn launch **MVP** nhanh, nó giúp bạn tập trung vào business logic thay vì mất cả ngày setup và viết code. Vậy nên có thể nó sẽ chưa phù hợp với team của bạn nếu như team đông người và cần một base project chuẩn **Clean Architecture**.

Nếu bạn thấy nó có ích, đừng ngần ngại tặng mình 1 star nhé!

> Link GitHub: https://github.com/namanh11611/flutter_mvvm_riverpod
