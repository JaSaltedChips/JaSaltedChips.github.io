---
layout: post
title: "Understanding type deductions in C++"
category: cpp-techniques
---

Understanding **type deduction** in C++ leads to cleaner, simpler [^1], and more maintainable code. It improves **generic programming** and boosts efficiency with **universal references** and **move semantics**.

[^1]: Sometimes, it feels like trying to decode an ancient scroll written by a caffeinated wizard.

This blog breaks down **Items 1, 2, and 3** from *Effective Modern C++*, giving you the lowdown on type deduction. Think of it as a quick cheat-sheet to help you grasp the key ideas.
But hey, while this is a handy summary, nothing beats diving into the book itself for the full scoop straight from the legend, Scott Meyers. It’s worth it!

Alright, let’s dive right in!

<div style="display: flex; justify-content: space-between; align-items: center;">
  <div style="flex: 1; text-align: left; border-radius: 10px; padding: 10px;">
    <p>
      <strong>"Type inference and deduction is the ultimate in language design: letting the compiler do as much work as possible while retaining a clear and efficient syntax for the programmer." 
      <br>— Bjarne Stroustrup</strong>
    </p>
  </div>
  <div style="flex: 1; display: flex; justify-content: center; border-radius: 10px; overflow: hidden;">
    <img src="../../assets/images/lets_dive.png" alt="My Photo" style="width: 350px; border-radius: 50%;">
  </div>
</div>

<br>

---

In C++, there are three kinds of type deductions:

1. **`template` type deduction**
2. **`auto` type deduction**
3. **`decltype` type deduction**

## Understanding template type deduction

- `template` type deduction is a fundamental concept in C++ that determines how the compiler deduces types for template parameters. The general form of a template function and its call is:

   ```cpp
   template<typename T>
   void func(ParamType param);

   func(expr);
   ```

- The type deduced for `T` depends on both `expr` and the form of `ParamType`:

   1. **Pointer or Reference (`T&` or `T*`)**: The compiler ignores reference-ness in `expr` and deduces `T` based on the underlying type.
      - If `expr` is `int&`, `T` is deduced as `int`.

   2. **Universal Reference (`T&&`)**: 
      - If `expr` is an lvalue, `T` is deduced as an lvalue reference.
      - If `expr` is an rvalue, `T` is deduced as the type of the rvalue.

   3. **Neither Pointer nor Reference (Pass-by-Value, `T`)**: The compiler ignores both reference-ness and const-ness in `expr`. 
      - If `expr` is `const int&`, `T` is deduced as `int`.

   <!-- color can also be rgba format -> background-color: rgba(220, 201, 246, 0.8); -->
   <details style="background-color:rgba(172, 206, 230, 0.5); padding: 10px; border-radius: 5px; border: 1px solid #ddd;">
   <summary style="cursor: pointer; font-weight: bold;">Counter Intuitive (?)</summary>
   <p>Case 3 might sound counter-intuitive because in case 1, <code>ParamType</code> had a reference, which was ignored while deducing. Here, there is no reference, yet it is still ignored. The key takeaway is that you should not see this as a formula but rather apply the underlying idea:</p>
   <ul>
      <li><strong>In case 1,</strong>  although the type is deduced as <code>T</code>, the object itself remains a reference.</li>
      <li><strong>In case 3,</strong>  since the object is passed by value, keeping the reference makes no sense.</li>
      <li>The same applies to <code>const</code> or <code>volatile</code> qualifiers—they become irrelevant because the function works with a **copy**, making the state of the original object unimportant.</li>
   </ul>
   </details>
   <br>

   <div style="border: 1px solid #3498db; padding: 10px; background-color: #eaf5fc; border-radius: 5px;">
   <strong>Note:</strong>
   <ul>
      <li> Unless <code>ParamType</code> is a universal reference (<code>T&&</code>), the deduced type will never be a reference. </li>
      <li> And this deduced type reference will only be an l-value reference. </li>
   </ul>  
   </div>

### Key Takeaways:
- `T` deduction depends on both `expr` and `ParamType`.
- References and const-ness are ignored in certain cases (*Case 3*).
- Universal references behave differently based on lvalue/rvalue properties.

---

## Understanding auto type deduction

-  `auto` type deduction follows the same rules as template type deduction.

   ```cpp
   auto x = 27;            // T is int
   const auto cx = x;      // T is int (const is ignored), but type specifier is 'const auto'
   const auto& rx = x;     // type specifier is 'const auto&', so T is int
   ```

-  Internally, the compiler treats these as: 

   ```cpp
   template<typename T>
   void func_for_x(T param);

   template<typename T>
   void func_for_cx(const T param);

   template<typename T>
   void func_for_rx(const T& param);
   ```

-  There are also three cases of `auto` type deduction, like `template` type deduction

   ```cpp
   auto x = 27;         // Case 3: T is int
   const auto& rx = x;  // Case 1: T is int
   auto&& uref1 = x     // x is int -> T is int&
   auto&& uref2 = cx;   // cx is const int -> T is const int&
   auto&& uref3 = 27;   // 27 is an r-value -> T is int&&
   ```

-  **Brace initialization is not supported in `template` type deduction**
   
   Consider the below example:
   ```cpp
   template <typename T>
   void func(T param);

   func({1, 2, 3}); // Error: template type deduction fails
   ```

