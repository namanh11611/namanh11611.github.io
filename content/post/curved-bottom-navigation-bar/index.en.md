---
title: Custom Curved Bottom Navigation Bar trong Flutter
description: Ở dự án gần đây, mình được toàn quyền quyết định UI/UX của app. Vậy nên, mình muốn cải thiện thêm Bottom Navigation Bar, muốn nó trônng màu mè hơn một chút, thay vì dùng Bottom Navigation Bar mặc định của Flutter.
date: 2022-10-17T22:36:00+07:00
slug: curved-bottom-navigation-bar
image: navigation.jpeg
categories: Technical
tags: [Flutter, Bottom Navigation Bar]
---

# Đi tìm một Bottom Navigation Bar ưng ý
Gần đây, trong quá trình làm dự án ở công ty, mình được tìm hiểu thêm về **Flutter**. Đó đúng là một framework giúp build mobile app cross platform rất nhanh. Theo một số bài đo đạc đánh giá thì **Flutter** còn có hiệu năng gần bằng native (**Kotlin** và **Swift**), đương nhiên là vượt xa **React Native**.

Ở dự án này, mình được toàn quyền quyết định UI/UX của app. Vậy nên, mình muốn cải thiện thêm Bottom Navigation Bar, muốn nó trônng màu mè hơn một chút, thay vì dùng Bottom Navigation Bar mặc định của Flutter. Thế là sau một hồi lang thang trên Google, mình tìm thấy package [curved_navigation_bar](https://pub.dev/packages/curved_navigation_bar), tuy nhiên thì package này lại chỉ cho hiển thị icon ở Bottom Navigation Bar item như thế này.

![origin](https://images.viblo.asia/2f2337ec-a728-4e44-abda-2a02a5ef8ffa.jpg)

<br>

Phần code example:
```dart
bottomNavigationBar: CurvedNavigationBar(
  items: <Widget>[
    Icon(Icons.add, size: 30),
    Icon(Icons.list, size: 30),
    Icon(Icons.compare_arrows, size: 30),
    Icon(Icons.call_split, size: 30),
    Icon(Icons.perm_identity, size: 30),
  ],
)
```

<br>

Mình muốn thêm label cho từng item thì chỉ có thể thêm bằng cách thay đổi mỗi item thành một `Column` widget:
```dart
bottomNavigationBar: CurvedNavigationBar(
  items: <Widget>[
    Column(
      children: [Icon(Icons.add, size: 30), Text("Add")],
    ),
    Column(
      children: [Icon(Icons.list, size: 30), Text("List")],
    ),
    Column(
      children: [Icon(Icons.compare_arrows, size: 30), Text("Compare")],
    ),
    Column(
      children: [Icon(Icons.call_split, size: 30), Text("Split")],
    ),
    Column(
      children: [Icon(Icons.perm_identity, size: 30), Text("Person")],
    ),
  ],
)
 ```
 
 <br>
 
Và kết quả sẽ trở thành như thế này:
![origin label](https://images.viblo.asia/1ba2a125-2f8e-4b3d-ba6c-df47591713fa.jpg)

Trông nó không được đẹp lắm nhỉ 🤔

Mình thì muốn label của selected item hiển thị ở dưới vùng có background màu trắng, và khi mình chọn một Bottom Navigation Bar item khác, label vẫn sẽ nằm yên ở đó. Vậy thì không còn cách nào khác là phải custom lại package này rồi. Đó là lúc chúng ta đến với bước 2 của bài viết này.
# Custom Curved Navigation Bar
Mình quyết định tạo một package mới linh động hơn, có thể phù hợp với cả hai dạng là item có label và item không có label.

Tadaa... Và đây là package của mình: [curved_labeled_navigation_bar](https://pub.dev/packages/curved_labeled_navigation_bar)

Sau đây, mình sẽ hướng dẫn bạn cách implement của cả hai dạng.

## Navigation Bar có label
Mình có tạo ra một Widget mới là `CurvedNavigationBarItem` để wrap cả Icon và label.

Code example:
```dart
bottomNavigationBar: CurvedNavigationBar(
    items: [
      CurvedNavigationBarItem(
        child: Icon(Icons.home_outlined),
        label: 'Home',
      ),
      CurvedNavigationBarItem(
        child: Icon(Icons.search),
        label: 'Search',
      ),
      CurvedNavigationBarItem(
        child: Icon(Icons.chat_bubble_outline),
        label: 'Chat',
      ),
      CurvedNavigationBarItem(
        child: Icon(Icons.newspaper),
        label: 'Feed',
      ),
      CurvedNavigationBarItem(
        child: Icon(Icons.perm_identity),
        label: 'Personal',
      ),
    ],
)
```

<br>

UI demo:

<br>

![label](https://images.viblo.asia/78add20a-e7af-4927-bb7f-b900f972ad0e.jpg)

## Navigation Bar không có label
Code example:
```dart
bottomNavigationBar: CurvedNavigationBar(
    items: [
      CurvedNavigationBarItem(
        child: Icon(Icons.home_outlined),
      ),
      CurvedNavigationBarItem(
        child: Icon(Icons.search),
      ),
      CurvedNavigationBarItem(
        child: Icon(Icons.chat_bubble_outline),
      ),
      CurvedNavigationBarItem(
        child: Icon(Icons.newspaper),
      ),
      CurvedNavigationBarItem(
        child: Icon(Icons.perm_identity),
      ),
    ],
)
```

<br>

UI demo:

<br>

![no label](https://images.viblo.asia/c8e6ef01-4c97-4789-88f8-d46c4513ca94.jpg)

# Lời kết
Hy vọng package nhỏ này của mình giúp ích được cho những bạn đang muốn có một Bottom Navigation Bar khác lạ. Bạn có thể ủng hộ tinh thần mình bằng cách tặng mình một like, share nhé. Nếu bạn có ý tưởng gì để mình có thể cải thiện hơn nữa thì cũng đừng ngần ngại chia sẻ với mình.

* Link package: https://pub.dev/packages/curved_labeled_navigation_bar
* Github: https://github.com/namanh11611/curved_labeled_navigation_bar

Mình chân thành cảm ơn!
