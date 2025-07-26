---
title: "Design Pattern: Adapter - làm thế nào để sạc iPhone bằng cáp type C"
description: "Bạn sẽ làm gì khi bạn muốn sạc iPhone nhưng trong tay chỉ có sợi dây cáp type C."
date: 2022-10-04T22:50:00+07:00
slug: design-pattern-adapter
image: adapter.webp
categories: [Technical]
tags: [Design Pattern, Adapter, Java]
---

# Đặt vấn đề
Khi bạn muốn sạc iPhone bằng cáp type C => hãy dùng một cái Adapter mà có lỗ type C và đầu lightning.

Khi bạn đang dùng data dưới dạng XML, nhưng lại muốn sử dụng một function của third party library mà nó chỉ nhận param là JSON => hãy dùng Adapter để convert XML sang JSON.

Trong Android, bạn có một list item và muốn hiển thị nó lên `RecyclerView` => hãy dùng Adapter để convert data lên từng `item_view`.
# Khái niệm
Adapter pattern cho phép **interface** của một **class đã có sẵn** được dùng như là một **interface khác**. Nó sẽ giúp **class đã tồn tại đó** làm việc với những thằng khác mà **không thay đổi source code**.

Nghe vẫn hơi trừu tượng đúng không nhỉ? Chúng ta sẽ từ từ đi bóc tách hết cái lớp vỏ ngoài khó hiểu này nhé.

Tuy nhiên thì có 2 cách dùng adapter là **Object Adapter** và **Class Adapter**. Chúng ta sẽ cùng đi tìm hiểu chi tiết từng cách.
# Object Adapter Pattern
## Sơ đồ lớp
![](https://images.viblo.asia/d9569a80-519f-4188-9484-3dffd19db10c.png)

Chúng ta sẽ implement interface bằng cách uỷ quyền tới object **adaptee** lúc run-time.

Mình sẽ giải thích chi tiết thêm một chút:

1. **Client** là class đã có sẵn mà mình nhắc đến trong phần khái niệm nhé.
2. **Client Interface** giờ là interface cha của **Client**. Những thằng khác muốn giao tiếp **Client** thì phải follow thằng interface cha này nhé.
3. **Service** là một thằng third party mà chỉ nhận param là JSON như mình đặt vấn đề.
4. **Adapter** implement **Client Interface** và chứa instance của class **Service** (object adaptee). Mỗi khi gọi đến method của **Adapter**, nó sẽ gọi đến method của **Service** này.
5. Như vậy chúng ta đã có thể làm việc với thằng **Service** mà không cần sửa code ở **Client**.
## Code sample
```java
// Client Interface
interface TypeCPhone {
    void chargeByTypeC();
}

// Interface của Service
interface LightningPhone {
    void chargeByLightning();
}

// Service
class IPhone implements LightningPhone {
    @Override
    public void chargeByLightning() {}
}

// Adapter
class TypeCToLightningPhoneAdapter implements TypeCPhone {
    private LightningPhone lightningPhone;
    
    public TypeCToLightningPhoneAdapter(LightningPhone lightningPhone) {
        this.lightningPhone = lightningPhone;
    }
    
    @Override
    public void chargeByTypeC() {
        lightningPhone.chargeByLightning();
    }
}

public class Demo {
    static void chargeMyXiaomiPhone(TypeCPhone phone) {
        phone.chargeByTypeC();
    }
    
    public static void main(String[] args) {
        IPhone iPhone = new IPhone();
        TypeCToLightningPhoneAdapter adapter = new TypeCToLightningPhoneAdapter(iPhone);
        chargeMyXiaomiPhone(adapter);
    }
}
```
# Class Adapter pattern
![](https://images.viblo.asia/3d70c8a9-c9cb-4eb6-a0eb-17a0d700acd6.png)
Class Adapter thì không cần wrap object nào cả vì nó kế thừa từ cả **Client** và **Service**. 

Rất tiếc là Java không support đa kế thừa nên chúng ta không có code sample bằng Java cho phần này.

# Kết luận
**Adapter** là một design pattern khá quen thuộc với chúng ta. Qua phần giải thích và một chút code mẫu, mong mọi người hiểu thêm về nó.

**Reference**
- https://en.wikipedia.org/wiki/Adapter_pattern
- https://refactoring.guru/design-patterns/adapter
