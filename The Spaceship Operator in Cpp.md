---
title: The Spaceship Operator in C++
layout: note
tags:
  - Cpp
  - operator
---
The spaceship operator (`<=>`) in C++20, also known as the three-way comparison operator, is a new feature introduced to simplify and standardize comparisons. It is used to perform a three-way comparison between two values, returning a value that indicates whether one value is less than, equal to, or greater than the other.

Here's a detailed explanation:

### Syntax
The spaceship operator is written as `<=>`. Its syntax for overloading is similar to other operators:

```cpp
#include <compare>

class MyClass {
public:
    int value;

    // Overload the spaceship operator
    auto operator<=>(const MyClass& other) const = default;
};
```

### Return Type
The `<=>` operator returns a type from the `<compare>` header, typically one of:
- `std::strong_ordering`
- `std::weak_ordering`
- `std::partial_ordering`

These types provide the result of the comparison:
- `std::strong_ordering`: Indicates a strong ordering (less, equal, or greater).
- `std::weak_ordering`: Indicates a weak ordering, which allows for equivalence (e.g., when two distinct objects are considered equivalent).
- `std::partial_ordering`: Indicates a partial ordering, useful when some elements are incomparable.

### Default Behavior
You can automatically generate a default spaceship operator by using `= default`. This instructs the compiler to generate the operator based on the member-wise comparison of the class.

### Example Usage
Here's a practical example demonstrating the spaceship operator:

```cpp
#include <iostream>
#include <compare>

struct Point {
    int x, y;

    // Default spaceship operator
    auto operator<=>(const Point& other) const = default;
};

int main() {
    Point p1{1, 2};
    Point p2{1, 3};

    if (auto result = p1 <=> p2; result < 0) {
        std::cout << "p1 is less than p2\n";
    } else if (result > 0) {
        std::cout << "p1 is greater than p2\n";
    } else {
        std::cout << "p1 is equal to p2\n";
    }

    return 0;
}
```

In this example:
- `p1 <=> p2` performs a three-way comparison between `p1` and `p2`.
- The result of `p1 <=> p2` is compared to `0` to determine if `p1` is less than, equal to, or greater than `p2`.

### Advantages
1. **Simplification**: It simplifies the code by reducing the need to write multiple comparison operators (`<`, `<=`, `>`, `>=`, `==`, `!=`).
2. **Consistency**: It ensures consistency in the comparison logic.
3. **Performance**: Potentially more efficient as the compiler can optimize the comparison operations better.

### Custom Implementation
You can also implement a custom spaceship operator if default behavior is not suitable:

```cpp
struct CustomPoint {
    int x, y;

    auto operator<=>(const CustomPoint& other) const {
        if (auto cmp = x <=> other.x; cmp != 0) {
            return cmp;
        }
        return y <=> other.y;
    }
};
```

In this custom implementation, the `x` coordinates are compared first, and if they are equal, the `y` coordinates are compared.

### Lexicographical Comparison
When using the spaceship operator for composite objects, the comparison is usually lexicographical. This means the comparison is done member by member, in the order they are declared. For `Point` with members `x` and `y`, the comparison proceeds as follows:
1. Compare the `x` members of both `Point` objects.
2. If the `x` members are different, the comparison result of the `x` members determines the overall result.
3. If the `x` members are the same, then compare the `y` members.
4. The result of comparing the `y` members determines the overall result if the `x` members are equal.

### Example
Here’s a more detailed example demonstrating how this works:

```cpp
#include <iostream>
#include <compare>

struct Point {
    int x, y;

    // Default spaceship operator
    auto operator<=>(const Point& other) const = default;
};

int main() {
    Point p1{1, 2};
    Point p2{1, 3};
    Point p3{2, 1};

    // p1 and p2 comparison
    if (auto result = p1 <=> p2; result < 0) {
        std::cout << "p1 is less than p2\n";
    } else if (result > 0) {
        std::cout << "p1 is greater than p2\n";
    } else {
        std::cout << "p1 is equal to p2\n";
    }

    // p1 and p3 comparison
    if (auto result = p1 <=> p3; result < 0) {
        std::cout << "p1 is less than p3\n";
    } else if (result > 0) {
        std::cout << "p1 is greater than p3\n";
    } else {
        std::cout << "p1 is equal to p3\n";
    }

    return 0;
}
```

