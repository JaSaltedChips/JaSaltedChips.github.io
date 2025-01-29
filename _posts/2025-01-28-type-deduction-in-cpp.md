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

In C++, template type deduction is a fundamental concept that determines how the compiler deduces the types for template parameters. The general form of a template function and its call is as follows:

```cpp
template<typename T>
void func(ParamType param);

func(expr);
```

Here, the type deduced for `T` depends not only on the type of the expression `expr` but also on the form of `ParamType`. Specifically, the deduction rules vary based on whether `ParamType` is:

1. **A reference or pointer type (`T&` or `T*`)**:
   - The compiler ignores reference-ness in `expr` (if any) and deduces `T` based on the underlying type.
   - For example, if `expr` is an `int&`, `T` is deduced as `int`.

2. **A universal reference (`T&&`)**:
   - If `expr` is an lvalue, `T` is deduced as an lvalue reference.
   - If `expr` is an rvalue, `T` is deduced as the type of the rvalue.

3. **Neither a reference nor a pointer (pass-by-value, `T`)**:
   - The compiler ignores both reference-ness and const-ness in `expr`.
   - For example, if `expr` is a `const int&`, `T` is deduced as `int`.

Understanding these rules is crucial for writing effective templates and avoiding unexpected behavior in type deduction.

## Key Takeaways:
- The type deduction for `T` depends on both the type of `expr` and the form of `ParamType`.
- The rules differ based on whether `ParamType` is a reference, a universal reference, or a value type.
- References and const-ness in `expr` are often ignored during deduction, depending on the form of `ParamType`.
```

---

# Understanding `auto` Type Deduction

## `auto` Type Deduction is `template` Type Deduction

`auto` type deduction follows the same rules as template type deduction. There is a direct mapping between `template` type deduction and `auto` type deduction.

### Template Type Deduction
Consider the following general function template:

```cpp
template <typename T>
void func (ParamType param);
```

And a general call:

```cpp
func(expr);
```

When calling `func`, the compiler uses `expr` to deduce types for `T` and `ParamType`.

### `auto` as a Template Equivalent
When a variable is declared using `auto`, `auto` plays the role of `T` in the template, and the type specifier for the variable acts as `ParamType`. This is easier to demonstrate than to describe:

```cpp
auto x = 27; // T is int
const auto cx = x; // type specifier is 'const auto', so T is int
auto cx2 = cx; // cx is const int, but T is int (const is ignored)
const auto& rx = x; // type specifier is 'const auto&', so T is int
```

To deduce types for `x`, `cx`, and `rx`, the compiler acts as if there were a template function for each declaration along with a call to that function with the corresponding initializing expression:

```cpp
template<typename T>
void func_for_x(T param);

template<typename T>
void func_for_cx(const T param);

template<typename T>
void func_for_rx(const T& param);
```

### Three Cases of `auto` Type Deduction
In a variable declaration using `auto`, the type specifier takes the place of `ParamType`. The deduction follows three cases:

1. **Pointer or Reference**
2. **Universal Reference**
3. **Neither a Pointer nor a Reference**

#### Example Cases
```cpp
auto x = 27;         // Case 3: T is int (neither pointer nor reference)
const auto& rx = x;  // Case 1: T is int (reference case)

auto&& uref2 = cx;   // ParamType is auto&&, cx is const int -> T is const int
auto&& uref3 = 27;   // ParamType is auto&&, 27 is an r-value -> T is int
```

**Important:** Unless `ParamType` is a universal reference (`T&&`), the deduced template type will never be a reference (i.e., an lvalue reference).

### `std::initializer_list<T>` Exception
One notable difference is that `auto` type deduction can deduce `std::initializer_list<T>`, whereas template type deduction cannot. This seems to be a special rule rather than a consequence of the general deduction mechanism.

### `auto` in Function Return Type Deduction
When `auto` is used in function return type deduction, it follows the same rules as template type deduction, meaning it does **not** deduce `std::initializer_list<T>`.

```cpp
auto func() { return {1, 2, 3}; } // Error: Cannot deduce type from initializer list


