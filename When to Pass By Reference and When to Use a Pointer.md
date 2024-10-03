---
title: When to Pass By Reference and When to Use a Pointer
layout: note
tags:
  - Cpp
---

In C++, whether to pass an object's pointer or reference largely depends on the specific needs of your program and how you intend to use the object within the function.

### When to Pass by Reference:
1. **Non-nullable Object**: If the function requires that an object must be passed and cannot be null, use a reference. References must be bound to valid objects and cannot be null, which provides a guarantee that the function will always have a valid object to work with.

2. **Simpler Syntax**: References are easier to use syntactically since you don’t need to dereference them to access the object’s members. This can make your code cleaner and more readable.

3. **No Ownership Transfer**: If the function should not take ownership of the object and just needs to operate on it without changing the ownership semantics, references are a good choice.

4. **Const Correctness**: If the object should not be modified within the function, passing a `const` reference ensures that the function cannot change the object’s state.

### When to Pass by Pointer:
1. **Nullable Object**: If the object is optional or may not always be provided, pass a pointer. A pointer can be null, which allows the function to check for the existence of an object before operating on it.

2. **Indicating Ownership**: Passing a pointer can imply that the function might take ownership of the object or manage its lifetime in some way (e.g., if the function is expected to delete the object).

3. **Multiple Objects**: If the function needs to modify which object a pointer points to (e.g., reassign the pointer to a different object), you must pass a pointer (or a reference to a pointer).

4. **Legacy Code**: If you're interfacing with C libraries or older C++ code, you might need to use pointers because that’s how the API expects to receive objects.

### Example Usage:

```cpp
void modifyObject(MyClass& obj) {
    // Cannot be null, simpler syntax, modifies the object
    obj.modify();
}

void optionalModify(MyClass* obj) {
    if (obj) {
        // Checks if object is not null, then modifies it
        obj->modify();
    }
}
```

- **`modifyObject`** requires a valid object and will always operate on it.
- **`optionalModify`** allows for a null object and operates conditionally.

# Returning a Reference or Pointer

When deciding between returning a pointer or a reference in C++, the choice depends on how you want to handle the ownership, the possibility of returning a null value, and the expected usage in the calling code.

### When to Return a Reference:
1. **Guaranteed Valid Object**: If you are certain that the function will always have a valid object to return, and the object’s lifetime is guaranteed to outlast the reference, return a reference. This avoids the possibility of returning a null value, simplifying the caller's logic.

2. **Non-Null Object**: If the returned value must always be a valid object and cannot be null, a reference is appropriate. It enforces that the caller will always receive a valid object.

3. **Performance**: Returning a reference avoids copying the object and is generally more efficient when dealing with large objects, as it avoids the overhead of returning by value.

4. **Chaining**: Returning a reference allows for function chaining, where multiple member functions can be called in sequence on the returned object.

   ```cpp
   MyClass& modifyObject() {
       // Modify internal state and return reference to *this
       this->modify();
       return *this;
   }
   ```

### When to Return a Pointer:
1. **Optional Object**: If the function might not always have an object to return (e.g., if a search fails or a condition isn’t met), returning a pointer is appropriate. This allows the caller to check for a null pointer, indicating the absence of an object.

2. **Ownership Considerations**: If the function is responsible for managing the object's lifetime and the caller might need to delete the object or take ownership, returning a pointer (especially a raw or smart pointer) makes the ownership semantics clearer.

3. **Legacy Code or API Design**: If your function is interacting with legacy code or an API that expects pointers, you might need to return a pointer.

4. **Reassignment Possibility**: If the object that might be returned could change during the program's execution, returning a pointer might make it clear that the object is subject to such changes.

   ```cpp
   MyClass* findObject(int id) {
       if (objectExists(id)) {
           return &objects[id];
       } else {
           return nullptr; // Indicates that no object was found
       }
   }
   ```

