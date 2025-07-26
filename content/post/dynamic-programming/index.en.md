---
title: "LeetCode: Dễ dàng nhận biết 5 dạng bài Dynamic Programming"
description: "Có thể nói, trong quá trình làm LeetCode thì Dynamic Programming là một dạng bài mọi người thường xuyên gặp nhất, nhưng cũng là một trong những dạng khó nhằn nhất."
date: 2024-05-05T00:00:00+07:00
slug: dynamic-programming
image: dynamic_programming.webp
toc: true
categories: [Technical]
tags: [Dynamic Programming, Algorithm, LeetCode, Java]
---

Có thể nói, trong quá trình làm LeetCode thì **Dynamic Programming (DP)**, hay còn gọi là **Quy hoạch động** trong tiếng Việt, là một dạng bài mọi người thường xuyên gặp nhất, nhưng cũng là một trong những dạng khó nhằn nhất. Khi gặp một bài mới, nếu bạn nhận ra nó có thể giải bằng DP là bạn đã đi được **80% quãng đường** rồi. Vậy nên, để giúp cho quá trình *"nhận ra"* đấy trở nên dễ dàng hơn, trong bài viết này, mình sẽ chia sẻ với các bạn 5 dạng phổ biến của DP trên LeetCode.

# Tìm giá trị nhỏ nhất / lớn nhất

Đề bài thường có dạng:

> Cho một mục tiêu, tìm giá trị nhỏ nhất / lớn nhất để đạt mục tiêu đó.

Hướng giải:

> Tìm giá trị nhỏ nhất / lớn nhất trong số tất cả các cách có thể trước đó, sau đó cộng với giá trị của trạng thái hiện tại.

```java
routes[i] = min(routes[i-1], ... , routes[i-k]) + cost[i]
```

Khi giải một bài DP, chúng ta có 2 hướng làm là **Top-Down** và **Bottom-Up**.

Theo hướng tiếp cận **Top-Down**, chúng ta sẽ bắt đầu bằng **bài toán lớn nhất** hay là **bài toán ở mức trên cùng** sau đó dùng **phương pháp đệ quy** để gọi lời giải cho các **bài toán con** ở mức thấp hơn tiếp theo. Quá trình tiếp tục cho đến khi gặp bài toán nhỏ nhất. Đệ quy sẽ tự động tổ hợp kết quả của các bài toán con để được kết quả bài toán ban đầu. Chúng ta thường sẽ dùng một mảng `memo` để lưu kết quả các bài toán con, tránh việc phải tính đi tính lại chúng gây ra lỗi `Time Limit Exceeded`.

```java
for (int j = 0; j < ways.size(); ++j) {
    result = min(result, topDown(target - ways[j]) + cost[i]);
}
return memo[/*state parameters*/] = result;
```

Còn theo hướng **Bottom-Up**, chúng ta sẽ giải các **bài toán con** ở mức thấp nhất, sau đó dùng các kết quả này để tính **bài toán ở mức trên**. Quá trình tiếp tục cho đến khi chúng ta tìm được kết quả bài toán mức cao nhất. Cá nhân mình thì thường làm theo hướng nay hơn.

```java
for (int i = 1; i <= target; ++i) {
   for (int j = 0; j < ways.size(); ++j) {
       if (ways[j] <= i) {
           dp[i] = min(dp[i], dp[i - ways[j]] + cost[i]);
       }
   }
}
return dp[target]
```

