---
title: "Design Pattern: Delegation trong Kotlin - cách để nhờ người khác làm bài tập về nhà"
description: "Trước đây mình cũng chưa biết về Delegation Pattern, cho đến khi học Kotlin thì thấy người ta hay dùng keyword by trong lúc khai báo một biến. Vậy là thử tìm hiểu thì cả một chân trời mở ra về pattern mới này."
date: 2023-05-12T16:50:00+07:00
slug: design-pattern-delegation
image: delegation.webp
categories: [Technical]
tags: [Design Pattern, Delegation, Kotlin]
---

# Khái niệm

Trước đây mình cũng chưa biết về Delegation Pattern, cho đến khi học Kotlin thì thấy người ta hay dùng keyword `by` trong lúc khai báo một biến. Vậy là thử tìm hiểu thì cả một chân trời mở ra về pattern mới này.

~~ *Hơi đao to búa lớn quá rồi* 😅😅 ~~

Để cho dễ hiểu thì **Delegation** dịch ra tiếng Việt là **Sự uỷ nhiệm**. Nhưng mình nghĩ là chúng ta sẽ dùng các từ "delegation" và "delegate" thay cho từ "sự uỷ nhiệm" và "uỷ nhiệm" để giữ gìn sự trong sáng của tiếng Anh. Đùa vậy thôi chứ trong công việc hay lúc đi phỏng vấn, các bạn nên dùng từ gốc tiếng Anh để cho chuyên nghiệp và đối phương cũng dễ nắm bắt ý của mình nhé.

> The **Delegation Pattern** is an object-oriented design pattern that allows object composition to achieve the same code reuse as inheritance.
>
> *Dịch nôm na: Delegation Pattern cho phép object tái sử dụng code tương tự như kế thừa.*

Trong delegation, chúng ta có 2 thành phần:

* Receiving object
* Delegate object

Khi có một request cần handle, **receiving object** sẽ không trực tiếp handle nó mà delegate tác vụ đó cho **delegate object**. Nó giống như việc bạn có một ông anh và bà chị rất giỏi Toán, mỗi lần cô giáo giao bài tập về nhà, bạn lại nhờ họ làm hộ vậy.

Ơ thế nó khác gì **Kế thừa** nhỉ? Trong **Kế thừa** chúng ta cũng có thể gọi đến method của parent class mà?

Đúng là Kế thừa rất hữu ích nhưng chúng ta chỉ dùng khi child class thực sự có liên quan về mặt ý nghĩa chính xác với parent class. Ví dụ như class `Cat` có thể kế thừa class `Animal` nhưng không nên kế thừa class `Transportation` vậy. Chưa kể child class phải override tất cả các abstract method của parent class, đôi khi điều đó là không cần thiết.  Vậy nên Delegation giúp chúng ta linh hoạt hơn.

# Ví dụ minh hoạ

Quay trở lại với ví dụ ban nãy, giả sử bạn có một ông anh là kỹ sư và một bà chị là bác sỹ. Họ đều là những người rất tài năng.

```kotlin
interface TalentPerson {
    fun doHomework()
}

class Engineer : TalentPerson {
    override fun doHomework() {}
}

class Doctor : TalentPerson {
    override fun doHomework() {}
}
```

Còn bạn là một học sinh hơi lười một chút nên cần đến một class helper, với param là một người tốt bụng nào đó luôn sẵn sàng giúp đỡ bạn mọi lúc khó khăn.

```kotlin
class LazyStudentHelper(private val kindPerson: TalentPerson) {
    fun doHomeworkByMyself() {
        kindPerson.doHomework()
    }
}
```

Vậy là bây giờ, mỗi lần giáo viên giao bài tập về nhà, bạn sẽ lại nhờ đến ông anh hoặc bà chị làm hộ.

```kotlin
fun main() {
    val brother = Engineer()
    val lazyBoy = LazyStudentHelper(brother)
    lazyBoy.doHomeworkByMyself()
    
    val sister = Doctor()
    val lazyGirl = LazyStudentHelper(sister)
    lazyGirl.doHomeworkByMyself()
}
```

# Delegation Pattern trong Kotlin

## Keyword `by` trong Kotlin

Trong Kotlin đã support **Delegation Pattern** thông qua keyword `by`, giúp chúng ta giảm boilerplate code.

```kotlin
class LazyStudentHelper(
    private val kindPerson: TalentPerson
): TalentPerson by kindPerson

fun main() {
    val brother = Engineer()
    val lazyBoy = LazyStudentHelper(brother)
    lazyBoy.doHomework()

    val sister = Doctor()
    val lazyGirl = LazyStudentHelper(sister)
    lazyGirl.doHomework()
}
```

Giờ đây, class `LazyStudentHelper` phải implement interface `TalentPerson`, qua đó nó có thể delegate method `doHomework` qua instance `kindPerson`.

## Delegated properties

Chúng ta có một số cách để ứng dụng Delegation khi khai báo biến trong Kotlin:

* **Lazy** properties: giá trị sẽ được tính toán trong lần đầu tiên access.
* **Observable** properties: listeners sẽ được thông báo về những thay đổi của property này.

### Lazy properties

`lazy` là một function có param là lambda và trả về kết quả là một instance của class `Lazy<T>`.

```kotlin
val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}

fun main() {
    println(lazyValue)
    println(lazyValue)
}
```

Lần gọi biến `lazyValue` đầu tiên sẽ tính toán kết quả và lưu lại giá trị, những lần gọi sau chỉ trả về giá trị mà không cần tính toán kết quả. Vậy nên kết quả in ra sẽ là:

```
computed!
Hello
Hello
```

### Observable properties

```kotlin
import kotlin.properties.Delegates

class User {
    var name: String by Delegates.observable("<no name>") { prop, old, new ->
        println("$old -> $new")
    }
}

fun main() {
    val user = User()
    user.name = "first"
    user.name = "second"
}
```

`Delegates.observable()` có 2 argument:

* Giá trị khởi tạo
* Function để xử lý khi biến `name` thay đổi. Nó được gọi mỗi khi chúng ta assign giá trị cho biến.

Kết quả như sau:

```
<no name> -> first
first -> second
```

# Reference

* https://en.wikipedia.org/wiki/Delegation_pattern
* https://kotlinlang.org/docs/delegation.html
* https://kotlinlang.org/docs/delegated-properties.html
