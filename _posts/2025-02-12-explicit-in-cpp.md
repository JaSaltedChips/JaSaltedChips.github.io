---
layout: post
title: "Explicit in C++"
category: cpp-techniques
---

The `explicit` keyword in C++ is used to **prevent implicit conversions and unintended object creations**. It is primarily applied to constructors and conversion operators.


- **Prevent Implicit Conversions**
    By default, C++ allows **implicit** conversions when a constructor takes a single argument:
    ```cpp
    class Example {
    public:
        Example(int x) { }  // Implicit conversion allowed
    };

    int main() {
        Example e1 = 42;  // Works! But might be unintended.
    }

    // - Here, `e1` is created from an `int` without explicitly calling the
    //  constructor.
    // - This can lead to **unexpected behavior**.

    /////////////////////////// Another Example //////////////////////////////////

    void foo(Example e) {
    // Function expects an Example object
    }

    int main() {
        foo(42);  // Allowed due to implicit conversion
    }

    // - `42` is **implicitly** converted to `Example`, which may be **undesired**.

    ```

    **Fix: Use explicit**

    ```cpp
    class Example {
    public:
        explicit Example(int x) { }
    };

    int main() {
        // Error: Explicit constructor prevents implicit conversion
        Example e1 = 42;  
        // OK: Direct initialization is allowed
        Example e2(42);   
    }

    // - Now, `Example e1 = 42;` causes a **compile-time error**.
    // - The constructor **must be explicitly called**.

    /////////////////////////// Another Example //////////////////////////////////

    void foo(Example e) { }

    int main() {
        foo(42);       // Error: No implicit conversion
        foo(Example(42));  // OK: Explicit conversion
    }

    // - Now, `foo(42);` is **not allowed**, forcing explicit object creation.

    ```

    **Prevent Implicit Type Conversions for Conversion Operators**
    The `explicit` keyword can also be used on **conversion operators**:
    ```cpp
    class Example {
    public:
        explicit operator bool() const { return true; }
    };

    int main() {
        Example e;
        if (e) { }  // Error: Cannot implicitly convert Example to bool
    }
    ```
    
    - Without `explicit`, `if (e)` would **implicitly convert** `Example` to `bool`.
    - Now, `if (e)` gives an **error**, and conversion must be explicit:
    
    ```cpp
    if (static_cast<bool>(e)) { }  // OK
    ```

**When to Use `explicit`**:
1. **Single-argument constructors** to prevent implicit conversions.
2. **Conversion operators** (`operator type()`) to avoid unintended type conversions.
3. **Factory functions** that return objects where implicit conversion should not occur.

## **Can the `explicit` be used with regular functions?**

No, the `explicit` keyword **cannot** be used with regular functions. However, in **C++20**, `explicit` can be used with **conversion operators in function templates** (not regular functions).  

### Why Can't `explicit` Be Used with Regular Functions?
- **Regular functions** are already **explicitly called** by name, so there is no implicit conversion issue.
- **Function overloading and templates** already provide control over argument conversion, making `explicit` unnecessary.
- It is specifically designed for **constructors** and **conversion operators** to prevent implicit conversions. 

---

### **Summary**
- `explicit` **prevents implicit conversions**, making code safer.
- It **forces direct initialization** (`Example obj(42);`) instead of implicit (`Example obj = 42;`).
- It can be used on **constructors** and **conversion operators**.