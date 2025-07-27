---
title: "Design Pattern: Bill Pugh Singleton in Java – surprisingly simple"
description: "The Singleton pattern is probably the simplest design pattern that almost everyone knows. It helps create a single instance of a class."
date: 2020-12-29T15:00:00+07:00
slug: design-pattern-singleton
image: singleton.webp
categories: [Technical]
tags: [Design Pattern, Singleton, Java]
---

# Introduction

The Singleton pattern is probably the simplest design pattern that almost everyone knows. It helps create a single instance of a class, often used for classes like Database, Manager, etc. Today, while reading code in my current project, I discovered a really neat way to implement Singleton, called the Bill Pugh Singleton, named after its creator. So I wrote this article to share this approach to Singleton.

# Lazy Initialization

First, let's look at the Singleton initialization method most people use. The Singleton pattern is implemented by creating an instance in a public method. The downside is that when running in multiple threads, multiple instances can be created. In that case, Singleton is no longer a Singleton.

```java
public class LazyInitializedSingleton {

    private static LazyInitializedSingleton instance;
    
    private LazyInitializedSingleton() { }
    
    public static LazyInitializedSingleton getInstance() {
        if (instance == null) {
            instance = new LazyInitializedSingleton();
        }
        return instance;
    }
}
```

# Thread Safe Singleton

To fix the drawback of Lazy Initialization, we add `synchronized` to the public method. This way, only one instance is created by one thread at a time.

```java
public class ThreadSafeSingleton {

    private static ThreadSafeSingleton instance;
    
    private ThreadSafeSingleton() { }
    
    public static synchronized ThreadSafeSingleton getInstance() {
        if (instance == null) {
            instance = new ThreadSafeSingleton();
        }
        return instance;
    }
    
}
```

However, this approach still has the drawback of reducing app performance because `getInstance()` is a `synchronized method`. So, we have another improved approach:

```java
public class ThreadSafeSingleton {

    private static ThreadSafeSingleton instance;
    
    private ThreadSafeSingleton() { }
    
    public static ThreadSafeSingleton getInstance() {
        if (instance == null) {
            synchronized (ThreadSafeSingleton.class) {
                if (instance == null) {
                    instance = new ThreadSafeSingleton();
                }
            }
        }
        return instance;
    }
    
}
```

This way, we only pay the cost on the first call to `getInstance()`.

# Bill Pugh Singleton Implementation

Before Java 5, Java memory had many issues and the above methods could fail when too many threads called the Singleton class's `getInstance()` method simultaneously. So, Bill Pugh introduced a new way to implement Singleton using an inner static helper class.

```java
public class BillPughSingleton {

    private static class SingletonHelper {
        static final BillPughSingleton INSTANCE = new BillPughSingleton();
    }
    
    private BillPughSingleton() { }
    
    public static BillPughSingleton getInstance() {
        return SingletonHelper.INSTANCE;
    }
    
}
```

What do you think of this method? It's fast, concise, and still safe. When the Singleton class is loaded, the SingletonHelper class is not loaded into memory yet. Only when the `getInstance()` method is called is the helper class loaded and the singleton instance created. This method also doesn't require synchronization or multiple null checks.

# Conclusion

Within the scope of this article, there are a few more methods I haven't listed, but I hope this gives you a new perspective on the Singleton pattern. Thank you for reading!

**Reference**

https://www.journaldev.com/1377/java-singleton-design-pattern-best-practices-examples
