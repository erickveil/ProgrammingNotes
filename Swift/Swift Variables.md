
In Swift, you can define **variables** using the `var` keyword and **constants** using the `let` keyword. Here's how you can declare and define variables and constants:

1. Declare and define a variable:
```Swift
var myVariable: Int = 10
```

2. Declare a variable without defining it (initialize it later):
```Swift
var myVariable: Int 
// You can assign a value to 'myVariable' later in your code
```

3. Declare and define a constant:
```Swift
let myConstant: String = "Hello, Swift!"
```

4. Declare a constant without defining it (initialize it later):
```Swift
let myConstant: String 
// You can assign a value to 'myConstant' later in your code, but you can't change it once it's set
```

You cannot define a variable first and then declare it later separately in Swift. Once you declare a variable or constant, you either define it at the same time or leave it undefined if you plan to assign a value to it later.

# Default Types

Several default data types are available for storing different kinds of values. Here are some of the most commonly used default data types:

1. Int
2. Double
3. Float
4. Bool
5. String
6. Character
7. Array
8. Dictionary
9. Set

# Arrays Dictionaries and Sets

## Defining:

Here are examples of how to declare and define arrays, dictionaries, and sets in Swift, both separately and in one step:

1. Arrays:
```Swift
// Declare and define an array of Integers
var numbers: [Int] = [1, 2, 3, 4, 5]

// Declare an array without defining it (initialize it later)
var names: [String]

// Define the array later
names = ["Alice", "Bob", "Charlie"]

```

2. Dictionaries:
```Swift
// Declare and define a dictionary with String keys and Int values
var ages: [String: Int] = ["Alice": 30, "Bob": 25, "Charlie": 35]

// Declare a dictionary without defining it (initialize it later)
var scores: [String: Int]

// Define the dictionary later
scores = ["Alice": 95, "Bob": 87, "Charlie": 92]

```


3. Sets:
```Swift
// Declare and define a set of Integers
var uniqueNumbers: Set<Int> = [1, 2, 3, 4, 5]

// Declare a set without defining it (initialize it later)
var uniqueLetters: Set<Character>

// Define the set later
uniqueLetters = ["a", "b", "c"]

```

## Referencing:

