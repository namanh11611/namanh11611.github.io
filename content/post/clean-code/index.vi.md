---
title: "3 cách mình áp dụng để code gọn gàng sạch đẹp hơn"
description: "Có lẽ Clean Code là một vấn đề kinh điển trong ngành lập trình, trong bài viết này mình muốn chia sẻ một vài tip mà mình hay áp dụng để giữ code luôn được gọn gàng sạch đẹp."
date: 2024-05-01T00:00:00+07:00
slug: clean-code
image: tidy.jpg
toc: true
categories: Technical
tags: [Clean Code]
---

Có lẽ **Clean Code** là một vấn đề kinh điển trong ngành lập trình, mình đã có lần nhắc đến cuốn sách **Clean Code** của **Uncle Bob** trong bài viết [**Những điều giá như mình biết từ khi còn là Junior**](https://viblo.asia/p/nhung-dieu-gia-nhu-minh-biet-tu-khi-con-la-junior-bXP4Wx6oJ7G). Tuy nhiên thì lần này mình sẽ không viết lại những nội dung trong đó nữa, các bạn nên đọc trực tiếp sách để có trải nghiệm tốt nhất. Mình chỉ muốn chia sẻ một vài tip mà mình hay áp dụng để giữ code luôn được **gọn gàng sạch đẹp**.

# Dùng return để tránh if/else hell

Không biết bạn đã bao giờ gặp **if/else hell** kiểu như này trong code chưa?

```java
void doSomething() {
    if (condition1) {
        doFirstTask()
        if (condition2 != null) {
            doSecondTask()
            if (condition3.isNotEmpty()) {
                doThirdTask()
            }
        }
    }
}
```

Về lý thuyết thì đoạn code trên không có vấn đề gì cả. Thế nhưng mình lại không thích nhìn kiểu code lồng nhau nhiều lớp như vậy. Vậy nên mình thường sẽ dùng `return` để tránh phải viết quá nhiều dấu ngoặc `{}` lồng nhau. Chúng ta có thể refactor lại đoạn code trên như sau:

```java
void doSomething() {
    if (!condition1) return
    doFirstTask()
    if (condition2 == null) return
    doSecondTask()
    if (condition3.isEmpty()) return
    doThirdTask()
}
```

Về mặt logic code thì tương đương nhau, tuy nhiên thì về mặt ý nghĩa thì hơi khác một chút. Với trường hợp ban đầu, khi đồng nghiệp đọc code, họ sẽ hiểu là *"Nếu thoả mãn điều kiện này thì code sẽ xử lý tiếp như thế này"*. Còn với trường hợp thứ hai, họ sẽ hiểu thành *"Nếu không thoả mãn điều kiện này thì code sẽ không xử lý tiếp"*. Tư duy theo cách thứ 2 đôi khi hơi ngược một chút, nên bạn cũng **áp dụng tuỳ trường hợp** thôi nhé, chứ đừng áp dụng máy móc cho tất cả các trường hợp.

# Dùng Map hoặc Set thay cho List

**Map** và **Set** là 2 cấu trúc dữ liệu đặc biệt mà chắc chắn các bạn đã được học ở đại học. Tuy nhiên, trong dự án, mọi người thường luôn luôn sử dụng **List** vì nó "tiện". Với mình thì trước khi chọn loại dữ liệu cho một list, mình luôn đặt câu hỏi liệu dùng Map hay Set có giúp **code ngắn gọn** và **tăng performance** hơn không? Nếu có, chắc chắn mình sẽ **ưu tiên dùng 2 loại dữ liệu** này trước.

Ví dụ, mình có một biến `existingValueList` gồm một list các value đã tồn tại, khi user nhập một value mới, mình cần check xem nó đã tồn tại chưa, nếu có thì trả về lỗi. Nếu `existingValueList` có kiểu dữ liệu là `List`, mình sẽ cần viết hàm check như sau:

```java
boolean isExisting = existingValueList.contains(value)
```

Nhưng nếu kiểu dữ liệu là Set, chúng ta sẽ viết như sau:

```java
boolean isExisting = existingValueSet.contains(value)
```

*"Ủa, rồi khác gì nhau?"*. Đúng là cách viết thì không khác gì nhau, nhưng sự khác nhau nằm trong chính hàm `contains()`, run time của `List` sẽ là `O(n)`, trong khi đó của `Set` chỉ là `O(1)`.

99% là bạn đã biết điều này, nhưng đôi khi bạn sẽ sử dụng **List** như một thói quen. Vậy nên lần sau, trước khi tạo một **List**, bạn hãy chậm lại một nhịp để suy nghĩ xem có nên dùng **Map** hay **Set** không nhé.

Ngoài ra chúng ta còn nhiều cấu trúc dữ liệu khác như **Stack**, **Queue**, **Tree**... Nhưng trong dự án thực tế thì mình hiếm gặp những use-case cần áp dụng bọn này.

# Đừng comment code không dùng nữa, hay xoá nó

Mình thấy một số bạn có thói quen khi sửa một tính năng gì đấy, thường sẽ **comment lại code cũ** thay vì xoá nó. Hoặc có những **function cũ không dùng nữa**, nhưng cũng chẳng buồn comment hay xoá gì, cứ để nó nằm vậy trơ gan cùng tuế nguyệt. Nhưng tin mình đi, **vài tháng** hoặc thậm chí **vài năm** sau, chắc bạn sẽ chẳng bao giờ uncomment lại đống code đấy đâu. Mỗi lần vài dòng thôi, nhưng qua năm tháng sẽ thành một núi **dead code** khổng lồ.

Nếu muốn xem lại code cũ, [**Git**](https://viblo.asia/p/quy-trinh-lam-viec-chuan-chi-voi-git-eW65G10RZDO) đã có thể giúp bạn rất tốt rồi. Vậy nên hãy mạnh dạn xoá nó, không chỉ là **vài dòng code**, **một function**, mà còn cả **resource** của app mà bạn không dùng nữa. Điều này sẽ giúp giảm dung lượng của source code đi nhiều.

Nếu bạn có tip nào hay ho mà đang áp dụng, hãy chia sẻ thêm với mình nhé.
