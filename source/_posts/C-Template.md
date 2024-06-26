---
title: C++_Template
date: 2024-03-24 03:59:20
tags:
index_img: /images/template.jpg
banner_img: /images/template.jpg
---

# Class Template-Deduction Guide
## iterator
```C++
# include <type_traits> // std::remove_reference_t
# include <vector> // std::vector
template<typename T>
struct Container {
template<typename Iter>
Container(Iter beg, Iter end) {}
};
template<typename Iter>
Container(Iter b, Iter e) -> // deduction guide
Container<typename std::iterator_traits<Iter>::value_type>;
std::vector v{1, 2, 3};
Container c{v.begin(), v.end()}; // construct 'Container<int>'
```
## string
```C++
template<typename T>
struct MyString {
MyString(T) {}
};
// constructor class instantiation
MyString(char const*) -> MyString<std::string>; // deduction guide
MyString s{"abc"}; // construct 'MyString<std::string>'
```
# 类与成员函数
有两种方式：
- Generic class + generic function
- Full class specialization + generic/full specialization function
```C++
template<typename T, typename R>
struct A {
    template<typename X, typename Y>
    void f();
};
template<typename T, typename R>
template<typename X, typename Y>
void A<T, R>::f() {}
// ok, A<T, R> and f<X, Y> are not specialized
template<>
template<typename X, typename Y>
void A<int, int>::f() {}
// ok, A<int, int> is full specialized
// ok, f<X, Y> is not specialized
template<>
template<>
void A<int, int>::f<int, int>() {}
// ok, A<int, int> and f<int, int> are full specialized
template<typename T>
template<typename X, typename Y>
void A<T, int>::f() {}
// error A<T, int> is partially specialized
// (A<T, int> class must be defined before)
template<typename T, typename R>
template<typename X>
void A<T, R>::f<int, X>() {}
// error function members cannot be partially specialized
template<typename T, typename R>
template<>
void A<T, R>::f<int, int>() {}
// error function members of a non-specialized class cannot be specialized
// (requires a binding to a specific template instantiation at compile-time)
```
# enable_if
## 原理
```C++
template<bool Condition, typename T = void>
struct enable_if {
// "type" is not defined if "Condition == false"
};
template<typename T>
struct enable_if<true, T> {
    using type = T;
};
```
## array/pointer
```C++
# include <type_traits>
template<typename T, int Size>
void f(T (&array)[Size]) {} // (1)
//template<typename T, int Size>
//void f(T* array) {} // (2)
template<typename T>
std::enable_if_t<std::is_pointer_v<T>>
f(T ptr) {} // (3)
int array[3];
f(array); // It is not possible to call (1) if (2) is present
// The reason is that 'array' decays to a pointer
// Now with (3), the code calls (1)
```
## 检查类是否含有某个成员
```C++
struct A {
static int x;
int y;
using type = int;
};
struct B {};

# include <type_traits>
template<typename T, typename = void>
struct has_x : std::false_type {};
template<typename T>
struct has_x<T, decltype((void) T::x)> : std::true_type {};
template<typename T, typename = void>
struct has_y : std::false_type {};
template<typename T>
struct has_y<T, decltype((void) std::declval<T>().y)> : std::true_type {};
has_x< A >::value; // returns true
has_x< B >::value; // returns false
has_y< A >::value; // returns true
has_y< B >::value; // returns false
```
## 判断是否定义某个类型
```C++
template<typename...>
using void_t = void; // included in C++17 <utility>
template<typename T, typename = void>
struct has_type : std::false_type {};
template<typename T>
struct has_type<T,
std::void_t<typename T::type> > : std::true_type {};
has_type< A >::value; // returns true
has_type< B >::value; // returns false
```
## 判断是否支持流操作
```C++
template<typename T>
using EnableP = decltype( std::declval<std::ostream&>() <<
std::declval<T>() );
template<typename T, typename = void>
struct is_stream_supported : std::false_type {};
template<typename T>
struct is_stream_supported<T, EnableP<T>> : std::true_type {};
struct A {};
is_stream_supported<int>::value; // returns true
is_stream_supported<A>::value; // returns false
```
# 模板参数类型
- integral type
- enum, enum class
- floating-point type(c++20)
- auto placeholder(c++ 17)
- class literals and concepts(c++20)
- typename一般类型
- function
- reference/pointer to global static function or object
- pointer to member type
- nullptr(c++14)
## array/pointer
```c++
template<int* ptr> // pointer
void g() {
cout << ptr[0];
}
template<int (&array)[3]> // reference
void f() {
cout << array[0];
}
int array[] = {2, 3, 4}; // global
int main() {
f<array>(); // print 2
g<array>(); // print 2
}
```
## class member
```c++
struct A {
int x = 5;
int y[3] = {4, 2, 3};
};
template<int A::*x> // pointer to
void h1() {} // member type
template<int (A::*y)[3]> // pointer to
void h2() {} // member type
int main() {
h1<&A::x>();
h2<&A::y>();
}
```
## function type
```c++
template<int (*F)(int, int)> // <-- signature of "f"
int apply1(int a, int b) {
return F(a, b);
}
int f(int a, int b) { return a + b; }
int g(int a, int b) { return a * b; }
template<decltype(f) F> // alternative syntax
int apply2(int a, int b) {
return F(a, b);
}
int main() {
apply1<f>(2, 3); // return 5
apply2<g>(2, 3); // return 6
}
```
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
## 存到数组
```C++
template<typename... TArgs>
void f(TArgs... args) {
constexpr int Size = sizeof...(args);
int array[] = {args...};
for (auto x : array)
cout << x << " ";
}
f(1, 2, 3); // print "1 2 3"
f(1, 2, 3, 4); // print "1 2 3 4"
```
## Folding Expression
- Example1
```C++
template<typename... Args>
auto add_unary(Args... args) { // Unary folding
return (... + args); // unfold: 1 + 2.0f + 3ull
}
template<typename... Args>
auto add_binary(Args... args) { // Binary folding
return (1 + ... + args); // unfold: 1 + 1 + 2.0f + 3ull
}
add_unary(1, 2.0f, 3ll); // returns 6.0f (float)
add_binary(1, 2.0f, 3ll); // returns 7.0f (float) 
```
- Example2
```C++
template<typename T>
T square(T value) { return value * value; }
template<typename... TArgs>
auto add_square(TArgs... args) {
return (square(args) + ...); // square() is applied to each
} // variadic argument
add_square(2, 2, 3.0f); // returns 17.0f
```
## 获取函数的形参个数
```C++
template <typename T>
struct GetArity;
// generic function pointer
template<typename R, typename... Args>
struct GetArity<R(*)(Args...)> {
static constexpr int value = sizeof...(Args);
};
// generic function reference
template<typename R, typename... Args>
struct GetArity<R(&)(Args...)> {
static constexpr int value = sizeof...(Args);
};
// generic function object
template<typename R, typename... Args>
struct GetArity<R(Args...)> {
static constexpr int value = sizeof...(Args);
}; 
void f(int, char, double) {}
int main() {
// function object
GetArity<decltype(f)>::value;
auto& g = f;
// function reference
GetArity<decltype(g)>::value;
// function reference
GetArity<decltype((f))>::value;
auto* h = f;
// function pointer
GetArity<decltype(h)>::value;
}
```
## 获得lamda或operator()的参数个数
```C++
template <typename T>
struct GetArity;
template<typename R, typename C, typename... Args>
struct GetArity<R(C::*)(Args...)> { // class member
static constexpr int value = sizeof...(Args);
};
template<typename R, typename C, typename... Args>
struct GetArity<R(C::*)(Args...) const> { // "const" class member
static constexpr int value = sizeof...(Args);
};
struct A {
void operator()(char, char) {}
void operator()(char, char) const {}
};
GetArity<A>::value; // call GetArity<R(C::*)(Args...)>
GetArity<const A>::value; // call GetArity<R(C::*)(Args...) const>
```
# Concepts和requires(c++20)
## concepts语法
```C++
[template arguments]
concept [name] = [compile-time boolean expression];
```
## concepts例子
```C++
template<typename T>
concept Arithmetic = std::is_arithmetic_v<T>;
template<Arithmetic T>
T add(T valueA, T valueB) {
return valueA + valueB;
}
```
## requires clause语法
```C++
requires [compile-time boolean expression or Concept]
```
### concept example
```C++
template<typename T>
requires Arithmetic<T>
T add(T valueA, T valueB) {
return valueA + valueB;
}
```
### bool expression example
```C++
template<typename T>
T add(T valueA, T valueB) requires (sizeof(T) == 4) {
return valueA + valueB;
}
```
## requires expression语法
A requires expression is a compile-time expression of type bool that defines the
constraints on template arguments
```C++
requires [(arguments)] {
[SFINAE contrain]; // or
requires [predicate];
} -> bool
```
### example
```C++
template<typename T>
concept MyConcept = requires (T a, T b) { // First case: SFINAE constrains
a + b; // Req. 1 - support add operator
a[0]; // Req. 2 - support subscript operator
a.x; // Req. 3 - has "x" data member
a.f(); // Req. 4 - has "f" function member
typename T::type; // Req. 5 - has "type" field
};
# include <concept>
template<typename T>
concept MyConcept2 = requires (T a, T b) {
{*a + 1} -> std::convertible_to<float>; // Req. 6 - can be deferred and the sum
// with an integer is convertible
// to float
{a * a} -> std::same_as<int>; // Req. 7 - "a * a" must be valid and
// the result type is "int"
};
```
## requires Clause + Expression
```C++
template<typename T>
void f(T a) requires requires { T::value; }
// clause -> SFINAE followed by
// expression -> bool (zero args)
{}

template<typename T>
T increment(T a) requires requires (T x) { x + 1; }
// clause -> SFINAE followed by
// expression -> bool (one arg)
{
return a + 1;
}
```

# Examples
## 利用特例化判断类型是否一致
```C++
template<typename T, typename R> // GENERIC template declaration
struct is_same {
static constexpr bool value = false;
};
template<typename T>
struct is_same<T, T> { // PARTIAL template specialization
static constexpr bool value = true;
};
cout << is_same< int, char>::value; // print false, generic template
cout << is_same<float, float>::value; // print true, partial template
```
## 利用特例化判断const指针
```C++
# include <type_traits>
// std::true type and std::false type contain a field "value"
// set to true or false respectively
template<typename T>
struct is_const_pointer : std::false_type {}; // GENERIC template declaration
template<typename R> // PARTIAL specialization
struct is_const_pointer<const R*> : std::true_type {};
cout << is_const_pointer<int*>::value; // print false, generic template
cout << is_const_pointer<const int*>::value; // print true, partial template
cout << is_const_pointer<int* const>::value; // print false, generic template
```
