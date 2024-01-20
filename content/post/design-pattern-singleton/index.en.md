---
title: "Design Pattern: Bill Pugh Singleton trong Java - đơn giản đến không ngờ"
description: "Singleton pattern có lẽ là design pattern đơn giản nhất mà hầu như ai cũng biết. Nó giúp tạo một instance duy nhất của class."
date: 2020-12-29T15:00:00+07:00
slug: design-pattern-singleton
image: singleton.jpeg
categories: Technical
tags: [Design Pattern, Singleton, Java]
---

# Lời mở đầu
Singleton pattern có lẽ là design pattern đơn giản nhất mà hầu như ai cũng biết. Nó giúp tạo một instance duy nhất của class, thường dùng để tạo các class cho Database, Manager... Hôm nay mình đọc code của project đang làm thì phát hiện ra một cách khởi tạo Singleton hay quá, tìm hiểu thì nó gọi là Bill Pugh Singleton, lấy theo tên của tác giả nghĩ ra cách này. Vậy nên mình viết bài này chia sẻ cho mọi người một cách tiếp cận với Singleton.

# Lazy Initialization
Đầu tiên, chúng ta nhìn qua cách khởi tạo Singleton mà đa số mọi người đang dùng. Singleton pattern được implement bằng cách tạo một instance trong một public method. Cách này có nhược điểm là khi chạy ở multiple threads thì có thể bị khởi tạo instance nhiều lần. Lúc đó thì Singleton không còn là Singleton nữa rồi.

```
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
Để khắc phục nhược điểm của Lazy Initialization, chúng ta thêm `synchronized` vào public method. Khi đó, chỉ có một instance được khởi tạo bởi một thread tại một thời điểm.

```
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

Tuy nhiên, cách trên vẫn có nhược điểm là làm giảm hiệu năng của app khi mỗi lần gọi vì `getInstance()` là một `synchronized method`. Vậy nên chúng ta có 1 cách khác bổ sung như sau.

```
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

Như vậy, chúng ta chỉ tốn sức trong lần gọi `getInstance()` đầu tiên.

# Bill Pugh Singleton Implementation
Trước Java 5, java memory có rất nhiều issue và các cách trên đều fail khi có quá nhiều thread gọi method `getInstance()` của Singleton class đồng thời. Vì vậy, Bill Pugh đưa ra một cách triển khai Singleton mới bằng cách sử dụng inner static helper class.

```
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

Mọi người thấy cách này thế nào? Quá nhanh, quá gọn mà vẫn an toàn. Khi Singleton class được load, SingletonHelper class sẽ vẫn chưa được load vào memory. Chỉ khi method `getInstance()` được gọi, helper class mới được load và tạo singleton class instance. Cách này cũng không yêu cầu synchronization và check null nhiều lần.

# Lời kết
Trong khuôn khổ bài viết, còn một số phương pháp nữa mà mình không liệt kê hết được, chỉ mang đến góc nhìn mới về Singleton pattern cho mọi người. Cảm ơn mọi người đã đọc!

Tham khảo: https://www.journaldev.com/1377/java-singleton-design-pattern-best-practices-examples