Chúng ta cùng áp dụng vào bài [Min Cost Climbing Stairs](https://leetcode.com/problems/min-cost-climbing-stairs)

> Cho mảng `cost`, trong đó `cost[i]` là chi phí của bước thứ `i` trên cầu thang. Khi bạn trả phí, bạn có thể bước 1 hoặc 2 bước. Bạn được chọn bước từ vị trí 0 hoặc 1.
> 
> Tìm chi phí nhỏ nhất để đi hết cầu thang.

Bạn có thể nhận thấy rằng, khi xét bước thứ `i`, bạn chỉ có thể bước đến đây từ bước thứ `i - 1` (bước 1 bước) hoặc từ bước thứ `i - 2` (bước 2 bước). Vậy nên, chi phí nhỏ nhất khi bước đến bước thứ `i` sẽ bằng tổng của giá trị nhỏ hơn trong 2 bước trước đó cộng với `cost[i]`. Từ đó chúng ta có 2 cách giải tương ứng như sau.

Top-Down

```java
int result = min(minCost(n-1, cost, memo), minCost(n-2, cost, memo))
             + (n == cost.size() ? 0 : cost[n]);
return memo[n] = result;
```

Bottom-Up

```java
for (int i = 2; i <= n; ++i) {
   dp[i] = min(dp[i-1], dp[i-2]) + (i == n ? 0 : cost[i]);
}
return dp[n]
```

# Tìm số cách khác nhau

Đề bài thường có dạng:

> Cho một mục tiêu, tìm số cách khác nhau để đạt mục tiêu đó.

Hướng giải:

> Tính tổng tất cả các cách có thể để đạt đến mục tiêu.

```java
routes[i] = routes[i-1] + routes[i-2] + ... + routes[i-k]
```

**Top-Down**

```java
for (int j = 0; j < ways.size(); ++j) {
    result += topDown(target - ways[j]);
}
return memo[/*state parameters*/] = result;
```

**Bottom-Up**

```java
for (int i = 1; i <= target; ++i) {
   for (int j = 0; j < ways.size(); ++j) {
       if (ways[j] <= i) {
           dp[i] += dp[i - ways[j]];
       }
   }
}
return dp[target]
```

Chúng ta cùng áp dụng vào bài [Climbing Stairs](https://leetcode.com/problems/climbing-stairs)

> Bạn đang leo lên một chiếc cầu thang. Nó cần `n` bước để leo đến đỉnh. Mỗi lần bạn chỉ có thể leo 1 hoặc 2 bước.
>
> Hãy tính bạn có bao nhiêu cách khác nhau để leo đến đỉnh?

Tương tự như bài trên, khi xét bước thứ `i`, bạn chỉ có thể bước đến đây từ bước thứ `i - 1` (bước 1 bước) hoặc từ bước thứ `i - 2` (bước 2 bước). Vậy nên, số cách khác nhau leo đến bước thứ `i` sẽ bằng tổng của số cách đến bước `i - 1` và số cách đến bước `i - 2`. Từ đó chúng ta có 2 cách giải tương ứng như sau.

Top-Down

```java
int result = climbStairs(n-1, memo) + climbStairs(n-2, memo);
return memo[n] = result;
```

Bottom-Up

```java
for (int stair = 2; stair <= n; ++stair) {
    dp[stair] = dp[stair-1] + dp[stair-2];
}
```

# Hợp nhất các khoảng giá trị

Đề bài thường có dạng:

> Cho một bộ số, tìm giải pháp tối ưu cho một bài toán mà xét số hiện tại và kết quả tốt nhất bạn có thể nhận được từ bên trái và bên phải.

Hướng giải:

> Tìm tất cả giải pháp tối ưu cho mỗi khoảng và trả về kết quả tốt nhất có thể.

```java
// From i to j
dp[i][j] = dp[i][k] + result[k] + dp[k+1][j]
```

**Top-Down**

```java
for (int k = i; k <= j; ++k) {
    result = max(result, topDown(nums, i, k) + result[k] + topDown(nums, k+1, j));
}
return memo[/*state parameters*/] = result;
```

**Bottom-Up**

```java
for(int l = 1; l<n; l++) {
   for(int i = 0; i<n-l; i++) {
       int j = i+l;
       for(int k = i; k<j; k++) {
           dp[i][j] = max(dp[i][j], dp[i][k] + result[k] + dp[k+1][j]);
       }
   }
}
return dp[0][n-1];
```

Chúng ta cùng áp dụng vào bài [Minimum Cost Tree From Leaf Values](https://leetcode.com/problems/minimum-cost-tree-from-leaf-values)

> Cho một mảng `arr` các số dương. Chọn tất cả cây nhị phân thoả mãn điều kiện:
> * Mỗi node có 0 hoặc 2 node con
> * Giá trị các phần tử của mảng `arr` tương ứng với giá trị của mỗi **lá** (node không có con) khi duyệt cây theo **in-order**
> * Giá trị của mỗi **node không phải lá** thì bằng tích của **lá lớn nhất** trong **cây con trái** và **phải** của nó
> 
> Trong tất cả các cây được chọn, tìm cây có tổng các **node không phải lá** nhỏ nhất.

```java
for (int l = 1; l < n; ++l) {
   for (int i = 0; i < n - l; ++i) {
       int j = i + l;
       dp[i][j] = INT_MAX;
       for (int k = i; k < j; ++k) {
           dp[i][j] = min(dp[i][j], dp[i][k] + dp[k+1][j] + maxs[i][k] * maxs[k+1][j]);
       }
   }
}
```

# String

Đề bài cho `String` thường có rất nhiều dạng, nhưng hầu hết đều là cho 1 hoặc 2 string với độ dài không quá lớn.

> Cho 2 string `s1` và `s2`, trả về kết quả gì đấy.

Hướng giải:

> Hầu hết các bài toán dạng này có thể giải với kết quả có độ phức tạp về thời gian là `O(n)`.

```java
// i - indexing string s1
// j - indexing string s2
for (int i = 1; i <= m; ++i) {
   for (int j = 1; j <= n; ++j) {
       if (s1[i-1] == s2[j-1]) {
           dp[i][j] = /*code*/;
       } else {
           dp[i][j] = /*code*/;
       }
   }
}
```

Nếu đề bài chỉ cho 1 string thì cách giải sẽ hơi khác một chút:

```java
for (int l = 1; l < n; ++l) {
   for (int i = 0; i < n-l; ++i) {
       int j = i + l;
       if (s[i] == s[j]) {
           dp[i][j] = /*code*/;
       } else {
           dp[i][j] = /*code*/;
       }
   }
}
```

Chúng ta cùng áp dụng vào bài [Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence)

> Cho 2 string `text1` và `text2`, tìm chiều dài lớn nhất của subsequence chung.

Xét ký tự thứ `i` của `text1` và ký tự thứ `j` của `text2`.
* Nếu chúng giống nhau, chúng ta tăng chiều dài subsequence chung thêm 1
* Nếu chúng khác nhau, chúng ta sẽ lấy chiều dài lớn hơn trong 2 trường hợp `i - 1` với `j` hoặc `i` với `j - 1`

```java
for (int i = 1; i <= m; ++i) {
   for (int j = 1; j <= n; ++j) {
       if (text1[i-1] == text2[j-1]) {
           dp[i][j] = dp[i-1][j-1] + 1;
       } else {
           dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
       }
   }
}
```

Với bài toán 1 string, chúng ta sẽ áp dụng vào bài [Palindromic Substrings](https://leetcode.com/problems/palindromic-substrings)

> Cho 1 string `s`, tính số lượng palindromic substring của nó.

Chúng ta xét ký tự thứ `i` và `j`, nếu chúng giống nhau và substring từ `i + 1` đến `j - 1` đã là palindromic thì chúng ta tìm được một palindromic mới.

```java
for (int l = 1; l < n; ++l) {
   for (int i = 0; i < n-l; ++i) {
       int j = i + l;
       if (s[i] == s[j] && dp[i+1][j-1] == j-i-1) {
           dp[i][j] = dp[i+1][j-1] + 2;
       } else {
           dp[i][j] = 0;
       }
   }
}
```

# Ra quyết định

Dạng bài thì nổi tiếng với loạt bài [House Robber](https://leetcode.com/problems/house-robber) và [Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock).

Đề bài thường có dạng:

> Cho một tập giá trị, tìm đáp án với tuỳ chọn là chọn hoặc bỏ qua giá trị hiện tại.

Hướng giải:

> Nếu bạn quyết định chọn giá trị hiện tại, hãy sử dụng kết quả trước đó, trong đó giá trị trước đó bị bỏ qua. Ngược lại, nếu bạn quyết định bỏ qua giá trị hiện tại, hãy sử dụng giá trị trước đó, trong đó giá trị trước đó được sử dụng.

```java
// i - indexing a set of values
// j - options to ignore j values
for (int i = 1; i < n; ++i) {
    for (int j = 1; j <= k; ++j) {
        dp[i][j] = max({dp[i][j], dp[i-1][j] + arr[i], dp[i-1][j-1]});
        dp[i][j-1] = max({dp[i][j-1], dp[i-1][j-1] + arr[i], arr[i]});
    }
}
```

Chúng ta cùng áp dụng vào bài [House Robber](https://leetcode.com/problems/house-robber)

> Bạn là một tên trộm chuyên nghiệp đang có kế hoạch ăn trộm cả một dãy phố. Tuy nhiên dãy phố này có hệ thống an ninh liên kết với nhau, và nó sẽ tự động báo cảnh sát nếu 2 ngôi nhà cạnh nhau cùng bị trộm.
>
> Cho một mảng `nums` đại diện cho số tiền trong mỗi ngôi nhà. Hãy tính số tiền tối đa bạn có thể trộm được mà không bị báo cảnh sát.

Chúng ta tạo 1 mảng 2 chiều với công thức:
* `dp[i][1]` là tổng số tiền nếu quyết định trộm nhà thứ `i`
* `dp[i][0]` là tổng số tiền nếu quyết định **không** trộm nhà thứ `i`
```java
for (int i = 1; i < n; ++i) {
    dp[i][1] = max(dp[i-1][0] + nums[i], dp[i-1][1]);
    dp[i][0] = dp[i-1][1];
}
```

# Reference

* https://leetcode.com/discuss/general-discussion/458695/Dynamic-Programming-Patterns
