---
title: "LeetCode: Binary Search template chinh phục mọi problem"
description: "Gần đây mình có tham gia luyện LeetCode để tìm niềm vui sau những giờ làm việc căng thẳng. Thỉnh thoảng mình gặp khúc mắc khi đụng một vài bài cần phải áp dụng Binary Search để giải. Cho đến một ngày, mình tìm được một công thức thần thánh..."
date: 2024-02-17T00:00:00+07:00
slug: binary-search
image: binary_search.webp
toc: true
math: true
categories: [Technical]
tags: [Binary Search, Algorithm, LeetCode, Java]
---

# Vấn đề

Gần đây mình có tham gia luyện **LeetCode** để tìm niềm vui sau những giờ làm việc căng thẳng. Thỉnh thoảng mình gặp một vài bài cần phải áp dụng **Binary Search** để giải. Dành cho anh em nào chưa biết thì:

> **Binary Search** là thuật toán tìm kiếm một giá trị **target** trong mảng đã được sắp xếp. Nó sẽ so sánh **target** với phần tử chính giữa của mảng (**middle**), nếu **target** != **middle** thì sẽ tiếp tục tìm kiếm trong một nửa mảng có giá trị nhỏ hơn hoặc lớn hơn **middle**, tuỳ theo điều kiện bài toán. Quá trình này được lặp lại cho đến khi tìm được vị trí của phần tử bằng với **target**.
>
> Sau mỗi bước tìm kiếm, số lượng phần tử sẽ giảm đi một nửa. Vậy nên độ phức tạp của thuật toán này là **O(log n)**.

Tuy nhiên, khi áp dụng thuật toán này thì mình gặp một số câu hỏi cần trả lời là:

- Khi nào nên dừng vòng lặp? Khi `left < right` hay `left <= right`? (`left` và `right` là 2 giá trị biên của mảng sau mỗi vòng lặp)
- Nên update các giá trị biên như thế nào? Dùng `left = mid` hay `left = mid + 1`? Dùng `right = mid` hay `right = mid + 1`?

Nhìn có vẻ đơn giản nhưng việc chọn sai công thức có thể dẫn đến kết quả sai, rồi mình lại phải sửa đi sửa lại rất mất thời gian.

Cho đến một ngày, mình tìm được một công thức thần thánh...

# Template đơn giản

