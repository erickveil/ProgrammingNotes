---
title: Swift Guard Condition
date: 01-04-24
layout: post
tags:
  - swift
  - "#conditionals"
---
Swift has all the usual conditional components you'd expect to find in a programming language. The only weird thing is the absence of parentheses in the condition definition.

There is another unusual component in that your condition definition can be an assignment. Using an assignment in place of a condition implicitly tests for `nil`.

```Swift
var myObj: SomeOptionalType? = ...

if let myObj = myObj {
    // myObj has a value and is not nil, do something with unwrapped myObj
} else {
    // myObj is nil, do something else
}
```

You can invert this check using the `guard` statement:

```Swift
var myObj: SomeOptionalType? = ...

guard let myObj = myObj else {
    // myObj is nil, do something then exit the scope
    return
}

// Proceed with non-nil myObj
```

You can use this to detect when the value is `nil`, then take some action - usually an early exit or perhaps assign a contextual default value instead.
