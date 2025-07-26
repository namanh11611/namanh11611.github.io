---
title: "Applying LeetCode Algorithms to the Speakie English Learning App"
description: "There's a long-standing debate: Is grinding algorithms only for coding interviews and rarely applicable to real work? Let's explore how LeetCode algorithms can be applied to the Speakie English learning app!"
date: 2025-05-11T01:00:00+07:00
slug: speakie
image: speakie.webp
toc: true
categories: [Technical]
tags: [Algorithm, LeetCode, Speakie, Java]
---

There's a long-standing debate: *"Is grinding algorithms only for **preparing for coding interviews** and rarely **applicable to real work**?"* Of course, there are obvious examples like **ride-hailing apps** that must use **pathfinding algorithms**, a classic topic we've learned since university. But writing such important algorithms seems reserved for a **small group of senior developers**, while most of us do simpler tasks and wonder when we'll get to use the knowledge we've worked so hard to acquire. Are we **wasting a lot of time**?

So, today's article shares a problem I encountered in a project, and how practicing **LeetCode** helped me solve it. I'm not bragging—this is just a small algorithm, not something **world-changing**. I just want to show: *"Oh, so grinding algorithms can sometimes be applied to real projects like this"* 😜

# The Problem

I was building an English speaking app called [**Speakie**](https://speakie.xyz/), mainly to solve my own pain point: I couldn't speak full sentences in conversation, often pausing to think about grammar. So, the lessons in the app are based on **common conversational sentence patterns**.

> Quick ad break:
>
> Android: https://play.google.com/store/apps/details?id=com.areser.speakie
>
> iOS: https://apps.apple.com/app/speakie/id6593695505

The problem: When a user speaks a sentence, I need to compare it to the sample sentence, marking correct words in **green** and incorrect words in **red**. And I need to mark as many words green as possible to encourage the user.

> Example:
>
> **Sample sentence:** Hello World! Welcome to my app Speakie.
>
> **User's sentence:** Hello Henry! Welcome to my world.
>
> ==> In this case, the sample sentence would have "Hello", "Welcome", "to", "my" marked green.

A naive solution is to use a `Set` to store the words from the user's sentence, then check each word in the sample sentence—if the `Set` contains the word, mark it green; otherwise, mark it red. But this doesn't preserve word order. For example, if "world" appears out of order, this method would still mark it green.

While working, I noticed some popular English learning apps still make this mistake. You can test it by intentionally saying words out of order 😉

# LeetCode problem

When thinking of a solution, I immediately remembered the [Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence) problem:

> Given two strings `text1` and `text2`, return the length of their longest common subsequence. If there is no common subsequence, return 0.

# Solution

The solution is to use dynamic programming (DP). We build a 2D array where `dp[i][j]` represents the length of the longest common subsequence of `text1[0..i-1]` and `text2[0..j-1]`. The relationship is:

- If `text1[i-1] == text2[j-1]`, then `dp[i][j] = dp[i-1][j-1] + 1`.
- Otherwise, `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`.

The answer will be in `dp[text1.length()][text2.length()]`.

## Java code

Here's the Java code I implemented in the project:

```java
public class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        int m = text1.length(), n = text2.length();
        int[][] dp = new int[m + 1][n + 1];

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }

        return dp[m][n];
    }
}
```

## Time complexity

The time complexity is `O(m*n)` and the space complexity is `O(m*n)`.

# Applying to Speakie

In Speakie, we don't need the length of the longest common subsequence. We need to reconstruct the subsequence itself. So, after finding the DP array, we backtrack to find the sequence.

## Java code

Here's the Java code for Speakie:

```java
public class Solution {
    public String longestCommonSubsequence(String text1, String text2) {
        int m = text1.length(), n = text2.length();
        int[][] dp = new int[m + 1][n + 1];

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }

        // Backtrack to find the sequence
        StringBuilder sb = new StringBuilder();
        int i = m, j = n;
        while (i > 0 && j > 0) {
            if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                sb.append(text1.charAt(i - 1));
                i--;
                j--;
            } else if (dp[i - 1][j] > dp[i][j - 1]) {
                i--;
            } else {
                j--;
            }
        }

        return sb.reverse().toString();
    }
}
```

## Time complexity

The time complexity is `O(m*n)` and the space complexity is `O(m*n)`.

# Conclusion

Through this article, I hope to show one of the ways **LeetCode** can be applied to real projects, not just for interviews. If you have any questions, feel free to leave a comment. And don't forget to try out the Speakie app to improve your English speaking skills!