1. **p1 vs. p2**:
   - `p1.x` is 1 and `p2.x` is 1: They are equal.
   - `p1.y` is 2 and `p2.y` is 3: `p1.y` is less than `p2.y`.
   - Therefore, `p1 < p2`.

2. **p1 vs. p3**:
   - `p1.x` is 1 and `p3.x` is 2: `p1.x` is less than `p3.x`.
   - Therefore, `p1 < p3` without needing to compare the `y` values.

### Custom Comparison
If lexicographical order is not desired or if a more complex comparison is needed, you can implement the spaceship operator manually:

```cpp
struct Point {
    int x, y;

    auto operator<=>(const Point& other) const {
        if (x != other.x) {
            return x <=> other.x;
        }
        return y <=> other.y;
    }
};
```

This custom implementation explicitly defines the lexicographical comparison logic, allowing for potential customization if different comparison rules are needed.

# Non Numeric Comparisons

If the members of a class aren't numbers, but instead other types (e.g., strings, custom objects, etc.), the comparison can still be done using the spaceship operator (`<=>`), provided that these member types themselves support comparison operations. 

### Example with Strings

Let's consider a class with `std::string` members:

```cpp
#include <iostream>
#include <string>
#include <compare>

struct Person {
    std::string firstName;
    std::string lastName;

    // Default spaceship operator
    auto operator<=>(const Person& other) const = default;
};

int main() {
    Person p1{"John", "Doe"};
    Person p2{"Jane", "Doe"};
    Person p3{"John", "Smith"};

    // p1 and p2 comparison
    if (auto result = p1 <=> p2; result < 0) {
        std::cout << "p1 is less than p2\n";
    } else if (result > 0) {
        std::cout << "p1 is greater than p2\n";
    } else {
        std::cout << "p1 is equal to p2\n";
    }

    // p1 and p3 comparison
    if (auto result = p1 <=> p3; result < 0) {
        std::cout << "p1 is less than p3\n";
    } else if (result > 0) {
        std::cout << "p1 is greater than p3\n";
    } else {
        std::cout << "p1 is equal to p3\n";
    }

    return 0;
}
```


1. **Comparison Process**:
   - `p1` vs. `p2`: 
     - Compare `firstName`: "John" vs. "Jane". "John" is lexicographically greater than "Jane".
     - Therefore, `p1 > p2`.
   
   - `p1` vs. `p3`:
     - Compare `firstName`: "John" vs. "John". They are equal.
     - Compare `lastName`: "Doe" vs. "Smith". "Doe" is lexicographically less than "Smith".
     - Therefore, `p1 < p3`.

### Custom Comparison

If you need a specific comparison logic, you can define the spaceship operator manually:

```cpp
struct Person {
    std::string firstName;
    std::string lastName;

    auto operator<=>(const Person& other) const {
        if (auto cmp = firstName <=> other.firstName; cmp != 0) {
            return cmp;
        }
        return lastName <=> other.lastName;
    }
};
```

### Handling Custom Objects

If the members are custom objects, ensure those objects also support the spaceship operator or other comparison operators.

Example with a custom type:

```cpp
#include <iostream>
#include <string>
#include <compare>

struct Date {
    int year, month, day;

    auto operator<=>(const Date& other) const = default;
};

struct Event {
    std::string title;
    Date date;

    auto operator<=>(const Event& other) const {
        if (auto cmp = date <=> other.date; cmp != 0) {
            return cmp;
        }
        return title <=> other.title;
    }
};

int main() {
    Event event1{"Meeting", {2023, 5, 24}};
    Event event2{"Conference", {2024, 5, 24}};

    // event1 and event2 comparison
    if (auto result = event1 <=> event2; result < 0) {
        std::cout << "event1 is less than event2\n";
    } else if (result > 0) {
        std::cout << "event1 is greater than event2\n";
    } else {
        std::cout << "event1 is equal to event2\n";
    }

    return 0;
}
```


1. **Date** struct supports the spaceship operator.
2. **Event** struct compares `date` first and `title` second.

The spaceship operator can handle members that are not numbers, provided those member types support comparison. For standard library types like `std::string` and user-defined types with appropriate comparison operators, the same principles apply. If necessary, custom comparison logic can be defined by manually implementing the spaceship operator.