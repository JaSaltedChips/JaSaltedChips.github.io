---
layout: post
title: "Delete in C++"
category: cpp-techniques
---

In C++, `= delete` is a special syntax introduced in C++11 that explicitly marks a function as **deleted**, meaning it cannot be used. This is typically used to prevent certain operations, such as copying or moving an object, or to enforce specific coding constraints.

### **Common Use Cases**

1. **Prevent Copy and Move Operations**
   ```cpp
   class Example {
   public:
       Example() = default;  // Allow default constructor

       // Prevent copying
       Example(const Example&) = delete;
       Example& operator=(const Example&) = delete;

       // Prevent moving
       Example(Example&&) = delete;
       Example& operator=(Example&&) = delete;
   };

   int main() {
       Example e1;
       Example e2 = e1;  // Error: Copy constructor is deleted
   }
   ```

2. **Prevent Use of Certain Overloads**
   ```cpp
   struct NoBool {
       explicit NoBool(int) {}
       operator bool() = delete;  // Prevent implicit conversion to bool
   };

   int main() {
       NoBool obj(42);
       if (obj) { }  // Error: bool conversion is deleted
   }
   ```

### **Why Use `= delete`?**
- Provides **explicit** error messages instead of relying on private or undefined methods.
- Improves code clarity by clearly expressing design intent.
- Helps catch mistakes at compile time.

---

So the functionality provided by `= delete` can often be achieved by other means in C++. However, `= delete` offers a more **explicit, modern, and compiler-enforced** way to restrict certain operations. Here’s how similar goals were achieved without `= delete`:

1. **Prevent Copy/Move Without `= delete`**
    Before C++11, a common technique was to **declare** a function in the class but not provide a definition:

    ```cpp
    class Example {
    private:
        Example(const Example&);  // Declared but not defined
        Example& operator=(const Example&);  // Declared but not defined

    public:
        Example() {}  // Default constructor
    };
    ```

    - This works because attempting to use the deleted function results in a **linker error** instead of a compiler error.
    - The downside is that the intent is not immediately clear, and the error messages can be confusing.

2. **Using `private` to Restrict Copying**
    Another pre-C++11 trick was to declare the functions as `private`:
    ```cpp
    class Example {
    private:
        Example(const Example&) {}  // Prevent copying
        Example& operator=(const Example&) { return *this; }

    public:
        Example() {}  
    };
    ```
    - This prevents **external** code from calling these functions.
    - However, it does **not** prevent usage inside the class itself.

---

### **Why `= delete` Is Better**
- **Clearer intent**: The code explicitly tells the reader and compiler that the function should not exist.
- **Stronger compile-time checking**: Results in a **compiler error** instead of a **linker error** (like the older approaches).
- **Prevents accidental internal use**: Even within the class itself.

While `= delete` isn't *absolutely* necessary, it is the **best practice** in modern C++ when you want to prevent certain operations.
