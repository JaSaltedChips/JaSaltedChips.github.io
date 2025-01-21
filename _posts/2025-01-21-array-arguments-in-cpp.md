---
layout: post
title: "Array Arguments in C++"
category: cpp-techniques
---

The goal of this post is to understand how arrays behave as function arguments in C++. In C++, functions cannot have parameters that are true arrays because **arrays decay into pointers** when passed to functions. Read on for a detailed explanation and the implications of this behavior.

#### **Arrays Decay to Pointers**
When you pass an array as a parameter to a function in C++, the array automatically "decays" into a pointer to its first element. This means:
- The function does not receive the actual array.
- The function receives a pointer instead, which points to the first element of the array.
- The size information of the array is lost.

For example:

```cpp
void func(int arr[]) {  // This is equivalent to void func(int* arr)
    std::cout << sizeof(arr) << std::endl;  // Size of a pointer, not the array
}

int myArray[5] = {1, 2, 3, 4, 5};
func(myArray);
```

Here, `arr` inside the function is a pointer (`int*`), not the actual array. As a result:
- `sizeof(arr)` returns the size of the pointer (e.g., 8 bytes on a 64-bit system), not the size of the array.
- The compiler treats `void func(int arr[])`, `void func(int arr[10])`, and `void func(int* arr)` as identical.

#### **Workaround: References to Arrays**
If you need a function to work with an actual array (preserving its size and type), you can use a **reference to an array**. This bypasses the "decay to pointer" behavior.

For example:

```cpp
void func(int (&arr)[5]) {  // Reference to an array of size 5
    std::cout << sizeof(arr) << std::endl;  // Correctly returns size of the array
}

int myArray[5] = {1, 2, 3, 4, 5};
func(myArray);  // Works, and enforces that the array has exactly 5 elements
```

Key points:
- The reference preserves the array type and size (`int[5]` in this case).
- The compiler ensures that only arrays of size 5 can be passed to the function.
- `sizeof(arr)` correctly gives the size of the entire array (e.g., `5 * sizeof(int)`).

#### **Why Does C++ Use This Design?**
- **Efficiency**: Passing a pointer (usually 8 bytes on a 64-bit system) is much faster than copying the entire array.
- **Backward Compatibility**: This behavior is inherited from C, where arrays also decay to pointers.

#### **Summary**
- Functions in C++ **cannot declare parameters that are truly arrays** because arrays decay into pointers when passed to functions.
- If you want to preserve the size and type of the array, you need to use a **reference to an array**.

This distinction is an important part of C++'s design and a common source of confusion for beginners.

#### **References**
- Scott Meyers, *Effective Modern C++*, Item 1: "Understand template type deduction".