-  **´auto´ in function return types (C++11)**  
   When `auto` is used in function return types, it follows **template type deduction rules**, meaning **brace initialization is not supported**:  

   ```cpp
   auto func()
   { 
      return {1, 2, 3}; // Error: Cannot deduce initializer list
   } 
   ```  

<div style="border: 1px solid #3498db; padding: 10px; background-color: #eaf5fc; border-radius: 5px;">
   <strong>Note:</strong>
   <p> Unlike template type deduction, auto type deduction <strong>can</strong> deduce std::initializer_list.</p>
   <p><strong> Why this behavior?</strong> It is simply a rule of the language.</p>
   <p> Thus, <strong>´auto´ type deduction = ´template´ type deduction + brace initialization support.</strong></p>
</div>

### Key Takeaways
Here are the key takeaways for `auto` type deduction:

1. **Follows template type deduction rules**  
   - `auto` behaves like a template type (`T`) in type deduction.

2. **`std::initializer_list<T>` special case**  
   - `auto` can deduce `std::initializer_list<T>` from `{}` brace initialization, unlike templates.  
   - Example: `auto x = {1, 2, 3};` → `x` is `std::initializer_list<int>`.

3. **Function return type deduction**  
   - `auto` in function return types follows template deduction rules.  
   - Brace initialization `{}` return is therefore **not** allowed.  

---

## Understanding decltype type deduction

-  **`decltype` is a Parrot:** `decltype` returns the exact type of an expression without modification. This makes it useful in cases where precise type control is needed.

-  In C++ 11, primary use of `decltype` is declaring function templates where the function's return type depends on the parameter type.

   ```cpp
   template <typename Container, typename Index>
   auto authAndAccess(Container& c, Index i) -> decltype(c[i]) {
      authenticateUser();
      return c[i];
   }
   ```
   The trailing return type has the advantage that the function's parameters can be used in the specification of return type. If we have the return type preceed the function name in the conventional way,
   `c` and `i` would not be available because they wound not have been declared yet.

-  **C++14 Enhancement**: C++14 allows `auto` to deduce function return types, removing the need for explicit trailing return types

   ```cpp
   template <typename Container, typename Index>
   auto authAndAccess(Container& c, Index i) {
      authenticateUser();
      return c[i];
   }
   ```

   However, without `decltype`, references can be dropped unexpectedly i.e., we have seen the example already `auto& rx = x;` `T` is deduced as `int`, while the type specifier is whole `auto&`:

   ```cpp
   authAndAccess(d, 5) = 10; // Problem: authAndAccess(d, 5) is r-value
   ```

   -  Using `decltype(auto)` ensures correctness:

      ```cpp
      template <typename Container, typename Index>
      decltype(auto) authAndAccess(Container& c, Index i) {
         authenticateUser();
         return c[i];
      }
      ```

      -  `auto` says the type needs to be deduced.
      -  `decltype` says `decltype` rule needs to be applied.

-  **Final C++14 Universal reference solution:**

```cpp
template <typename Container, typename Index>
decltype(auto) authAndAccess(Container&& c, Index i) { // Item 24
    authenticateUser();
    return std::forward<Container>(c[i]); // Item 25: admonition to apply std::forward to Universal references.
}
```

-  **Final C++11 Universal reference solution:**

```cpp
template <typename Container, typename Index>
auto authAndAccess(Container&& c, Index i) -> decltype(std::forward<Container>(c[i])) { // Item 24
    authenticateUser();
    return std::forward<Container>(c[i]); // Item 25: admonition to apply std::forward to Universal references.
}
```

### Pay attention!

```cpp
int x = 19;
decltype(x) a = x;  // Type: int
decltype((x)) b = x; // Type: int&
```
**Check this out**: `decltype((x))` sneakily deduces the type as `int&`. If you’re not paying attention, this little gremlin can send you on a wild debugging goose chase, wondering why your code is acting all kinds of weird.
It’s like one of those magic tricks—right there in front of you, but somehow still invisible. Poof!

Consider this program:
```cpp
decltype(auto) f()
{
   int x = 23;
   return (x); // returns reference to a local object!
}

int a = 0;
a = f();
```
This code, like Scott Meyers says, is a free ticket to hop on the *Undefined Behavior Express*—a train you absolutely, positively, under no circumstances, should ever board.

### How to know the type deduced?
Luckily, there’s more than one way to crack this nut. My go-to move? **Compiler Diagnostics**: let the compiler spill the beans for you. Just force an error in your code, and the compiler will (very helpfully) throw a tantrum in its error message, revealing the deduced type.
It’s like getting the answer straight from the horse’s mouth—no guesswork needed!

```cpp

template <typename T>   // declaration only
class TD;               // TD: Type Displayer

auto x = 23;
TD<decltype(x)> td; // this would elicit errors because class TD is not defined.
```

---

## Summary

- `template` type deduction and `auto` type deduction follow the same rules.
- `auto` can deduce `std::initializer_list<T>`, unlike templates.
- `decltype` preserves exact types and is useful for function return types.
- `decltype(auto)` ensures reference correctness when returning values.

---

## References
- Scott Meyers, *Effective Modern C++*, Chapter 1: "Deducing Types".

---