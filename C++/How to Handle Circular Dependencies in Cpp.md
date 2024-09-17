---
title: How to Handle Circular Dependencies in C++
layout: note
tags:
  - Cpp
---

# The Problem

I have a chicken and the egg problem with my C++ program. In ClassA I need to call a method from ClassB. But in Class B, I need to call a method from ClassA. The linker doesn't like it when ClassA has an include for ClassB and also ClassB has an include for ClassA. How do I restructure this to work?

```c++

#include "MyClassB.h"

class MyClassA {

public:
	void method1() { 
		MyClassB objB;
		objB.method3();
	}

	void method2() { 
	}

}
```

```c++

#include "MyClassA.h"

class MyClassB {

public:
	void method3() {
	
	}
	
	void method4() {
		MyClassA objA;
		objA.method2();
	}

}
```

In languages like C#, this is automatically handled for us and we don't have to worry about it too much. But in C++, we have to do a little boilerplate work to make it happen.

# Forward Declaration

To solve the circular dependency problem in C++, you can use forward declarations to break the direct inclusion cycle. Here's how you can restructure your code:

1. **Use forward declarations** to inform each class about the existence of the other without including their headers directly.

2. **Include headers in the implementation files** (.cpp files) rather than in the header files (.h files).

Here's how you can adjust your classes:

### MyClassA.h
```c++
// Forward declaration of MyClassB
class MyClassB;

class MyClassA {
public:
    void method1();
    void method2();
};
```

### MyClassB.h
```c++
// Forward declaration of MyClassA
class MyClassA;

class MyClassB {
public:
    void method3();
    void method4();
};
```

### MyClassA.cpp
```c++
#include "MyClassA.h"
#include "MyClassB.h"

void MyClassA::method1() { 
    MyClassB objB;
    objB.method3();
}

void MyClassA::method2() {
    // Implementation of method2
}
```

### MyClassB.cpp
```c++
#include "MyClassB.h"
#include "MyClassA.h"

void MyClassB::method3() {
    // Implementation of method3
}

void MyClassB::method4() {
    MyClassA objA;
    objA.method2();
}
```

By forward-declaring the classes in the header files and including the full definitions in the implementation files, you break the circular dependency. This approach allows the linker to resolve the dependencies correctly.
