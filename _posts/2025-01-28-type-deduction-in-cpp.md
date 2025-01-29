---
layout: post
title: "Understanding Type Deductions in C++"
category: cpp-techniques
---

In C++, there are three kinds of type deductions:

1. **`template` type deduction**
2. **`auto` type deduction**
3. **`decltype` type deduction**

---

# Understanding `template` Type Deduction

Template type deduction is a fundamental concept in C++ that determines how the compiler deduces types for template parameters. The general form of a template function and its call is:

```cpp
template<typename T>
void func(ParamType param);

func(expr);
```

The type deduced for `T` depends on both `expr` and the form of `ParamType`:

1. **Pointer or Reference (`T&` or `T*`)**: The compiler ignores reference-ness in `expr` and deduces `T` based on the underlying type.
   - Example: If `expr` is `int&`, `T` is deduced as `int`.

2. **Universal Reference (`T&&`)**: 
   - If `expr` is an lvalue, `T` is deduced as an lvalue reference.
   - If `expr` is an rvalue, `T` is deduced as the type of the rvalue.

3. **Neither Pointer nor Reference (Pass-by-Value, `T`)**: The compiler ignores both reference-ness and const-ness in `expr`.
   - Example: If `expr` is `const int&`, `T` is deduced as `int`.

## Key Takeaways:
- `T` deduction depends on both `expr` and `ParamType`.
- References and const-ness are ignored in certain cases.
- Universal references behave differently based on lvalue/rvalue properties.

---

# Understanding `auto` Type Deduction

## `auto` Type Deduction is Similar to `template` Type Deduction

`auto` type deduction follows the same rules as template type deduction. `auto` plays the role of `T`, and the type specifier acts as `ParamType`.

### Example:

```cpp
auto x = 27; // T is int
const auto cx = x; // T is int, but type specifier is 'const auto'
auto cx2 = cx; // cx is const int, but T is int (const is ignored)
const auto& rx = x; // type specifier is 'const auto&', so T is int
```

The compiler internally treats these as:

```cpp
template<typename T>
void func_for_x(T param);

template<typename T>
void func_for_cx(const T param);

template<typename T>
void func_for_rx(const T& param);
```

### Three Cases of `auto` Type Deduction
1. **Pointer or Reference**
2. **Universal Reference**
3. **Neither a Pointer nor a Reference**

#### Examples:

```cpp
auto x = 27; // Case 3: T is int
const auto& rx = x; // Case 1: T is int

auto&& uref2 = cx; // cx is const int -> T is const int
auto&& uref3 = 27; // 27 is an r-value -> T is int
```

**Important:** Unless `ParamType` is a universal reference (`T&&`), the deduced type will never be a reference.

### Special Case: `std::initializer_list<T>`
Unlike template type deduction, `auto` can deduce `std::initializer_list<T>`:

```cpp
auto x = {1, 2, 3}; // x is deduced as std::initializer_list<int>
```

However, for templates:

```cpp
template <typename T>
void func(T param);

func({1, 2, 3}); // Error: template type deduction fails
```

Thus, `auto type deduction = template type deduction + brace initialization support.`

### `auto` in Function Return Type Deduction in C++ 11
When `auto` is used in function return types, it follows template type deduction rules, meaning **brace initialization is not supported**:

```cpp
auto func() { return {1, 2, 3}; } // Error: Cannot deduce initializer list
```

## Key Takeaways
Here are the key takeaways for `auto` type deduction:

1. **Follows Template Type Deduction Rules**  
   - `auto` behaves like a template type (`T`) in type deduction.

2. **Reference and `const` Handling**  
   - If `auto` is not explicitly a reference, `const` qualifiers are ignored.  
   - Example: `const auto cx = x;` → `T` is `int`, but `cx` is `const int`.  
   - Example: `auto cx2 = cx;` → `T` is `int` (const is dropped).

3. **Universal References with `auto&&`**  
   - If `auto&&` is used, type deduction follows reference collapsing rules:  
     - If bound to an lvalue, `T` is an lvalue reference.  
     - If bound to an rvalue, `T` is the underlying type.

4. **`std::initializer_list<T>` Special Case**  
   - `auto` can deduce `std::initializer_list<T>` from `{}` brace initialization, unlike templates.  
   - Example: `auto x = {1, 2, 3};` → `x` is `std::initializer_list<int>`.

5. **Function Return Type Deduction**  
   - `auto` in function return types follows template deduction rules.  
   - Brace initialization `{}` is **not** allowed.  
   - Use `decltype(auto)` when returning expressions to preserve references.

---

# Understanding `decltype` Type Deduction

## `decltype` is a Parrot

`decltype` returns the exact type of an expression without modification. This makes it useful in cases where precise type control is needed.

### Example:

```cpp
int x = 19;
decltype(x) a = x;  // Type: int
decltype((x)) b = x; // Type: int&
```

### Using `decltype` for Function Return Types

In templates, `decltype` helps determine return types when needed:

```cpp
template <typename Container, typename Index>
auto authAndAccess(Container& c, Index i) -> decltype(c[i]) {
    authenticateUser();
    return c[i];
}
```

### C++14 Enhancement

C++14 allows `auto` to deduce function return types, removing the need for explicit trailing return types:

```cpp
template <typename Container, typename Index>
auto authAndAccess(Container& c, Index i) {
    authenticateUser();
    return c[i];
}
```

However, without `decltype`, references can be dropped unexpectedly:

```cpp
authAndAccess(d, 5) = 10; // Problem: return type might lose reference
```

Using `decltype(auto)` ensures correctness:

```cpp
template <typename Container, typename Index>
decltype(auto) authAndAccess(Container& c, Index i) {
    authenticateUser();
    return c[i];
}
```

### Final C++14 Universal Reference Solution:

```cpp
template <typename Container, typename Index>
decltype(auto) authAndAccess(Container&& c, Index i) {
    authenticateUser();
    return std::forward<Container>(c[i]);
}
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