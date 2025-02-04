---
layout: post
title: "Pointer vs Array Pointer in C++"
category: cpp-techniques
---

In C++, there is a key distinction between:

1. A **pointer to the first element of an array** (`T*`).
2. A **pointer to the whole array** (`T(*)[N]`).

### 1. Pointer to the First Element (`T*`)
A pointer to the first element of an array is a simple pointer to the type of the elements in the array.

```cpp
int arr[5] = {1, 2, 3, 4, 5};
int* p1 = arr;  // Equivalent to &arr[0]
```

- `arr` decays to a pointer to its first element (`int*`).
- `p1` can be incremented (`p1 + 1` moves to `arr[1]`).
- It does not "know" the size of the array.

### 2. Pointer to the Whole Array (`T(*)[N]`)
A pointer to the whole array preserves the size information.

```cpp
int (*p2)[5] = &arr;  // Pointer to the whole array
```

- `p2` is of type `int (*)[5]`, meaning it points to an array of 5 integers.
- `p2 + 1` moves past the entire array, not just one element.
- Useful when you want to pass the entire array to a function while keeping its size.

### **Key Differences**
| Feature              | `T*` (Pointer to First Element) | `T(*)[N]` (Pointer to Whole Array) |
|----------------------|---------------------------------|----------------------------------|
| Type                | `int*`                          | `int (*)[N]`                   |
| Points To           | First element (`arr[0]`)       | Entire array (`arr`)           |
| Pointer Arithmetic  | `p + 1` moves to next element  | `p + 1` moves past whole array |
| Size Awareness      | No size information            | Knows full array size          |
| Usage Example      | Iterating elements              | Passing the entire array       |

### **Example to Illustrate the Difference**
```cpp
#include <iostream>

void printFirstElement(int* p) {
    std::cout << "First element: " << *p << std::endl;
}

void printArraySize(int (*p)[5]) {
    std::cout << "Size of array: " << sizeof(*p) << std::endl;
}

int main() {
    int arr[5] = {1, 2, 3, 4, 5};
    
    printFirstElement(arr);   // Works: `arr` decays to `int*`
    printArraySize(&arr);     // Works: `&arr` gives `int(*)[5]`
    
    return 0;
}
```

### **Output**:
```
First element: 1
Size of array: 20  // (5 * sizeof(int))
```

### **Summary**
- Use `T*` when you need to work with individual elements.
- Use `T(*)[N]` when you need to ensure the function knows the full array's size.

---

Let's analyze the code carefully.

### **Breaking Down the Typedefs**
```cpp
typedef slow uint8[10];   // `slow` is now an alias for `uint8[10]`
typedef medium uint8[20]; // `medium` is now an alias for `uint8[20]`
typedef fast uint8[30];   // `fast` is now an alias for `uint8[30]`
```
- This means `slow` is `uint8[10]`, `medium` is `uint8[20]`, and `fast` is `uint8[30]`.
- These are array types, not pointers.

### **Structure Definition**
```cpp
typedef struct
{
    slow* a;
    medium* b;
    fast* c;
} ProcMem;
```
- Here, `a`, `b`, and `c` are **pointers to `slow`, `medium`, and `fast`**, respectively.
- Since `slow` is `uint8[10]`, `medium` is `uint8[20]`, and `fast` is `uint8[30]`, the actual pointer types become:

  - `slow* a` → `uint8 (*)[10]` (Pointer to an array of 10 `uint8`)
  - `medium* b` → `uint8 (*)[20]` (Pointer to an array of 20 `uint8`)
  - `fast* c` → `uint8 (*)[30]` (Pointer to an array of 30 `uint8`)

### **Answer**
- `a`, `b`, and `c` are **pointers to whole arrays**, not pointers to the first element.
- Their types are `uint8 (*)[10]`, `uint8 (*)[20]`, and `uint8 (*)[30]`.

### **Example Usage**
```cpp
#include <iostream>

typedef unsigned char uint8;
typedef uint8 slow[10];
typedef uint8 medium[20];
typedef uint8 fast[30];

typedef struct
{
    slow* a;   // Pointer to an array of 10 uint8
    medium* b; // Pointer to an array of 20 uint8
    fast* c;   // Pointer to an array of 30 uint8
} ProcMem;

int main()
{
    slow s = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    medium m = {11, 12, 13, 14, 15, 16, 17, 18, 19, 20};
    fast f = {21, 22, 23, 24, 25, 26, 27, 28, 29, 30};

    ProcMem mem;
    mem.a = &s; // Assigning address of whole array
    mem.b = &m;
    mem.c = &f;

    std::cout << "First element of a: " << (int)(*mem.a)[0] << std::endl;
    std::cout << "First element of b: " << (int)(*mem.b)[0] << std::endl;
    std::cout << "First element of c: " << (int)(*mem.c)[0] << std::endl;

    return 0;
}
```

### **Key Takeaways**
1. `a`, `b`, and `c` are **pointers to whole arrays** (`uint8 (*)[10]`, `uint8 (*)[20]`, `uint8 (*)[30]`).
2. They store the address of an entire array rather than just a single element.
3. `(*a)[0]` accesses the first element of the array that `a` points to.

