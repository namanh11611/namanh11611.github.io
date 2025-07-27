---
title: "3 Ways I Apply to Write Cleaner and Tidier Code"
description: "Clean Code is probably a classic topic in programming. In this article, I want to share a few tips that I often use to keep my code tidy and clean."
date: 2024-05-01T00:00:00+07:00
slug: clean-code
image: tidy.webp
toc: true
categories: [Technical]
tags: [Clean Code]
---

**Clean Code** is probably a classic topic in programming. I once mentioned the book **Clean Code** by **Uncle Bob** in the article [**Things I Wish I Knew When I Was a Junior**](../junior). However, this time I won't rewrite the contents of that book—you should read it directly for the best experience. I just want to share a few tips that I often use to keep my code **tidy and clean**.

# Use return to avoid if/else hell

Have you ever encountered **if/else hell** like this in your code?

```java
void doSomething() {
    if (condition1) {
        doFirstTask();
        if (condition2 != null) {
            doSecondTask();
            if (condition3.isNotEmpty()) {
                doThirdTask();
            }
        }
    }
}
```

In theory, the above code is fine. But I don't like looking at code with so many nested layers. So I often use `return` to avoid writing too many nested braces `{}`. We can refactor the code above as follows:

```java
void doSomething() {
    if (!condition1) return
    doFirstTask();
    if (condition2 == null) return
    doSecondTask();
    if (condition3.isEmpty()) return
    doThirdTask();
}
```

The logic is equivalent, but the meaning is slightly different. In the first case, when your colleague reads the code, they'll understand: *"If this condition is met, the code will continue to process like this."* In the second case, they'll understand: *"If this condition is not met, the code will not continue."* Thinking in the second way can sometimes feel a bit reversed, so **apply this tip as appropriate**, don't use it mechanically in all cases.

# Use Map or Set instead of List

**Map** and **Set** are two special data structures that you probably learned in university. However, in projects, people often always use **List** because it's "convenient." For me, before choosing a data type for a list, I always ask whether using Map or Set would make the **code shorter** and **improve performance**. If yes, I definitely **prioritize these two types**.

For example, I have a variable `existingValueList` which is a list of existing values. When a user enters a new value, I need to check if it already exists, and if so, return an error. If `existingValueList` is a `List`, I'd write the check like this:

```java
boolean isExisting = existingValueList.contains(value);
```

But if it's a Set, we'd write:

```java
boolean isExisting = existingValueSet.contains(value);
```

*"So, what's the difference?"* The way you write it is the same, but the difference lies in the `contains()` method: the runtime for `List` is `O(n)`, while for `Set` it's only `O(1)`.

99% of you probably know this, but sometimes you use **List** out of habit. So next time, before creating a **List**, pause for a moment to consider whether you should use **Map** or **Set**.

There are also other data structures like **Stack**, **Queue**, **Tree**... But in real projects, I rarely encounter use-cases for these.

# Don't comment out unused code—just delete it

I've noticed some people have the habit of **commenting out old code** when modifying a feature instead of deleting it. Or there are **old functions that are no longer used**, but they don't bother to comment or delete them, just leaving them there forever. But trust me, **a few months** or even **a few years** later, you'll probably never uncomment that code again. A few lines each time, but over the years it becomes a mountain of **dead code**.

If you want to review old code, [**Git**](../git-process) can help you very well. So be bold and delete it—not just **a few lines of code**, **a function**, but also any **resources** of the app that you no longer use. This will help reduce your source code size significantly.

If you have any cool tips that you use, please share them with me!