---

## 1. `template` and `auto` Type Deduction

`template` and `auto` deductions work in a similar manner, except for one key difference: `auto` deduction can differentiate between brace initialization and other forms of initialization.

### Example of Auto-Type Deduction:
```cpp
auto x = {1, 2, 3, 4}; // x is deduced as std::initializer_list<int>
```

However, for templates:
```cpp
template <typename T>
void func(T param);

func({1, 2, 3, 4}); // Error: template type deduction fails
```

Therefore, `auto type deduction = template type deduction + brace initialization support.`


### Using `auto` in Functions and Lambdas
In C++11, the `auto` keyword can also be used:
- For return types in functions
- In lambdas

However, the `auto` deduction in these cases uses only `template` type deduction, meaning **brace initialization is not supported**.

## 2. `decltype` Type Deduction

**`decltype`** is like a "parrot": it simply returns the exact type of the expression provided, without making any modifications. This makes it very straightforward, though a few edge cases can seem unintuitive.

### Primary Use of `decltype` in C++11
`decltype` is particularly useful in declaring function templates where the return type depends on the parameter type.

#### Example:
```cpp
template <typename Container, typename Index>
auto authAndAccess(Container& c, Index i) -> decltype(c[i]) {
    authenticateUser();
    return c[i];
}
```

**Explanation:**
- The trailing return type allows us to use function parameters (`c` and `i`) to specify the return type.
- If we were to declare the return type in the conventional way (before the function name), the parameters would not yet be available.

### C++14 Extension
In C++14, the return type of **all functions and lambdas** can be deduced, including those with multiple return statements.

#### Example:
```cpp
template <typename Container, typename Index>
auto authAndAccess(Container& c, Index i) {
    authenticateUser();
    return c[i];
}
```

### Issue Without `decltype`
If you omit `decltype` and rely on `auto`, template-type deduction drops references and type specifiers. This can lead to problems.

#### Problem Example:
```cpp
authAndAccess(d, 5) = 10; // Attempt to assign 10 to a container element
```
If `d` is a container that returns a reference, dropping the reference during type deduction would result in trying to assign to an r-value.

#### Fix:
```cpp
template <typename Container, typename Index>
decltype(auto) authAndAccess(Container& c, Index i) {
    authenticateUser();
    return c[i];
}
```
Here, `decltype(auto)` ensures the return type is deduced exactly as it appears in the return statement.

#### Example:
```cpp
Widget w;
const Widget& cw = w;

auto myWidget1 = cw;        // Type: Widget (reference dropped)
decltype(auto) myWidget2 = cw; // Type: const Widget&
```

### Final Solutions for All Cases

#### C++14:
```cpp
template <typename Container, typename Index>
decltype(auto) authAndAccess(Container&& c, Index i) { // Universal reference
    authenticateUser();
    return std::forward<Container>(c[i]); // Forwarding
}
```

#### C++11:
```cpp
template <typename Container, typename Index>
auto authAndAccess(Container&& c, Index i) -> decltype(std::forward<Container>(c[i])) {
    authenticateUser();
    return std::forward<Container>(c[i]);
}
```

### Key Edge Case
The difference between `decltype(x)` and `decltype((x))`:
- `decltype(x)` deduces the type of `x`.
- `decltype((x))` deduces `x` as a reference type.

#### Example:
```cpp
int x = 19;

decltype(x) a = x;  // Type: int
decltype((x)) b = x; // Type: int&
```

Returning a reference to a local variable results in **undefined behavior**:
```cpp
decltype(auto) function() {
    int x = 19;
    return (x); // Returns a reference to a local variable (undefined behavior)
}
```

---

This explanation ensures clarity and includes practical examples to illustrate the concepts effectively.


#### **References**
- Scott Meyers, *Effective Modern C++*, Item 1: "Understand template type deduction".
- Scott Meyers, *Effective Modern C++*, Item 1: "Understand auto type deduction".
- Scott Meyers, *Effective Modern C++*, Item 1: "Understand decltype type deduction".