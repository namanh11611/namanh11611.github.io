---
title: "Chuyện áp dụng thuật toán LeetCode vào ứng dụng học tiếng Anh Speakie"
description: "Có một vấn đề anh em thường tranh cãi lâu nay là: Có phải cày thuật toán chỉ để chuẩn bị cho vòng coding interview chứ chẳng áp dụng được mấy trong công việc? Vậy thì hãy cùng mình tìm hiểu xem thuật toán LeetCode có thể áp dụng vào ứng dụng học tiếng Anh Speakie như thế nào nhé!"
date: 2025-05-11T01:00:00+07:00
slug: speakie
image: speakie.webp
toc: true
categories: [Technical]
tags: [Algorithm, LeetCode, Speakie, Java]
---

Có một vấn đề anh em thường tranh cãi lâu nay là: *"Có phải **cày thuật toán** chỉ để **chuẩn bị cho vòng coding interview** chứ chẳng **áp dụng được mấy trong công việc**?"*. Tất nhiên vẫn có những ví dụ ngay trước mắt chúng ta như các **ứng dụng gọi xe** chẳng hạn, chắc chắn họ phải áp dụng **thuật toán tìm đường**, một thuật toán kinh điển mà anh em đã được học từ hồi sinh viên. Vậy nhưng công việc viết các thuật toán quan trọng ấy dường như chỉ dành cho **số ít các bác developer ở level cao**, còn đa số anh em vẫn làm những việc đơn giản hơn, không biết đến bao giờ mới có cơ hội áp dụng kiến thức mà mình đã vất vả cày cuốc để tích lũy. Có phải chúng ta đang **phí phạm một lượng lớn thời gian** không?

Vậy thì bài viết hôm nay sẽ chia sẻ về vấn đề mà mình gặp phải trong dự án, và cách mà quá trình luyện **LeetCode** đã giúp mình giải quyết nó như thế nào. Mình không có ý khoe khoang gì, vì đây chỉ là một thuật toán nhỏ, không cao siêu đến mức **thay đổi thế giới**. Chỉ là kể lại để anh em thấy: *"Oh, hóa ra cày thuật toán cũng có những lúc áp dụng vào dự án thực tế như vậy"* 😜

# Bài toán đặt ra

