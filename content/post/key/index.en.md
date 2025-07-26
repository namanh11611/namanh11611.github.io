---
title: "Investigation Case: Decoding the Mystery of Key in Flutter"
description: "Key appears everywhere in Flutter, from StatefulWidget to StatelessWidget. It seems so close yet so far, familiar yet mysterious. Today, the Flutter Vietnam detective team will take you deep into the corners of the Flutter gang to decode the role of Key."
date: 2025-05-12T00:00:00+07:00
slug: key
image: key.webp
toc: true
categories: [Technical, Flutter]
tags: [Flutter, Key]
---

> Tired of academic articles? Today, let's enjoy a detective-style article for a change of pace.

# Case Introduction

**Key** appears everywhere in Flutter, from **StatefulWidget** to **StatelessWidget**. It seems so close yet so far, familiar yet mysterious. Although Flutter developers often work with **Widget**, **Key**—the silent player behind all *power transfers*—rarely gets attention.

Today, the Flutter Vietnam detective team will take you deep into the corners of the **Flutter** gang, decoding the role of **Key** in **optimizing performance** for Flutter apps, and exploring **best practices** for using them.

# Searching for the Mastermind

The definition of Key in the [documentation](https://api.flutter.dev/flutter/foundation/Key-class.html) states:

> A Key is an identifier for Widgets, Elements and SemanticsNodes.
>
> A new widget will only be used to update an existing element if its key is the same as the key of the current widget associated with the element.
>
> *Translation:*
>
> *Key is an identifier for Widget, Element, and SemanticsNodes.*
>
> *A new widget will only be used to update an existing element if its key matches the key of the current widget associated with that element.*

As you know, in Flutter, everything is a **Widget**. This Widget gang includes familiar faces like **Row**, **Column**, **Container**... But these Widgets are just henchmen; according to our intelligence, the gang is run by a notorious boss: **Element**.

He manipulates all Widgets, from calling `initState`, `build`, `dispose` of Widgets to managing the Widget Tree. **Element** is also a crucial link between **Widget** and **RenderObject**—the one that draws the UI on the screen—to create masterpieces.

However, today we're not taking down the boss or the whole gang, just remember that **Element** orchestrates everything. The fish isn't big enough yet to cast the net. The goal of this case is to investigate **Key** and the four **masters**.

# Key

When you rebuild a Flutter app, do you know what happens in the shadows? **Element** decides which **Widgets** to keep, replace, or remove. This is where **Key** starts to show its power.

In the Widget world, whenever changes occur, Widgets aren't just **updated** but are **destroyed** and **reborn**. Key is the *identity card* that helps Widgets retain their identity during rebirth. After the Widget tree is rebuilt, Element uses **Widget type** and **Key** to decide whether the Element should be rebuilt. If Widget types differ, the Element is destroyed and recreated. Rebuilding the boss is much more expensive than rebuilding the henchmen Widgets. This can cause unwanted performance issues and sometimes make your app lag.

If Widget types are the same, Element compares the Key. If the Key matches, Element only updates the widget. Otherwise, Element is deactivated, meaning it's temporarily removed from the Element tree and may be reattached later.

There are two main types of Key: **LocalKey** and **GlobalKey**. LocalKey is further divided into **UniqueKey**, **ValueKey**, and **ObjectKey**. Let's unmask each one.

# UniqueKey – The Untraceable Assassin

A mysterious figure, appearing and disappearing, never showing up twice with the same value. It creates **unique values** to help Flutter **distinguish between two Widgets** even if they have the same type. Use it when you **don't want to reuse** any Widget, ensuring the Widget is completely rebuilt.

First, let's create an Item widget that simply displays a Text, but we'll use `StatefulWidget` so you can clearly see the init and rebuild process.

```dart
class Item extends StatefulWidget {
  final String text;

  const Item({super.key, required this.text});

  @override
  State<Item> createState() => _ItemState();
}

class _ItemState extends State<Item> {
  @override
  void initState() {
    super.initState();
    debugPrint('[_ItemState.initState] key = ${widget.key}, text = ${widget.text}');
  }

  @override
  Widget build(BuildContext context) {
    debugPrint('[_ItemState.build] key = ${widget.key}, text = ${widget.text}');
    return Text(widget.text);
  }
}
```

Next, create a `ListView` containing those Item widgets, without passing any Key to the items.

```dart
final names = ['Henry', 'Techie', 'Nam', 'Anh', 'Nguyen'];

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Demo Key')),
      body: ListView.builder(
        itemCount: _counter,
        itemBuilder: (context, index) => Item(
          text: names[index],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        child: Icon(Icons.add),
      ),
    );
  }
}
```

Check the log: when you click the `FloatingActionButton` to create each item, only the newest widget calls `initState`, others just `rebuild`. Here's the log when creating the 5th item:

```
[_ItemState.build] key = null, text = Henry
[_ItemState.build] key = null, text = Techie
[_ItemState.build] key = null, text = Nam
[_ItemState.build] key = null, text = Anh
[_ItemState.initState] key = null, text = Nguyen
[_ItemState.build] key = null, text = Nguyen
```

Now, let's assign the UniqueKey assassin. Add UniqueKey to Item:

```dart
body: ListView.builder(
  itemCount: _counter,
  itemBuilder: (context, index) => Item(
    key: UniqueKey(),
    text: names[index],
  ),
),
```

Everything changes: when clicking `FloatingActionButton` to create each item, all widgets are recreated. Even the Key value changes each time. Here's the log for the 5th item; not only item 5 but all items 1 to 4 are also re-initialized:

```
[_ItemState.initState] key = [#c24b5], text = Henry
[_ItemState.build] key = [#c24b5], text = Henry
[_ItemState.initState] key = [#02979], text = Techie
[_ItemState.build] key = [#02979], text = Techie
[_ItemState.initState] key = [#0a0db], text = Nam
[_ItemState.build] key = [#0a0db], text = Nam
[_ItemState.initState] key = [#23d4d], text = Anh
[_ItemState.build] key = [#23d4d], text = Anh
[_ItemState.initState] key = [#614a4], text = Nguyen
[_ItemState.build] key = [#614a4], text = Nguyen
```

# ValueKey – The Reliable Butcher

A simple-minded but highly effective worker, **ValueKey** is perfect when you have a clear identifier value, such as a `String` or `int`. This butcher helps Element know which Widget to keep based solely on that identifier, allowing you to **reuse** Widgets when the Key value doesn't change.

Let's update the previous `ListView` example to `ReorderableListView`. Since `ReorderableListView` requires each item to have a Key, we'll start with Item using `ValueKey`.

```dart
body: ReorderableListView(
  onReorder: (oldIndex, newIndex) {
    setState(() {
      if (newIndex > oldIndex) newIndex -= 1;
      final item = names.removeAt(oldIndex);
      names.insert(newIndex, item);
    });
  },
  children: names
      .map((item) => Item(
          key: ValueKey(item),
          text: item,
        ))
      .toList(),
),
```

Now, whenever you drag to reorder items, you'll see the items being rebuilt:

```
[_ItemState.build] key = [<'Henry'>], text = Henry
[_ItemState.build] key = [<'Techie'>], text = Techie
[_ItemState.build] key = [<'Nam'>], text = Nam
[_ItemState.build] key = [<'Anh'>], text = Anh
[_ItemState.build] key = [<'Nguyen'>], text = Nguyen
```

What if you change the value of an item after dragging by updating the `onReorder` function? For example, when dragging 'Henry', change its value to 'Henry Changed':

```dart
onReorder: (oldIndex, newIndex) {
  setState(() {
    if (newIndex > oldIndex) newIndex -= 1;
    final item = '${names.removeAt(oldIndex)} Changed';
    names.insert(newIndex, item);
  });
},
```

The value changes, so the ValueKey changes, and the item is recreated:

```
[_ItemState.initState] key = [<'Henry Changed'>], text = Henry Changed
[_ItemState.build] key = [<'Henry Changed'>], text = Henry Changed
```

# ObjectKey – The Strategic Advisor

Unlike ValueKey, this one is a master strategist, holding a complex object. **ObjectKey** relies on **object reference**. Two Keys are considered **equal** only if they reference the same object.

Similar to the ValueKey example, let's update it to use ObjectKey instead.

```dart
class _HomePageState extends State<HomePage> {
  final List<Person> people = [
    Person(name: "Henry"),
    Person(name: "Techie"),
    Person(name: "Nam"),
    Person(name: "Anh"),
    Person(name: "Nguyen"),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Demo Key')),
      body: ReorderableListView(
        onReorder: (oldIndex, newIndex) {
          setState(() {
            if (newIndex > oldIndex) newIndex -= 1;
            final item = people.removeAt(oldIndex);
            people.insert(newIndex, item);
          });
        },
        children: people
            .map((item) => Item(
          key: ObjectKey(item),
          text: item.name,
        ))
            .toList(),
      ),
    );
  }
}

class Person {
  final String name;
  Person({required this.name});
}
```

# GlobalKey – The Powerful Butler

In the gang, GlobalKey is the strongest. It knows everything in the Widget Tree. Not only does it store identity, but it also manages the entire state and allows direct access to `State`. This brings flexibility but can be easily abused.

The most common example is using GlobalKey to control and validate the state of a `Form`.

```dart
class GlobalKeyExample extends StatelessWidget {
  final GlobalKey<FormState> formKey = GlobalKey<FormState>();

  @override
  Widget build(BuildContext context) {
    return Form(
      key: formKey,
      child: Column(
        children: [
          TextFormField(validator: (value) => value!.isEmpty ? 'Required' : null),
          ElevatedButton(
            onPressed: () {
              if (formKey.currentState!.validate()) {
                print('Form is valid!');
              }
            },
            child: Text('Submit'),
          ),
        ],
      ),
    );
  }
}
```

# Best Practices for Using Key

**Key** is not just a tool, but a *treasure* for controlling your app. Understanding and using Key correctly not only optimizes performance but also ensures your app's logic remains stable and accurate. Overusing it can make your code unnecessarily complex, so use it only when needed. Remember, in the ever-changing world of Flutter, Key is the key to smooth apps you develop!
