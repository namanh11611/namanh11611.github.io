---
title: "Design Pattern: Delegation in Kotlin - how to get someone else to do your homework"
description: "I didn't know about the Delegation Pattern until I learned Kotlin and saw people using the 'by' keyword when declaring a variable. So I tried to learn more and discovered a whole new world of this pattern."
date: 2023-05-12T16:50:00+07:00
slug: design-pattern-delegation
image: delegation.webp
categories: [Technical]
tags: [Design Pattern, Delegation, Kotlin]
---

# Concept

I didn't know about the Delegation Pattern until I learned Kotlin and saw people using the `by` keyword when declaring a variable. So I tried to learn more and discovered a whole new world of this pattern.

~~ *A bit dramatic, isn't it?* 😅😅 ~~

To make it easy to understand, **Delegation** means **assigning tasks to someone else**. But let's just use the terms "delegation" and "delegate" to keep things clear and professional, especially in work or interviews.

> The **Delegation Pattern** is an object-oriented design pattern that allows object composition to achieve the same code reuse as inheritance.
>
> *In short: Delegation Pattern allows objects to reuse code similarly to inheritance.*

In delegation, there are two components:

* Receiving object
* Delegate object

When a request needs to be handled, the **receiving object** doesn't handle it directly but delegates the task to the **delegate object**. It's like having an older brother or sister who's really good at math, and every time the teacher gives you homework, you ask them to do it for you.

So how is this different from **Inheritance**? In inheritance, you can also call methods from the parent class, right?

That's true, but inheritance should only be used when the child class is truly related to the parent class. For example, a `Cat` class can inherit from `Animal`, but shouldn't inherit from `Transportation`. Also, the child class must override all abstract methods of the parent, which is sometimes unnecessary. Delegation gives us more flexibility.

# Example Illustration

Back to the earlier example, suppose you have a brother who's an engineer and a sister who's a doctor. They're both very talented.

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

You're a bit of a lazy student, so you need a helper class, with a parameter being a kind person who's always ready to help you out.

```kotlin
class LazyStudentHelper(private val kindPerson: TalentPerson) {
    fun doHomeworkByMyself() {
        kindPerson.doHomework()
    }
}
```

Now, every time the teacher gives homework, you can ask your brother or sister to do it for you.

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

# Delegation Pattern in Kotlin

## The `by` Keyword in Kotlin

Kotlin supports the **Delegation Pattern** through the `by` keyword, which helps reduce boilerplate code.

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

Now, the `LazyStudentHelper` class must implement the `TalentPerson` interface, so it can delegate the `doHomework` method to the `kindPerson` instance.

## Delegated Properties

There are several ways to use Delegation when declaring variables in Kotlin:

* **Lazy** properties: the value is computed the first time it is accessed.
* **Observable** properties: listeners are notified when the property changes.

### Lazy properties

`lazy` is a function that takes a lambda and returns an instance of the `Lazy<T>` class.

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

The first time `lazyValue` is called, the result is computed and stored. Subsequent calls just return the stored value. So the output will be:

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

`Delegates.observable()` takes two arguments:

* The initial value
* A function to handle changes to the `name` variable. It's called every time you assign a value to the variable.

The result will be:

```
<no name> -> first
first -> second
```

# Reference

* https://en.wikipedia.org/wiki/Delegation_pattern
* https://kotlinlang.org/docs/delegation.html
* https://kotlinlang.org/docs/delegated-properties.html