Chuyện là, mình đang xây dựng một ứng dụng học tiếng Anh giao tiếp tên [**Speakie**](https://speakie.xyz/), chủ yếu để giải quyết pain point của mình, đó là không bật ra được cả câu khi giao tiếp, mà mình thường bị khựng lại để nghĩ ngữ pháp xem đúng chưa. Vậy nên các bài học trong app được mình xây dựng dựa trên các **mẫu câu giao tiếp phổ biến** trong đời sống.

> Mình xin 5 giây dành cho quảng cáo:
>
> Android: https://play.google.com/store/apps/details?id=com.areser.speakie
>
> iOS: https://apps.apple.com/app/speakie/id6593695505

Vậy bài toán đặt ra là khi user nói một câu, mình phải so sánh với các từ trong câu mẫu, từ nào đúng thì **tô màu xanh**, từ nào sai thì **tô màu đỏ**. Và phải đảm bảo tô xanh được càng nhiều từ càng tốt để còn khuyến khích user.

> Ví dụ:
>
> **Câu mẫu:** Hello World! Welcome to my app Speakie.
>
> **Câu user nói:** Hello Henry! Welcome to my world.
>
> ==> Lúc này, câu mẫu sẽ được tô xanh các từ "Hello", "Welcome", "to", "my".

Một phương án ngây thơ để giải quyết là dùng `Set` để lưu các từ trong câu user nói, sau đó check từng từ trong câu mẫu, `Set` chứa từ nào thì tô xanh, không có thì tô đỏ. Vậy nhưng cách này sẽ không đảm bảo thứ tự xuất hiện của các từ. Ví dụ như từ "world" xuất hiện không đúng thứ tự nhưng nếu áp dụng cách này thì nó vẫn sẽ được tô xanh toàn bộ.

Trong quá trình làm, mình thấy một số app học tiếng Anh khá nổi tiếng còn đang mắc lỗi sai như trên, các bạn có thể test bằng cách cố tình nói sai thứ tự sẽ phát hiện ra 😉

# LeetCode problem

Khi nghĩ giải pháp, mình nhớ ngay đến bài [Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence). Đề bài như sau:

> Cho 2 string `text1` và `text2`, return độ dài subsequence chung dài nhất của chúng. Nếu không có subsequence chung, return 0.
>
> Subsequence của string là một string mới được tạo từ string gốc bằng cách xóa một vài ký tự (hoặc không xóa ký tự nào) mà không làm thay đổi thứ tự của các ký tự còn lại.
>
> Ví dụ, "ace" là một subsequence của "abcde".

## Base solution

Bài này cần áp dụng **Quy hoạch động** (Dynamic Programming) để giải. Ví dụ với `text1` = "**XMJYAUZ**", `text2` = "**MZJAWXU**", chúng ta cần tạo 1 bảng 2 chiều để lưu độ dài subsequence chung dài nhất của 2 string khi duyệt từ đầu đến cuối như sau.

![Solution 2D Array](https://images.viblo.asia/aca21a7a-f510-4cd5-b3f6-22b33c64b7b8.png)

Bạn có thể hiểu rằng:

* Hàng 0 là khi `text1` empty, `text2` = "MZJAWXU", không có subsequence => return **0**
* Hàng 1 là khi `text1` = "X", `text2` = "MZJAWXU", subsequence = "**X**" => return **1**
* Hàng 2 là khi `text1` = "XM", `text2` = "MZJAWXU", subsequence = "**X**" hoặc "**M**" => return **1**
* Hàng 3 là khi `text1` = "XMJ", `text2` = "MZJAWXU", subsequence = "**MJ**" => return **2**
* ...

Tổng quát, khi xét ký tự thứ `i` của `text1` và ký tự thứ `j` của `text2`:

* Nếu `text1[i]` == `text2[j]` thì `length[i][j]` = `length[i - 1][j - 1]` + 1
* Nếu `text1[i]` != `text2[j]` thì `length[i][j]` = max(`length[i - 1][j]`,`length[i][j - 1]`)

Toàn bộ source code của chúng ta như sau:

```java
public int longestCommonSubsequence(String text1, String text2) {
    int n1 = text1.length(), n2 = text2.length();
    int[][] length = new int[n1 + 1][n2 + 1];
    for (int i = 1; i <= n1; ++i) {
        for (int j = 1; j <= n2; ++j) {
            if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                length[i][j] = length[i - 1][j - 1] + 1;
            } else {
                length[i][j] = Math.max(length[i - 1][j], length[i][j - 1]);
            }
        }
    }
    return length[n1][n2];
}
```

## Space Optimization solution

Cách trên là dễ hiểu nhất và bám sát với phần phân tích của chúng ta. Tuy nhiên, vẫn còn một cách tối ưu hơn về bộ nhớ, bạn có thể đọc thêm ở phần [Solutions](https://leetcode.com/problems/longest-common-subsequence/solutions/351689/java-python-3-two-dp-codes-of-o-mn-o-min-m-n-spaces-w-picture-and-analysis/).

Khi duyệt row `i`, chúng ta chỉ cần các giá trị từ row `i - 1` cùng với giá trị của ô `length[i][j - 1]`. Vì vậy, thay vì dùng bảng 2 chiều, chúng ta chỉ cần 1 array với thêm 2 biến `prevRowCol`, `prevRow` như sau:

![Solution 1D Array](https://images.viblo.asia/2faf9d89-c5c2-4b52-a25c-4a7d02a4a93b.png)

```java
public int longestCommonSubsequence(String text1, String text2) {
    int n1 = text1.length(), n2 = text2.length();
    if (n1 < n2) {
        return longestCommonSubsequence(text2, text1);
    }
    int[] length = new int[n2 + 1];
    for (int i = 1; i <= n1; ++i) {
        for (int j = 1, prevRow = 0, prevRowCol = 0; j <= n2; ++j) {
            prevRowCol = prevRow;
            prevRow = length[j];
            if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                length[j] = prevRowCol + 1;
            } else {
                length[j] = Math.max(length[j - 1], prevRow);
            }
        }
    }
    return length[n2];
}
```

# Áp dụng vào bài toán

Bạn thấy sự liên quan của nó với bài toán ban đầu chứ? Chúng ta chỉ cần **thay các ký tự bằng các từ** là sẽ tìm được những **từ chung** trong câu mẫu và câu mà user nói. Tuy nhiên có một vấn đề là LeetCode problem trên chỉ đưa ra **độ dài của subsequence**, trong khi chúng ta muốn tìm cụ thể xem **subsequence đó gồm những từ nào** để còn tô màu.

## Boolean array

Ý nghĩ ban đầu lóe ra trong đầu mình là dùng `Boolean array` có độ dài bằng câu mẫu để lưu, từ nào xuất hiện thì set là `true`, từ nào không thì set là `false`. Tuy nhiên vấn đề phát sinh là tại mỗi thời điểm cân nhắc `length[i][j]` = max(`length[i - 1][j]`,`length[i][j - 1]`), chúng ta cần có **`Boolean array` của cả 2 ô `[i - 1][j]` và `[i][j - 1]`** để set **`Boolean array` cho ô `[i][j]`** theo ô có `length` lớn hơn. Điều này dẫn đến chúng ta cần thêm một array tương tự như array `length` ở trên, nhưng thay vì mỗi phần tử là một số `int` thì sẽ là một `Boolean array`. Như vậy nó sẽ là một `Boolean array 2 chiều`.

```java
// Để cho đơn giản, sample (câu mẫu) và input (câu user nói) đã được chuyển
// từ câu thành list các từ, chữ hoa thành chữ thường, bỏ dấu...
public void displaySampleSentence(String[] sample, String[] input) {
    int sampleLength = sample.length, inputLength = input.length;
    int[] length = new int[inputLength + 1];
    boolean[][] match = new boolean[inputLength + 1][sampleLength + 1];
    boolean[] prevRowMatch = new boolean[sampleLength + 1];
    boolean[] prevRowColMatch = new boolean[sampleLength + 1];
    
    for (int i = 1; i <= sampleLength; ++i) {
        for (int j = 1, prevRow = 0, prevRowCol = 0; j <= inputLength; ++j) {
            prevRowCol = prevRow;
            prevRow = length[j];
            prevRowColMatch = prevRowMatch;
            prevRowMatch = match[j];
            if (sample[i - 1].equals(input[j - 1])) {
                length[j] = prevRowCol + 1;
                match[j] = prevRowColMatch;
                match[j][i] = true;
            } else if (length[j - 1] >= prevRow) {
                length[j] = length[j - 1];
                match[j] = match[j - 1];
            }
        }
    }
    for (int i = 1; i <= sampleLength; ++i) {
        if (match[inputLength][i]) {
            // Display sample[i - 1] as green word
        } else {
            // Display sample[i - 1] as red word
        }
    }
}
```

## Integer và Bit manipulation

Để đơn giản và gọn nhẹ hơn, mình nghĩ đến việc dùng một số `int` thay cho `Boolean array`, và dùng các phép toán với **bit**, set bit = 1 thay cho biến `true`, set bit = 0 thay cho biến `false`. Với Android và iOS, `int` trong Dart dùng biến 64-bit. Nếu mỗi bit tương ứng với một từ, chúng ta có thể lưu được câu gồm tối đa 64 từ, vậy là quá đủ cho app của mình.

Nhắc lại một chút về các phép toán với bit. Để set 1 bit ở vị trí thứ `i` thành `true`, chúng ta có thể dùng phép toán `OR` và `SHIFT LEFT` như sau:

```java
match |= 1 << i;

// Ví dụ match = 1, i = 3
match  = 0001
1 << i = 1000
match | (1 << i) = 1001 // => match = 9
// Như vậy chúng ta đã set bit thứ 3 từ phải sang trái (0th index)
// thành giá trị 1 (TRUE)
```

Để get value của bit thứ `i`, chúng ta dùng phép toán `SHIFT RIGHT` và `AND` như sau:

```java
int value = (match >> i) & 1;

// Ví dụ với match = 25, i = 3
match      = 0001 1001
match >> i = 0000 0011
(match >> i) & 1 = 1
// Như vậy value của bit thứ 3 từ phải sang trái (0th index) là 1 (TRUE)
```

Áp dụng vào cách giải bên trên, chúng ta có:

```java
public void displaySampleSentence(String[] sample, String[] input) {
    int sampleLength = sample.length, inputLength = input.length;
    int[] length = new int[inputLength + 1];
    int[] match = new int[inputLength + 1];
    int prevRowMatch = 0, prevRowColMatch = 0;
    
    for (int i = 1; i <= sampleLength; ++i) {
        for (int j = 1, prevRow = 0, prevRowCol = 0; j <= inputLength; ++j) {
            prevRowCol = prevRow;
            prevRow = length[j];
            prevRowColMatch = prevRowMatch;
            prevRowMatch = match[j];
            if (sample[i - 1].equals(input[j - 1])) {
                length[j] = prevRowCol + 1;
                match[j] = prevRowColMatch;
                match[j][i] = true;
            } else if (length[j - 1] >= prevRow) {
                length[j] = length[j - 1];
                match[j] = match[j - 1];
            }
        }
    }
    for (int i = 1; i <= sampleLength; ++i) {
        if (match[inputLength][i]) {
            // Display sample[i - 1] as green word
        } else {
            // Display sample[i - 1] as red word
        }
    }
}
```

# Lời kết

**Thuật toán** vẫn là một yếu tố nền tảng trong ngành lập trình này. Qua thời gian, công nghệ này có thể lạc hậu, công nghệ kia có thể lên ngôi, nhưng thuật toán thì vẫn còn đó, trơ gan cùng tuế nguyệt.

Qua ví dụ thú vị này, hy vọng anh em thấy hứng thú hơn với việc học thuật toán và áp dụng trong các bài toán thực tế.

# Reference

* https://leetcode.com/problems/longest-common-subsequence
* https://wikipedia.org/wiki/Longest_common_subsequence
* https://dart.dev/guides/language/numbers