### Considerations for Smart Pointers:
- **`std::unique_ptr` or `std::shared_ptr`**: If you’re managing dynamic memory and want to clearly express ownership semantics, returning a smart pointer (like `std::unique_ptr` or `std::shared_ptr`) can be a better choice than a raw pointer. This helps avoid memory leaks and clearly communicates ownership to the caller.

### Example Usage:
```cpp
MyClass& getInstance() {
    static MyClass instance;
    return instance; // Guaranteed to return a valid reference
}

MyClass* findOrCreate(int id) {
    if (objectExists(id)) {
        return &objects[id];
    } else {
        return createNewObject(id); // May return a pointer to a new object or null
    }
}
```

- **`getInstance`** returns a reference to a singleton object, ensuring the caller always has a valid object.
- **`findOrCreate`** might return a pointer, allowing for null to indicate failure or non-existence.

# Is Returning a Pointer Safe Out Of the Scope That Created It?

If you are returning a pointer to an object that was **dynamically created** using `new`, the memory remains reserved for that object until you explicitly call `delete` on that pointer. This is because dynamically allocated memory lives on the **heap**, and the lifetime of objects on the heap is not tied to the function's scope. Therefore:

- The memory remains allocated and reserved until you call `delete`.
- The function that created the object can go out of scope, but the dynamically allocated object will persist because it's on the heap, not on the stack.

### Example:
```cpp
MyClass* createObject() {
    MyClass* obj = new MyClass(); // Dynamically allocate object on the heap
    return obj; // Return pointer to dynamically allocated object
}

int main() {
    MyClass* myObject = createObject(); // Now main() has a pointer to the created object
    // Use myObject...

    delete myObject; // Deallocate the memory when done
}
```
In this example:

- The `createObject` function dynamically allocates an object using `new` and returns a pointer to it.
- The memory for `myObject` will remain valid after `createObject` finishes, since the allocation was done on the heap.
- It is the responsibility of the caller (in this case, `main`) to `delete` the pointer when it is no longer needed, to avoid a **memory leak**.

### Memory Management:
- **Dynamically allocated objects** (using `new`) are stored in the heap, and their lifetime is controlled manually by the programmer. You must delete them when done, otherwise, you’ll have a memory leak.
- If you fail to call `delete` on the returned pointer, that memory will remain reserved, and you will effectively lose the ability to reclaim it, resulting in a **memory leak**.
- If you call `delete` and then continue to use the pointer, you are at risk of **undefined behavior** because the memory could be overwritten or reallocated by other parts of your program (known as a **dangling pointer**).

### Stack vs. Heap Allocation:
- When an object is allocated **on the stack** (e.g., a local variable inside a function), its memory is automatically deallocated when the function goes out of scope.
- When allocated **on the heap** using `new`, the object persists until explicitly deallocated, regardless of function scope.

Thus, as long as you dynamically allocate an object with `new` and return a pointer to it, that memory will stay reserved until you explicitly free it with `delete`, irrespective of the scope of the function in which the allocation occurred. This makes dynamic allocation useful for returning pointers from functions, but it also requires careful memory management to avoid leaks or dangling pointers.

### Modern C++: Smart Pointers
In modern C++ (C++11 and beyond), it's advisable to use **smart pointers** like `std::unique_ptr` or `std::shared_ptr` to manage dynamic memory:

- **`std::unique_ptr`** automatically deletes the allocated object when it goes out of scope, ensuring no memory leaks.
- **`std::shared_ptr`** keeps track of how many pointers refer to the object and deletes it when the last reference is gone.

Using smart pointers helps you manage the lifetime of dynamically allocated objects in a more reliable way, reducing the risk of memory leaks and dangling pointers.

### Example with Smart Pointers:
```cpp
#include <memory>

std::unique_ptr<MyClass> createObject() {
    return std::make_unique<MyClass>(); // Return a smart pointer managing the heap memory
}

int main() {
    std::unique_ptr<MyClass> myObject = createObject();
    // Use myObject...

    // Memory is automatically deallocated when myObject goes out of scope
}
```
Here, `std::unique_ptr` ensures that the memory is automatically freed when `myObject` goes out of scope, reducing the risk of forgetting to `delete` manually.
