---
title: Struct vs Class in iOS
layout: note
tags:
  - iOS
  - swift
---

In Swift, both `struct` and `class` are used to define data models, but they have key differences in behavior. In the case of your mood tracker app, a `struct` might be more appropriate than a `class` due to the following reasons:

### Value Semantics vs. Reference Semantics
- **Structs** use **value semantics**, meaning that when you assign a struct to a new variable or pass it to a function, you're making a **copy** of that data. This is beneficial when you want to maintain immutability or ensure that data doesn't get accidentally modified elsewhere.
- **Classes** use **reference semantics**, meaning when you assign a class instance to a new variable or pass it to a function, you’re passing a **reference** to the original instance. If the object is modified in one place, that modification is reflected everywhere else.

### Performance
- **Structs** are typically more lightweight and performant for small, simple data types because they're stored on the stack. Since mood entries are immutable once created, `struct` is a good fit.
- **Classes**, on the other hand, are stored on the heap and involve reference counting, which can add overhead when managing a large number of objects.

### Immutability
Structs are often used when the data should be **immutable**, meaning it should not change once it’s created. In the context of your app, a mood entry is something that, once recorded, doesn’t usually need to change. By using a `struct`, you implicitly gain immutability unless you mark properties as `var`.

### When to Use a Class
You might want to use a class if:
- You need **inheritance**, or a hierarchy of objects.
- You need to manage shared mutable state where one instance needs to be accessed and updated from multiple places.