Ta... da... Tác giả chia sẻ chi tiết về template rất đơn giản nhưng thần thánh ở [bài viết này](https://leetcode.com/discuss/general-discussion/786126/python-powerful-ultimate-binary-search-template-solved-many-problems).

Theo đó, giả sử chúng ta có một **search space**, nó có thể là một mảng hoặc một khoảng giá trị, thường sẽ được sắp xếp theo thứ tự **tăng dần**. Nhiệm vụ của chúng ta là tìm giá trị `num` **nhỏ nhất** sao cho hàm `condition(num)` trả về giá trị `true`.

Template của chúng ta như sau (mình sẽ viết bằng code **Java** cho đa số mọi người dễ đọc nhé):

```java
int binarySearch() {
    // Bạn tự định nghĩa giá trị khởi tạo của 2 biên
    int left = min(searchSpace), right = max(searchSpace);
    while (left < right) {
        // Viết như thế này để tránh tràn số
        int mid = left + (right - left) / 2;
        if (condition(mid)) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    // Có thể là left hoặc left - 1 tuỳ yêu cầu bài toán
    return left;
}

boolean condition(int num) {
    // Code check condition
}
```

Bạn có thể áp dụng template này vào tất cả các bài **binary search**, khi đó bạn cần thực hiện 3 việc:

- Tìm giá trị `left` và `right` phù hợp. Bạn cần chắc chắn rằng 2 giá trị biên này đã **bao gồm tất cả các giá trị** có thể là kết quả của bài toán.
- Định nghĩa hàm `condition` chính xác. Đây thường là **phần khó nhất** trong các bài binary search phức tạp.
- Chọn giá trị trả về là `left` hay `left - 1`. Nhớ rằng khi kết thúc vòng lặp `while`, `left` là **index của phần tử nhỏ nhất** thoả mãn hàm `condition` trả về `true`.

Hãy tưởng tượng, chúng ta có dãy số với index từ **0 đến n** như sau:

```text
Index:              0,     1,   ... left - 1, left, ... n - 1,  n
Condition return: False, False, ...  False,   True,     True,  True
```

Vậy nếu bài toán yêu cầu tìm **giá trị nhỏ nhất** thoả mãn `condition` return `true` thì bạn return `left`, còn nếu yêu cầu tìm **giá trị lớn nhất** thoả mãn `condition` return `false` thì bạn return `left - 1`.

# Áp dụng vào problem đơn giản

Bắt đầu với bài toán tìm căn bậc 2 của số X: [Sqrt(x)](https://leetcode.com/problems/sqrtx/description)

> Cho một số nguyên không âm `x`, tìm căn bậc 2 của `x`, làm tròn xuống số nguyên không âm gần `x` nhất.
>
> Điều kiện: $0 <= x <= 2^{31} - 1$

Ví dụ 1:

```text
Input: 4
Output: 2
```

Ví dụ 2:

```text
Input: 8
Output: 2
```

Chúng ta dễ dàng nhận thấy rằng hàm `condition` có thể viết như sau:

```java
boolean condition(int num) {
    return x / num < num;
}
```

Và khi kết thúc vòng lặp `while`, giá trị cần tìm của chúng ta sẽ là `left - 1`.

Như vậy, đáp án của chúng ta như sau:

```java
public int mySqrt(int x) {
    if (x < 2) return x;
    int left = 0;
    int right = x;

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (x / mid < mid) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return left - 1;
}
```

# Áp dụng vào problem phức tạp

Chúng ta cùng tìm hiểu bài toán sau: [Capacity To Ship Packages Within D Days](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/description)

> Một băng chuyền gồm **các gói hàng** phải giao từ cổng này tới cổng khác trong vòng `D` ngày. Gói hàng thứ `i` có khối lượng là `weights[i]`. Mỗi ngày, chúng ta chất các gói hàng lên một **khay hàng** trên băng chuyền theo thứ tự lần lượt được cho bởi mảng `weights`. Chúng ta không thể chất quá tải trọng của khay hàng.
>
> Tìm **tải trọng nhỏ nhất** có thể của khay hàng để đảm bảo tất cả các gói hàng được giao trong vòng `D` ngày.
>
> Điều kiện:
>
> $1 <= D <= weights.length <= 5 * 10^4$
>
> $1 <= weights[i] <= 500$

Trước tiên, chúng ta cùng phân tích một chút. Với các bài toán đơn giản như ví dụ trước thì đọc xong đề là chúng ta biết ngay cần phải giải bằng Binary Search rồi. Nhưng đôi khi chúng ta gặp những bài toán phức tạp hơn mà mình còn **không biết** là cần phải áp dụng Binary Search để giải. Thế là chúng ta đi áp dụng các phương pháp khác như **DFS**, **BFS** hay **Quy hoạch động** để giải, nhưng càng làm càng thấy bí. Như trong bài toán này, ý nghĩ thoáng qua trong đầu mình lúc mới đọc xong đề bài là áp dụng **Quy hoạch động**.

Vậy dấu hiệu để nhận biết một bài toán có thể giải bằng **Binary Search** là khi:
> Chúng ta nhận thấy đáp án của bài toán có **tính đơn điệu**, nghĩa là khi `condition(k)` return `true` thì `condition(k + 1)` cũng sẽ return `true`.

Quay trở lại với bài toán trên, chúng ta thấy rằng nếu có thể giao toàn bộ các gói hàng trong `D` ngày với khay hàng có tải trọng `M`, thì khi chúng ta sử dụng khay hàng có tải trọng lớn hơn `M` (ví dụ `M + 1`) thì hoàn toàn vẫn có thể giao trong tối đa `D` ngày.

Với hàm `condition`, chúng ta sẽ sử dụng thuật toán tham lam, tham số là **tải trọng của khay hàng** được cho trước. Mỗi ngày chúng ta đặt lần lượt các gói hàng lên khay đến khi hết tải trọng, từ đó tính ra tổng số ngày cần thiết để giao toàn bộ hàng, sau đó chúng ta so sánh nó với `D`, nếu ít hơn hoặc bằng `D` ngày thì return `true`, vượt quá thì return `false`.

Với hai giá trị biên, **tải trọng tối thiểu** của khay hàng bằng với **khối lượng của gói hàng lớn nhất**, để đảm bảo chúng ta có thể giao toàn bộ các gói hàng; còn **tải trọng tối đa** sẽ bằng **tổng khối lượng của toàn bộ hàng**, khi đó chúng ta chỉ mất 1 ngày để giao hàng.

Và đáp án của chúng ta là:

```java
public int shipWithinDays(int[] weights, int days) {
    int left = 0;
    int right = 0;
    for (int w : weights) {
        left = Math.max(left, w);
        right += w;
    }
    while (left < maxCap) {
        int mid = left + (right - left) / 2;
        if (isValidTime(weights, days, mid)) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return left;
}

private boolean isValidTime(int[] weights, int days, int capacity) {
    int totalDay = 1;
    int curCap = 0;
    for (int w : weights) {
        curCap += w;
        if (curCap > capacity) {
            totalDay++;
            curCap = w;
        }
    }
    return totalDay <= days;
}
```

# Kết luận

Hy vọng qua bài viết này, các bạn có thêm tự tin chinh chiến trên đấu trường **LeetCode**. Kể cả khi gặp các bài khó, chỉ cần bạn nhận ra rằng bài này có thể áp dụng **Binary Search**, thì có thể copy paste template này vào, sau đó là để đôi bàn tay lướt nhẹ trên bàn phím. Mọi việc còn lại thật quá dễ dàng!
