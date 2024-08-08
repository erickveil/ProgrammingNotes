---
title: C++ Int to Enum
layout: note
tags:
  - Cpp
---

To convert an `int` to an `enum` in C++, you can use a simple cast. 

First, let's define an enum:

```cpp
enum Color {
    RED,
    GREEN,
    BLUE
};
```

Now, let's assume you have a method that accepts an `int` and you want to set an enum value using that `int`. Here's how you can do it:

```cpp
#include <iostream>

enum Color {
    RED,
    GREEN,
    BLUE
};

class Example {
public:
    void setColor(int value) {
        Color color = static_cast<Color>(value);
        // Now you can use the color enum
        switch (color) {
            case RED:
                std::cout << "Color is RED" << std::endl;
                break;
            case GREEN:
                std::cout << "Color is GREEN" << std::endl;
                break;
            case BLUE:
                std::cout << "Color is BLUE" << std::endl;
                break;
            default:
                std::cout << "Invalid color" << std::endl;
                break;
        }
    }
};

int main() {
    Example example;
    example.setColor(1); // Output: Color is GREEN
    example.setColor(3); // Output: Invalid color
    return 0;
}
```

In this example, the `setColor` method takes an `int` value, casts it to the `Color` enum using `static_cast<Color>(value)`, and then uses it. Note that this does not prevent invalid values (e.g., 3 in this case) from being converted to the enum, so you may want to add some validation logic if necessary.

# Validation


```cpp
#include <iostream>

enum Color {
    RED,
    GREEN,
    BLUE,
    COLOR_COUNT // This is often used to get the count of enum values
};

class Example {
public:
    void setColor(int value) {
        if (value < 0 || value >= COLOR_COUNT) {
            std::cout << "Invalid color" << std::endl;
            return;
        }
        
        Color color = static_cast<Color>(value);
        // Now you can use the color enum
        switch (color) {
            case RED:
                std::cout << "Color is RED" << std::endl;
                break;
            case GREEN:
                std::cout << "Color is GREEN" << std::endl;
                break;
            case BLUE:
                std::cout << "Color is BLUE" << std::endl;
                break;
            default:
                std::cout << "Invalid color" << std::endl;
                break;
        }
    }
};

int main() {
    Example example;
    example.setColor(1); // Output: Color is GREEN
    example.setColor(3); // Output: Invalid color
    example.setColor(-1); // Output: Invalid color
    return 0;
}
```

In this example, the `COLOR_COUNT` value is added to the `enum` to represent the total number of valid colors. The `setColor` method checks if the provided `int` value is within the valid range of the enum values. If it is not, it prints "Invalid color" and returns early, avoiding the invalid cast and subsequent logic.