---
title: Swift Classes
date: 24-03-18
layout: post
tags:
  - swift
  - "#types"
  - "#class"
---
# Declaring a Class

In Swift, defining a class involves specifying its properties and methods within a class declaration. Here's how you can define a simple class in Swift:

```Swift
class Person {
    var name: String
    var age: Int

    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }

    func introduce() -> String {
        return "Hello, my name is \(name) and I am \(age) years old."
    }
}
```

**Initializer**: The `init` method is a special method called an initializer. It's used to create an instance of the class with its properties initialized to their starting values. Replace `propertyName` with the name of your property and `PropertyType` with the type of your property.

# Public or Private?

In Swift, the members (properties and methods) of a class are `internal` by default. This means they are accessible within the same module or target in which the class is defined, but not from outside the module.

Swift provides five levels of access control:

1. **`private`**: Access is limited to the enclosing declaration (e.g., a class) and to extensions of that declaration that are in the same file.
2. **`fileprivate`**: Access is limited to the current file.
3. **`internal`** (default): Access is limited to the module (e.g., an app or framework) that includes the definition.
4. **`public`**: Access is open to any file within the module that imports the module in which the access level is defined, but not to subclasses or overrides outside the module.
5. **`open`**: Similar to `public`, but also allows subclasses and overrides outside the module.

Here's an example to illustrate the default `internal` access level:

```Swift
class ExampleClass {
    var internalProperty = "This can be accessed within the same module"
    func internalMethod() {
        print("This can be accessed within the same module")
    }
}

public class PublicClass {
    var publicProperty = "This can be accessed from any module that imports this module"
    public func publicMethod() {
        print("This must be explicitly marked as public to be accessed from any module")
    }
}
```

## Modules and Scope

A "module" refers to a single unit of code distribution—a framework or application that can be imported by another module. It's a namespace that encapsulates its contents (classes, structs, functions, etc.), allowing you to organize your code into reusable components. A module can be built as a standalone library, framework, or application, and it provides a way to package code in a manner that promotes encapsulation and reduces naming conflicts.

Modules in Swift are similar to packages in some other programming languages. When you import a module into a Swift file, you gain access to its public and open interfaces (e.g., classes, structures, functions, and properties) but not to elements marked as internal, fileprivate, or private, unless the code accessing them is part of the same module.

Members declared as `internal` in a module are accessible only within the same module. When you import a module, you cannot access its `internal` members, because `internal` implies access is restricted to the module where the member is defined.

You have to mark the class itself as `public` or `open` to make it accessible outside the module when it is imported. By default, classes (and other entities like structures, enums, functions, and properties) are `internal`. This means they are accessible within the same module but not from outside the module.

# Instantiation

To instantiate the class you've defined, you create a new instance by calling its initializer with the required parameters. Here's how you do it:

```Swift
let person = Person(name: "John Doe", age: 30)
```

This line of code creates a new `Person` object with the name "John Doe" and age 30. You use the `let` keyword to declare a constant named `person` that refers to the instance. If you plan to modify the properties of the `Person` instance later, you should use `var` instead of `let` to declare it.

After instantiation, you can access its properties and methods like this:

```Swift
// Accessing properties
print(person.name) // Output: John Doe
print(person.age)  // Output: 30

// Calling a method
print(person.introduce()) // Output: Hello, my name is John Doe and I am 30 years old.
```

# Subclassing

Here's an example to illustrate how to create a subclass. Let's say we have a `Person` class, as defined previously:

```Swift
class Person {
    var name: String
    var age: Int

    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }

    func introduce() -> String {
        return "Hello, my name is \(name) and I am \(age) years old."
    }
}
```

Now, suppose we want to create an `Employee` class that inherits from `Person` but also includes an additional property for the employee's job title:

```Swift
class Employee: Person {
    var jobTitle: String

    init(name: String, age: Int, jobTitle: String) {
        self.jobTitle = jobTitle
        // Call the superclass's initializer
        super.init(name: name, age: age)
    }

    // Example of method overriding
    override func introduce() -> String {
        return super.introduce() + " I work as a \(jobTitle)."
    }
}
```

In this `Employee` class:

- It inherits from `Person` by specifying `: Person` after its name.
- It adds a new property, `jobTitle`.
- It overrides the `init` method to include the initialization of `jobTitle`. Inside the `init`, it calls `super.init(name: age:)` to ensure the properties inherited from `Person` are properly initialized.
- It overrides the `introduce` method to extend the behavior of the method inherited from `Person`. The `override` keyword is required to override a method. `super.introduce()` is called to include the original introduction message, and then it adds information about the job title.

To instantiate an `Employee` and use its properties and methods:

```Swift
let employee = Employee(name: "Jane Doe", age: 28, jobTitle: "Software Engineer")
print(employee.introduce())
// Output: Hello, my name is Jane Doe and I am 28 years old. I work as a Software Engineer.
```

# Interface?

In Swift, the concepts of interfaces or partially abstract classes are not directly represented as they might be in languages like Java or C#. However, Swift provides protocols and protocol extensions, which can serve similar purposes.

### Protocols

A protocol defines a blueprint of methods, properties, and other requirements that suit a particular piece of functionality. They do not provide implementations of these requirements; they merely define the interface that conforming types must implement.

```Swift
protocol SomeProtocol {
    var mustBeSettable: Int { get set }
    var doesNotNeedToBeSettable: Int { get }
    func someMethod()
}
```

Any type that conforms to `SomeProtocol` must implement the properties and methods it declares.

### Protocol Extensions

While protocols are similar to interfaces, protocol extensions are what enable you to provide a default implementation of methods, properties, and subscripts. This means you can create something like a partially abstract class.

```Swift
extension SomeProtocol {
    func someMethod() {
        print("This is a default implementation.")
    }
}

class MyClass: SomeProtocol {
    var mustBeSettable: Int = 0
    var doesNotNeedToBeSettable: Int = 0
    
    // MyClass doesn't need to provide an implementation of someMethod,
    // unless it wants to offer behavior different from the default.
}
```

# Does Swift Pass by Ref or Copy?

Swift does not have a "copy constructor" concept in the same way that languages like C++ do.
### Value Types (Structs and Enums)
Swift's value types (like structs and enums) are copied automatically when you assign them to a new variable or pass them to a function. This behavior is due to Swift's copy-on-write mechanism, which ensures that a unique instance of the variable is created only if and when the variable is modified. This makes copying of value types efficient and straightforward, without the need for a copy constructor:

```swift
struct ExampleStruct {
    var number: Int
}

var original = ExampleStruct(number: 1)
var copy = original // `copy` is now a separate instance
copy.number = 2 // Only `copy` is modified, not `original`
```

### Reference Types (Classes)
For classes, which are reference types, Swift does not automatically copy an instance when you assign it to a new variable or pass it around. Instead, both variables would refer to the same instance. If you need to create a copy of a class instance, you have to implement that functionality yourself, typically by defining a method or initializer that creates a new instance of the class with the same properties as the instance you're copying:

```swift
class ExampleClass {
    var number: Int
    init(number: Int) {
        self.number = number
    }
    
    // Custom method to "copy" the instance
    func copy() -> ExampleClass {
        return ExampleClass(number: self.number)
    }
}

let original = ExampleClass(number: 1)
let copy = original.copy() // `copy` is a new instance with the same `number` value
copy.number = 2 // Only `copy` is modified, not `original`
```

This method acts similarly to a copy constructor but is manually defined. Swift's emphasis on value types for data that needs to be copied and the explicit handling required for copying reference types helps clarify ownership and mutation in your code.