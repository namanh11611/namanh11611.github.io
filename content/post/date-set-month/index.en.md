---
title: "Cái bẫy chuột trong hàm Date.setMonth() của Javascript"
description: "Vào một ngày đẹp trời 29/3/2019, đang OT một cách vui vẻ và đầy hào hứng, tôi bỗng phát hiện ra một bug liên quan đến hiển thị ngày tháng trên Front-end."
date: 2019-03-30T01:33:00+07:00
image: month.jpeg
categories: Technical
tags: [JavaScript]
---

## Hoàn cảnh đáng thương
Chắc hẳn các lập trình viên JavaScript thì không còn xa lạ gì `Date`, đối tượng dùng để nhẩm ngày, đếm tháng, tính năm. Vào một ngày đẹp trời 29/3/2019, đang OT một cách vui vẻ và đầy hào hứng, tôi bỗng phát hiện ra một bug liên quan đến hiển thị ngày tháng trên Front-end. Ủa, hàm này truyền tham số là **1** thì trả về **Mar** (tháng 3), truyền tham số là **0** thì trả về **Jan** (tháng 1), trong khi kết quả mình muốn nhận là **Feb** (tháng 2). Chơi gì kỳ vậy???

Nhẹ nhàng bật F12 lên và debug thì phát hiện ra nguyên nhân là do hàm `Date.getMonth()` trả về kết quả kỳ cục như vậy.

Lúc ấy cũng đã 9h tối, không còn tin vào sự tỉnh táo của mình nên tôi chuyển qua tab Console để gõ thử xem liệu kết quả như thế nào? Liệu có phải do hàm `getMonth()` trong project bị ông nào custom rồi không?
Và kết quả nhận được như sau (kết quả được tái hiện lại vào sáng 30/3):
![](https://images.viblo.asia/91fc5303-ed6d-470d-92ff-b0bdcd46c988.PNG)

Các bạn có thể gõ thử trên console của mình để kiểm chứng:
```javascript
var date = new Date();
date.setMonth(1);
date.getMonth();
date.setMonth(0);
date.getMonth();
```

## Bẫy chuột này lớn quá
Hoá ra là sau khi gọi hàm `setMonth(1)`, `date` không được set thành tháng 2 (index = 1) mà chỉ bị lùi đi 28 ngày (từ ngày 30/3 thành ngày 2/3)
![](https://images.viblo.asia/5da81bdf-8f4e-4a61-af4b-8a21e9f15e4d.PNG)

Các bạn có thể tham khảo thông tin về hàm `setMonth()` tại [đây](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/setMonth).

Đọc kỹ phần [Description](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/setMonth#Description), chúng ta mới thấy rằng hàm `setMonth()` sẽ dùng chính ngày hiện tại của tháng cũ để set ngày cho tháng mới, nếu vượt quá ngày của tháng mới thì sẽ cộng tiếp sang ngày của tháng tiếp theo. Ví dụ, ngày hiện tại là 30, 31 thì `setMonth(1)` chắc chắn sẽ bị tính sang tháng 3, vì tháng 2 chỉ có tối đa 29 ngày. Ở đây, ngày hiện tại của chúng ta là 30/3, thế nên khi set về tháng 2/2019 (có 28 ngày) sẽ bị tính thêm 2 ngày là thành ngày 2/3.

Quả thật, cái bẫy chuột này ít nhất đã bẫy được một con chuột béo là mình rồi.

## Giải pháp
Từ đó, chúng ta rút ra rằng, để `setMonth()`, `getMonth()` được thoải mái nhất, tốt nhất là nên dùng ngày mùng 1.
```javascript
var date = new Date('March 1, 2019 00:00:01');
```

Lúc đó thì chúng ta có thể thoải mái `setMonth()`, `getMonth()` theo ý mình.

Chúc các bạn luôn đọc kỹ document và code ít bug. Đừng dính bẫy như mình.