---

Yes, there **is** a key advantage of using pointers to whole arrays (`T (*)[N]`) instead of pointers to the first element (`T*`). Here’s the main difference:

---

### **Advantage: Preserving Array Boundaries and Size Information**
When you use a **pointer to a whole array** (`T (*)[N]`), the pointer retains the array's structure and knows its full size. In contrast, a **pointer to the first element** (`T*`) loses this information and treats the array as just a sequence of elements.

#### **Key Differences**
| Feature               | `T*` (Pointer to First Element) | `T (*)[N]` (Pointer to Whole Array) |
|----------------------|---------------------------------|----------------------------------|
| What it points to   | First element (`&arr[0]`)      | Entire array (`&arr`)          |
| Pointer Arithmetic  | Moves by `sizeof(T)`           | Moves by `sizeof(T[N])`        |
| Array Size Awareness | **Lost** (acts like a generic pointer) | **Preserved** (knows `N`) |

---

### **Example: Difference in Pointer Arithmetic**
Consider the two versions below:

#### **Using `T*` (Pointer to First Element)**
```cpp
#include <iostream>

typedef int Array[5];

void printPointerArithmetic(int* p) {
    std::cout << "p + 1 points to: " << *(p + 1) << std::endl; // Moves by sizeof(int)
}

int main() {
    Array arr = {10, 20, 30, 40, 50};
    printPointerArithmetic(arr); // `arr` decays to `int*`
    
    return 0;
}
```
🔹 **Output:**
```
p + 1 points to: 20
```
Here, `p + 1` moves to the **next element** (`arr[1]`), stepping by `sizeof(int)`.

---

#### **Using `T (*)[N]` (Pointer to Whole Array)**
```cpp
#include <iostream>

typedef int Array[5];

void printPointerArithmetic(Array* p) {
    std::cout << "(*p)[1] is: " << (*p)[1] << std::endl;
    std::cout << "p + 1 moves by entire array size, not element size!" << std::endl;
}

int main() {
    Array arr = {10, 20, 30, 40, 50};
    printPointerArithmetic(&arr); // Passing pointer to whole array
    
    return 0;
}
```
🔹 **Output:**
```
(*p)[1] is: 20
p + 1 moves by entire array size, not element size!
```
Here, `p + 1` moves by `sizeof(int[5])`, **not just `sizeof(int)`**. This behavior prevents accidental element-wise pointer arithmetic.

---

### **When is `T (*)[N]` Useful?**
1. **Avoiding Accidental Out-of-Bounds Access**
   - A `T*` treats the array as a continuous sequence, so it can be mistakenly indexed beyond the array's size.
   - A `T (*)[N]` enforces the array boundary and ensures correct indexing.

2. **Preserving Multidimensional Array Structure**
   - In **multidimensional arrays**, using a pointer to the whole array avoids pointer decay issues.
   - Example:
     ```cpp
     void func(int (*arr)[4]) { /* Works with 2D arrays */ }
     int matrix[3][4];
     func(matrix); // Passes whole array correctly
     ```

3. **Maintaining Readability & Preventing Implicit Decay**
   - Using `T (*)[N]` explicitly signals that a full array is expected, improving code clarity.

---

### **When Should You Use `T*` Instead?**
1. If you only need **element-wise access** and don’t care about array size.
2. When working with dynamically allocated memory (`new T[N]`), where arrays are treated as simple pointer blocks.

---

### **Conclusion**
- Use **`T*` (pointer to first element)** if you need element-wise access and don’t care about array structure.
- Use **`T (*)[N]` (pointer to whole array)** when you want to **preserve size information**, prevent accidental pointer arithmetic errors, or work with **multidimensional arrays**.

---

```cpp
struct slow_copy
{
    uint8 a[10];
} slow_copy;

slow_copy mSlow;

void func(uint8 (*)[10] s);
```

### **Question**
How can I pass the address of my whole array `a[10]` to `func(..)`?

You can pass the address of `a[10]` in `mSlow` to `func(..)` like this:

```cpp
func(&mSlow.a);
```

### **Answer**
- `mSlow.a` is an array of 10 `uint8` (`uint8 a[10]`).
- The expression `&mSlow.a` gives a pointer to the entire array (`uint8 (*)[10]`), which matches the function parameter type `uint8 (*)[10]`.

---

### **Example Code**
```cpp
#include <iostream>

typedef unsigned char uint8;

struct slow_copy
{
    uint8 a[10];
} mSlow;

void func(uint8 (*s)[10]) {
    std::cout << "First element: " << (int)(*s)[0] << std::endl;
}

int main() {
    // Initialize array for demonstration
    for (int i = 0; i < 10; i++) {
        mSlow.a[i] = i;
    }

    // Pass the address of the whole array to func
    func(&mSlow.a);

    return 0;
}
```

### **Key Takeaways**
- `&mSlow.a` correctly gives a pointer to the whole array (`uint8 (*)[10]`).
- `func(&mSlow.a);` ensures that the function receives the **entire** array rather than decaying it to `uint8*`.
- Inside `func`, dereferencing `s` (`*s`) gives the original array, so `(*s)[0]` accesses the first element.

---
