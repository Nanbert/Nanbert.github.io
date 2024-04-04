---
title: C++_Template
date: 2024-03-24 03:59:20
tags:
index_img: /images/template.jpg
banner_img: /images/template.jpg
---

# type_traits
- `#include<type_traits>`
- std::is_integral<T>::value 等价于 std::is_integral_v<T>

|函数|说明|
|:-:|:-:|
|is_integral|checks for an integral type(bool,char,unsigned char,short,int,long,etc)|
|is_floating_point|checks for a floating-point type(float,double)|
|is_arithmetic|checks for a integral or floating-point type|
|is_signed|checks for a signed type|
|is_unsigned|checks for an unsigned type|
|is_enum|checks for an enumerator type(enum,enum class)|
|is_void|checks for(void)|
|is_pointer|checks for a pointer(`T*`)|
|is_null_pointer||
|is_reference|checks for a reference(T&)|
|is_array|checks for an array(`T (&)[N]`)|
|is_function||
|is_class|checks for a class type(struct class)|
|is_abstract|checks for a class with at least one pure virtual function|
|is_polymorphic|checks for a class with at least one virtual function|
|is_const||
|is_same<T,R>|checks if T and R are the same type|
|is_base_of<T,R>|checks if T is base of R|
|is_convertible<T,R>|checks if T can be converted to R|

- using U = typename std::make_unsigned<int>::type 等价于 std::make_unsigned_t<T>;

|函数|说明|
|:-:|:-:|
|make_signed|make a signed type|
|make_unsigned|makes an unsigned type|
|remove_pointer|remove pointer(`T*->T`)|
|add_pointer|add pointer(`T->T*`)|
|remove_reference|remove reference(`T*->T`)|
|add_lvalue_reference|add reference(T->T&)|
|remove_const|remove const|
|add_const|add const|
|common_type<T,R>|(see the example)|
|conditional<pred,T,R>|returns T if pred is true,R otherwise|
|decay<T>|returns the same type as a function parameter passed by-value|

```C++
# include <type_traits>
template<typename T, typename R>
std::common_type_t<R, T> // <-- return type
add(T a, R b) {
return a + b;
}
// we can also use decltype to derive the result type
using result_t = decltype(add(3, 4.0f));
result_t x = add(3, 4.0f);
```

- 如果 T 是一个引用类型（如 int&），std::decay<T>::type 就是 int。
- 如果 T 是一个数组类型（如 int[5]），std::decay<T>::type 就是 int*。
- 如果 T 是一个函数类型，std::decay<T>::type 就是相应的函数指针类型。
- 如果 T 是一个 cv-qualified 类型（如 const int 或 volatile int），std::decay<T>::type 就是没有这些限定符的 int。

# Variadic Template
- 可以用`sizeof...(args)`得出个数
```C++
// BASE CASE
template<typename T, typename R>
auto add(T a, R b) {
return a + b;
}
// RECURSIVE CASE
template<typename T, typename... TArgs> // Variadic typename
auto add(T a, TArgs... args) { // Typename expansion
return a + add(args...); // Arguments expansion
}
add(2, 3.0); // 5
add(2, 3.0, 4); // 9
add(2, 3.0, 4, 5); // 14
// add(2); // compile error the base case accepts only two arguments
```

# Concepts


