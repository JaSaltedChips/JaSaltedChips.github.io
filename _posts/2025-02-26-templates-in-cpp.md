---
layout: post
title: "Templates and their instantiation in C++"
category: cpp-techniques
---

# Table of Contents
- [Introduction](#introduction)
- [Why would this not lead to multiple definition linker error?](#why-would-this-not-lead-to-multiple-definition-linker-error)
- [How Do Linkers Merge Identical Instantiations?](#how-do-linkers-merge-identical-instantiations)
- [What Happens with Template Specialization?](#what-happens-with-template-specialization)
- [Now, Let's Talk About Partial Specialization](#now-lets-talk-about-partial-specialization)

### **Introduction**
In C++, when defining a function for a **template class**, it must be done in the header file (or an included file) rather than a separate `.cpp` file. This is due to **how templates are instantiated and compiled**. Here’s why:

1. **Templates Are Not Compiled Until Instantiated**
   - A **template is not an actual class or function**; it’s a blueprint for the compiler to generate code when a specific type is used.
   - Unlike regular functions or classes, template code is **only compiled when it is instantiated** (i.e., when it is used with a specific type).

2. **Header Files Ensure Visibility**
   - When you define a template function inside the header, let's say `MyTemplate.h`, it **remains visible to all translation units** that include the header.
   - This way, when the template is instantiated in another `.cpp` file, let's say `file1.cpp`, the compiler has access to its definition.

3. **Separate Compilation Problem**
   - If you put a template class's function definitions in a `.cpp` file, i.e., `MyTemplate.cpp`, the compiler won’t generate any code for it unless it sees an explicit instantiation in the same translation unit i.e., `template.o`
   - Since templates are instantiated when they are used, **separate compilation doesn’t work**—the compiler won’t know which types it needs to instantiate when it compiles the `MyTemplate.cpp` file.

4. **Possible Workarounds (Explicit Instantiations)**
   - A workaround to define template functions in a `MyTemplate.cpp` file is to use **explicit instantiations**:
     ```cpp
     // MyTemplate.cpp
     #include "MyTemplate.h"

     template class MyTemplate<int>;  // Explicit instantiation
     ```
   - However, this approach forces you to **predefine** all the template instantiations you will use, which limits flexibility.

#### **Conclusion**
Because templates are **not actual code until they are instantiated**, and because **separate compilation doesn't work well with templates** (you need to **predefine** all the template instantiations you will use), their function definitions must be kept in the header file. This ensures that the compiler can instantiate the template correctly whenever it encounters a use of the class with a specific type.

---
### **Why would this not lead to multiple definition linker error?**

Normally, having function definitions in a header file would lead to **multiple definition errors** if the header is included in multiple `.cpp` files, i.e., `file1.cpp` and `file2.cpp`, and then these two files are linked together to form a binary. However, **templates work differently** due to the way they are instantiated. Here's why **template definitions in headers do not cause multiple definition errors**:

1. **Templates Are Instantiated Only When Needed**
   - A **template is not compiled into actual code** until it is instantiated with a specific type.
   - Even though the template function definition is included in multiple translation units (because it's in a header file), **each translation unit only instantiates the template for the types it uses**.

2. **The One Definition Rule (ODR) and Templates**
   - The **One Definition Rule (ODR)** states that:
     1. Each translation unit can see the same definition of a template.
     2. As long as the definition is **identical in all translation units**, the linker will merge duplicate instantiations.
   - Since the template function is only **instantiated when used**, and the compiler ensures that each type instantiation happens only once per translation unit, and the linker, with the help from compiler, has the capability to merge duplicate instantiations, this does not lead to multiple definition errors.

3. **Inline Definition as an Implicit Solution**
   - Functions defined in a **class template** are automatically considered **inline**, which means: The compiler allows multiple copies of **identical** inline functions across translation units.
     ```cpp
     template <typename T>
     class MyTemplate {
     public:
         void foo() { /* definition in header */ }
     };
     ```
But linker, together with compiler, ensures they are merged at link time and there is only one definition.

4. **Explicit Instantiations Can Cause Multiple Definitions**
   - If you use **explicit template instantiations** in multiple `.cpp` files:
     ```cpp
     // If this appears in multiple .cpp files → multiple definition error
     template class MyTemplate<int>; 
     ```
     The explicitly instantiated objects (or functions) are no longer templates anymore. This will make the compiler lose the ability to pass on the information necessary for the linker to later merge the definitions into one. You will read more about this in the next [section](#how-do-linkers-merge-identical-instantiations). However, the takeaway is: the linker will complain because **the explicit instantiation forces the compiler to generate code in multiple translation units**.

5. **How to Avoid Multiple Definition Errors with Explicit Instantiations**
   - Move explicit instantiations to a **single** `.cpp` file:
     ```cpp
     // MyTemplate.cpp
     #include "MyTemplate.h"

     template class MyTemplate<int>;  // Only defined here
     ```
   - Other `.cpp` files can include `MyTemplate.h`, but they won't instantiate `MyTemplate<int>` because it's already done in `MyTemplate.cpp`. You just need to ensure that this translation unit `MyTemplate.o` is made available for linking. You can do this in your `CMakeLists.txt`, no problem.

#### **Conclusion**
Templates avoid multiple definition errors because they are **instantiated only when needed**, and the compiler ensures identical definitions across translation units are merged. However, if you use **explicit template instantiations** in multiple `.cpp` files, you **must** define them in only one place to avoid linker errors.

---
### **How Do Linkers Merge Identical Instantiations?**

The ability of linkers to merge identical instantiations is based on a technique called **"weak symbols"** and **"one-definition rule (ODR) merging."** The merging happens due to the **weak symbol mechanism** in many linkers, particularly in ELF-based (Linux), Mach-O (macOS), and COFF (Windows) object file formats.

**Step 1: Compilation – Each Translation Unit Generates Template Instantiations**
- When a template is used in multiple `.cpp` files, each translation unit (TU) **sees the template definition** and **generates its own copy** of the instantiated function/class.
- This means if `MyTemplate<int>` is used in `file1.cpp` and `file2.cpp`, **both object files will have their own compiled version of `MyTemplate<int>::foo()`**.

**Step 2: Linking – Weak Symbols and ODR Merging**

During linking, the **linker sees multiple identical definitions** of `MyTemplate<int>::foo()` across different object files.

1. **Weak Symbols (in ELF & Mach-O formats)**  
   - The compiler marks **instantiated template functions** as **weak symbols**.
   - Weak symbols **do not cause multiple definition errors** unless there are conflicting versions.
   ```cpp
   // file1.cpp
   template <>
   void MyTemplate<int>::foo() {
       std::cout << "Version 1\n";
   }
   ```
   ```cpp
   // file2.cpp
   template <>
   void MyTemplate<int>::foo() {
       std::cout << "Version 2\n";
   }
   ```
   - The linker keeps only **one copy** of `MyTemplate<int>::foo()` in the final binary.

2. **One Definition Rule (ODR) Merging**  
   - According to C++’s **One Definition Rule (ODR)**, multiple identical definitions of a function/class are **allowed** as long as they are **exactly the same**.
   - The linker checks whether the multiple definitions are **identical** and **merges them** into one.

**Step 3: Eliminating Duplicates**
- If `file1.o` and `file2.o` both contain the same symbol for `MyTemplate<int>::foo()`, the linker selects **one of them** and discards the others.
- This ensures that there is **only one definition** of `MyTemplate<int>::foo()` in the final executable.

**Example: Weak Symbols in Action**

If you compile a template-heavy project and inspect the object files with `nm` (on Linux/macOS), you may see symbols marked with a **"weak" (`W`)** status:

```sh
nm file1.o | grep foo
0000000000000000 W _ZN9MyTemplateIiE3fooEv
```

The `W` means that the function is a **weak symbol**, which the linker can safely discard if another identical definition exists.

#### **Conclusion**
- If the instantiations are **identical**, the linker **merges** them and keeps only one.
- If the instantiations **differ** (e.g., different specializations in different `.cpp` files), the linker **detects a conflict** and throws an error.

**What If a Platform Does Not Support Weak Symbols?**

Some platforms (older Windows toolchains, for example) do not support weak symbols well. In such cases:
- The compiler **may not generate template instantiations** in every translation unit.
- Instead, **explicit instantiations** in a `.cpp` file are recommended.

Example:

```cpp
// MyTemplate.cpp (only here we explicitly instantiate it)
template class MyTemplate<int>;
```

This ensures that **only one translation unit** defines `MyTemplate<int>`, avoiding multiple definitions.

**Summary**
1. **Templates are instantiated in each translation unit that uses them**.
2. **The compiler marks these instantiations as weak symbols**, meaning the linker can discard duplicates.
3. **The linker merges identical definitions**, ensuring only one exists in the final binary.
4. **If different versions exist, conflicting defintions, the linker throws a multiple definition error**.

---
### **What Happens with Template Specialization?**
Unlike regular templates, **explicit specializations** behave differently because **their definitions are not templated anymore**. This means they are **no longer implicitly inline**, which makes multiple definitions across translation units a problem.

#### **Example of Explicit Specialization:**
```cpp
// MyTemplate.h
template <typename T>
class MyTemplate {
public:
    void foo();  // Declaration
};

// Primary template definition
template <typename T>
void MyTemplate<T>::foo() {
    // Generic implementation
}

// Specialization for int
template <>
class MyTemplate<int> {  // Specialized class
public:
    void foo();  // Declaration
};

// Specialized function definition
template <>
void MyTemplate<int>::foo() {
    // Special implementation for int
}
```

**Why Does This Cause Multiple Definition Errors?**
- If `MyTemplate<int>::foo()` is **fully defined in the header**, every `.cpp` file that includes `MyTemplate.h` will have its own copy of the function.
- Since **specializations are not templated anymore**, the compiler treats them like normal functions/classes, leading to **multiple definition errors at link time**.

**How to Avoid Multiple Definition Errors in Specializations?**

To prevent this, **explicit specializations should be declared in the header but defined in a single `.cpp` file**.

#### **Correct Approach:**
1. **Declare the specialization in the header file (`MyTemplate.h`)**:
    ```cpp
    // MyTemplate.h
    template <typename T>
    class MyTemplate {
    public:
    void foo();
    };

    // Primary template function definition (should stay in the header)
    template <typename T>
    void MyTemplate<T>::foo() {
        // Generic implementation
    }

    // Specialization declaration only
    template <>
    class MyTemplate<int> {
    public:
        void foo();
    };
    ```

2. **Define the specialization in a single `.cpp` file (`MyTemplate.cpp`)**:
   ```cpp
    // MyTemplate.cpp
    #include "MyTemplate.h"

    // Define the specialization (only in this .cpp file!)
    template <>
    void MyTemplate<int>::foo() {
        // Specialized implementation for int
    }
    ```

#### **Conclusion**
For **template specializations**, the function definitions are **not templated anymore**, so **they should be defined in a single `.cpp` file** to avoid multiple definition linker errors. Note that, **the primary template remains available for all types**, but the specialization has its definition in only one translation unit, avoiding multiple definition errors.

---
### **Now, Let's Talk About Partial Specialization**

**Comparison: Full Specialization vs. Partial Specialization**  

| Feature                  | **Full Specialization**                                      | **Partial Specialization**                                 |
|--------------------------|------------------------------------------------|------------------------------------------------|
| **Definition Scope**     | Must be defined **separately** from the primary template. | Defined as a **modified version** of the primary template. |
| **Applies To**           | A **specific type** (e.g., `MyTemplate<int>`) | A **subset of types** (e.g., `MyTemplate<T, int>`) |
| **Need for `.cpp` File?** | Yes, to avoid multiple definition errors. | No, since it's still a template and must be available in the header. |
| **Template Parameters**  | No longer uses `template <typename T>` (because it is fully specialized). | Still has `template <typename T>` but modifies one or more parameters. |
| **Flexibility**          | Works only for **one exact type**. | Works for **a range of types with some constraints**. |
| **Instantiation**        | Not instantiated from the primary template. | Instantiated from the primary template with modifications. |

---

**Examples of Both Specializations**
- **Full Specialization (Only for `int`)**
A full specialization **replaces** the primary template for a specific type.

    ```cpp
    // MyTemplate.h
    template <typename T>
    class MyTemplate {
    public:
        void foo();
    };

    // Primary template definition (remains in the header)
    template <typename T>
    void MyTemplate<T>::foo() {
        // Generic implementation
    }

    // Specialization declaration (must be in the header)
    template <>
    class MyTemplate<int> {
    public:
        void foo();
    };
    ```
    ```cpp
    // MyTemplate.cpp
    #include "MyTemplate.h"

    // Specialized definition (must be in a .cpp file)
    template <>
    void MyTemplate<int>::foo() {
        // Specialized behavior for int
    }
    ```

    - The specialization **completely overrides** the primary template for `int`.  
    - The definition of `MyTemplate<int>::foo()` must be in a **single `.cpp` file** to avoid multiple definition errors.

- **Partial Specialization (For Some Types, but Not All)**
A partial specialization **modifies** some behavior while keeping the primary template valid for other types.

    ```cpp
    // MyTemplate.h
    template <typename T1, typename T2>
    class MyTemplate {
    public:
    void foo();
    };

    // Primary template definition (remains in the header)
    template <typename T1, typename T2>
    void MyTemplate<T1, T2>::foo() {
        // Generic implementation
    }

    // Partial specialization for when T2 is int
    template <typename T1>
    class MyTemplate<T1, int> {
    public:
        void foo();
    };

    // Partial specialization function definition (remains in the header)
    template <typename T1>
    void MyTemplate<T1, int>::foo() {
        // Specialized behavior when second type is int
    }
    ```

    - This specialization applies when **the second template parameter is `int`**, but the first one can be anything (`T1`).
    - Since it is **still a template**, the definition must remain in the **header file** (no multiple definition issues).

#### **Key Differences in Usage**

| Case | Which Specialization Applies? |
|------|------------------------------|
| `MyTemplate<double, double> obj;` | Uses **primary template**. |
| `MyTemplate<char, int> obj;` | Uses **partial specialization** (because `T2 = int`). |
| `MyTemplate<int> obj;` | Uses **full specialization** (if defined for `int`). |


#### **Conclusion**
1. **Full Specialization**  
   - Replaces the primary template **completely** for a specific type.  
   - Needs to be **defined in a `.cpp` file** to prevent multiple definitions.  

2. **Partial Specialization**  
   - Only **modifies some template parameters** while keeping others generic.  
   - Still a template, so it **must be in the header file**.  

---
---

