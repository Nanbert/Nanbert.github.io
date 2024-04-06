---
title: C++小知识
date: 2019-07-18 17:14:17
subtitle:
categories: C++
tags:
index_img: /images/Cpp.jpeg
banner_img: /images/Cpp.jpeg
---
## 指针与应用  
.初始化要求不同，引用创建的同时必须初始化，而指针不必，可以重新赋值。
.可修改性不同，引用一旦被初始化，就不可更改，而指针可以指向另一个变量
.不存在NULL引用，指针可以，指针更灵活，但风险也大
.指针是一个实体，引用是一个别名
.内存上，指针有分配的存储空间，而应用则不用
## tips
- c++无符号右移高位都补0
- `auto dp = vector<vector<int>>(length, vector<int>(length));`可以这样声明数组
- 八进制整型常量以"0"开头如:0123
- 十六进制以"0x(X)"开头
- 长整型以"L(l)"结尾
- 无符号型以"U(u)"结尾
- 默认数据类型为int型,默认实型为double型,若要表示float型在结尾加"F(f)"
- register+类型+变量名,存放在寄存器,加快调用
## lambda
格式:`[capture list] (parameter list) -> return type {function body}`,capture list可以为空，里面一般是包含此lambda的函数的(非static)局部变量,该函数外的变量可以在函数体内直接用,return type可以省略。

## 技巧
### 使用初始化列表比较大小
**差：**`small = min(x,min(y,z));`
**好：**`small = min({x,y,z,k});`
### 解构
```C++
pair<int,int> cur = {1,2};
auto [x,y] = cur;
```
### debug宏
```C++
#definde deb(x) cout<< #x << " = " << x
int ten = 10;
deb(ten); //prints "ten = 10"
```
### 支持多参数的debug宏
```C++
#define deb(...) logger(#__VA_ARGS__, __VA_ARGS__)
template<typename ...Args>
void logger(string vars, Args&&... values) {
    cout << vars << " = ";
    string delim = "";
    (..., (cout << delim << values, delim = ", "));
}

int xx = 3, yy = 10, xxyy = 103;
deb(xx); // prints "xx = 3"
deb(xx, yy, xxyy); // prints "xx, yy, xxyy = 3, 10, 103"
```
### 读写容器和多变量
```C++
template <typename... T>
void read(T &...args) {
    ((cin >> args), ...);
}

template <typename... T>
void write(string delimiter, T &&...args) {
    ((cout << args << delimiter), ...);
}

template <typename T>
void readContainer(T &t) {
    for (auto &e : t) {
        read(e);
    }
}

template <typename T>
void writeContainer(string delimiter, T &t) {
    for (const auto &e : t) {
        write(delimiter, e);
    }
    write("\n");
}
// Question: read three space seprated integers and print them in different lines.
	int x, y, z;
	read(x, y, z);
	write("\n", x, y, z);

// even works with variable data types :)
	int n;
	string s;
	read(n, s);
	write(" ", s, "has length", n, "\n");

// Question: read an array of `N` integers and print it to the output console.
	int N;
	read(N);
	vector<int> arr(N);
	readContainer(arr);
	writeContainer(" ", arr); // output: arr[0] arr[1] arr[2] ... arr[N - 1]
	writeContainer("\n", arr);
	/**
	* output:
	* arr[0]
	* arr[1]
	* arr[2]
	* ...
	* ...
	* ...
	* arr[N - 1]
	*/
```
### debug函数
```C++
template<typename ...T>
void printer(T&&... args) {
    ((cout << args << " "), ...);
}

int age = 25;
string name = "Rachit";
printer("I am", name, ',', age, "years old");
// ^ This prints the following
// I am Rachit, 25 years old
template<typename F>
auto debug_func(const F& func) {
    return [func](auto &&...args) { // forward reference
        cout << "input = ";
        printer(args...);
        auto res = func(forward<decltype(args)>(args)...);
        cout << "res = " << res << endl;
        return res;
    };
}

debug_func(pow)(2, 3);
// ^ this automatically prints
// input = 2 3 res = 8
```
### 默认构造
默认第一个类型的构造函数，第一个类型必须有构造函数，可以用monostate来作第一个参数，类似空指针
## span(c++20)
span(std::string_view类似)是对数组的引用，可以对C风格的数组引用，可以使用vector的部分风格，例如下
```C
void read(span<int> r) // read into the range of integers r
{
    cout<<r.size()<<endl;//100
}

int a[100];
read(a);        // better: let the compiler figure out the number of elements
```
## string_view和span<char>
string_view只读，span<char>可变
## [[maybe_unused]]
可以使用该attribute，声明条件用到的参数
`Value* find(const set<Value>& s, const Value& v, [[maybe_unused]] Hint hint)`
`[[maybe_unused]] int x = value`
## zstring或not_null<zstring>来表明C字符串
`int length(const char* p)`=>`int length(zstring p)`
`int length(not_null<zstring>)`
## 不变参数用模板
```C
int sum(...)
{
    // ...
    while (/*...*/)
        result += va_arg(list, int); // BAD, assumes it will be passed ints
    // ...
}

sum(3, 2); // ok
sum(3.14159, 2.71828); // BAD, undefined

template<class ...Args>
auto sum(Args... args) // GOOD, and much more flexible
{
    return (... + args); // note: C++17 "fold expression"
}

sum(3, 2); // ok: 5
sum(3.14159, 2.71828); // ok: ~5.85987
```
## 原始指针只表示内存地址，owner表示所有权
```C++
template<typename T>
class X2 {
public:
    owner<T*> p;  // OK: p is owning
    T* q;         // OK: q is not owning
    // ...
};
```
## C和C++互相调用
- 从C++调用C:
    - in C:
    double sqrt(double);
    - in C++:
    extern "C" double sqrt(double);
    sqrt(2);
- 从C调用C++:
    - in C:
    X call_f(struct Y*, int);
    - in C++:
    extern "C" X call_f(Y* p, int i)
    {
    return p->f(i);   // possibly a virtual function call
    }
## c++23的print
```C++
#include <print>
int main()
{
    std::print("Hello World! {}, {}, {}\n", 3, 4ll, "aa");
    // print "Hello World! 3 4 aa"
}
```

## c++20的`<=>`
```C++
(3 <=> 5) == 0; // false
('a' <=> 'a') == 0; // true
(3 <=> 5) < 0; // true
(7 <=> 5) < 0; // false
```
强大的default
```C++
# include <compare>
struct Obj {
int x;
char y;
short z[2];
auto operator<=>(const Obj&) const = default;
// if x == other.x, then compare y
// if y == other.y, then compare z
// if z[0] == other.z[0], then compare z[1]
};
```

## c++20的有符号与无符号的比较
```C++
bool cmp_equal(T1 a, T2 b)
bool cmp_not_equal(T1 a, T2 b)
bool cmp_less(T1 a, T2 b)
bool cmp_greater(T1 a, T2 b)
bool cmp_less_equal(T1 a, T2 b)
bool cmp_greater_equal(T1 a, T2 b)
# include <utility>
unsigned a = 4;
int b = -3;
bool v1 = (a > b); // false!!!, see next slides
bool v2 = std::cmp_greater(a, b); // true
```
## C++17 enum class支持attributes
```c++
enum class Color { RED, GREEN, BLUE [[deprecated]] };
auto x = Color::BLUE; // compiler warning
```
## 函数属性

|关键字|最低标准|含义|
|:-:|:-:|:-:|
|`[[noreturn]]`|c++11|表示函数不返回|
|`[[deprecated]],[[deprecated("reason")]]`|c++14|表示将会弃用函数，产生编译告警|
|`[[nodiscard]]`|c++17|见下|
|`[[nodiscard("reason")]]`|c++20|如果返回值没被使用，会产生告警|
|`[[maybe_unused]]`|c++17|未使用的变量不产生告警|

```C++
[[noreturn]] void f() { std::exit(0); }
[[deprecated]] void my_rand() { ... }
[[nodiscard]] bool g(int& x) {
update(x);
bool status = ...;
return status;
}
void h([[maybe_unused]] x) {
#if !defined(SKIP_COMPUTATION)
... use x ...
#endif
}
//----------------------------------------------------------------------
my_rand(); // WARNING "deprecated"
g(y); // WARNING "discard return value"
int z = g(); // no warning
h(3); // no warning if SKIP_COMPUTATION is defined
```

## 宏
### 使用条件
不建议使用宏，一般在以下情况使用宏
- 条件编译： 不同架构、编译器等
- 多语言混编
- 复杂名字代替
### 内置宏
```C++
# include <iostream>
void f(int p) {
std::cout << __FILE__ << ":" << __LINE__; // print 'source.cpp:4'
std::cout << __FUNCTION__; // print 'f'
std::cout << __func__; // print 'f'
}
// see template lectures
template<typename T>
float g(T p) {
std::cout << __PRETTY_FUNCTION__; // print 'float g(T) [T = int]'
return 0.0f;
}
void g1() { g(3); }
```
C++20在<source_location>提供了函数方法来代替这些宏。
```C++
# include <source_location>
void f(std::source_location s = std::source_location::current()) {
cout << "function: " << s.function_name() << ", line " << s.line();
} // column(),file_name() also support
f(); // print: "function: f, line 6"
```
- `__DATE__`:输出编译的开始日期以'mmm dd yyyy'格式
- `__TIME__`:输出编译的开始时间
### 常见的条件编译
可见网址![](https://sourceforge.net/p/predef/wiki/Home/)
可见网址![](https://abseil.io/docs/cpp/platforms/macros)

|语法|含义|
|:-:|:-:|
|`#if defined( cplusplus)`| C++ code|
|`#if cplusplus == 199711L`| ISO C++ 1998/2003,仅限于linux,MSVC的2011和2014也是该值|
|`#if cplusplus == 201103L`| ISO C++ 2011,仅限于linux|
|`#if cplusplus == 201402L`| ISO C++ 2014,仅限于linux|
|`#if cplusplus == 201703L`| ISO C++ 2017|
|`#if defined( GNUG )`| The compiler is gcc/g++ |
|`#if defined( clang )`| The compiler is clang/clang++|
|`#if defined( MSC VER)`| The compiler is Microsoft Visual C++|
|`#if defined( WIN64)`| OS is Windows 64-bit|
|`#if defined( linux )`| OS is Linux|
|`#if defined( APPLE )`| OS is Mac OS|
|`#if defined( MINGW32 )`| OS is MinGW 32-bit|

### 其他
- `#`:等价于加个双引号
```C++
# define STRING_MACRO(string) #string
cout << STRING_MACRO(hello); // equivalent to "hello"
# define INFO_MACRO(my_func) \
{ \
my_func \
cout << "call " << #my_func << " at " \
<< __FILE__ << ":" __LINE__; \
}
void g(int) {}
INFO_MACRO( g(3) ) // print: "call g(3) at my_file.cpp:7"
```
- `#error "text"`:编译器遇到时会输出error
- `#warning "text"`:编译器遇到时会输出warning
- `##`：连接字符串
```C++
# define FUNC_GEN_A(tokenA, tokenB) \
void tokenA##tokenB() {}
# define FUNC_GEN_B(tokenA, tokenB) \
void tokenA##_##tokenB() {}
FUNC_GEN_A(my, function)
FUNC_GEN_B(my, function)
myfunction(); // ok, from FUNC_GEN_A
my_function(); // ok, from FUNC_GEN_B
```
- `__VA_ARGS__`:多参数宏
```C++
void f(int a) { printf("%d", a); }
void f(int a, int b) { printf("%d %d", a, b); }
void f(int a, int b, int c) { printf("%d %d %d", a, b, c); }
# define PRINT(...) \
f( VA ARGS );
PRINT(1, 2)
PRINT(1, 2, 3) 
```
- `#if __has_include(<iostream>)`：c++17判断是否存在头文件
```C++
# if __has_include(<iostream>)
# include <iostream>
# endif
```
- `#if __cpp_constexpr`:C++20引入，判断编译器是否支持某特性，还有许多其它的宏
```C++
# if __cpp_constexpr
constexpr int square(int x) { return x * x; }
# endif
```
- `#pragma`:调用某个指令，依赖于编译器
### 技巧
**数字转字符串**
```C++
# define TO_LITERAL_AUX(x) #x
# define TO_LITERAL(x) TO_LITERAL_AUX(x)
int main() {
int x1 = 3 * 10;
int y1 = __LINE__ + 4;
char x2[] = TO_LITERAL(3);
char y2[] = TO_LITERAL(__LINE__);//这里你知道为啥包一层了把，有些并不是无意义的包
}
```

## Bitfield
```C++
struct S1 {
int b1 : 10; // range [0, 1023]
int b2 : 10; // range [0, 1023]
int b3 : 8; // range [0, 255]
}; // sizeof(S1): 4 bytes
struct S2 {
int b1 : 10;
int : 0; // reset: force the next field
int b2 : 10; // to start at bit 32
}; // sizeof(S1): 8 bytes
```

## type_info
```C++
struct A {
virtual void f() {}
};
struct B : A {};
A a;
B b;
A& a1 = b; // implicit upcasting
cout << typeid(a).name(); // print "1A"
cout << typeid(b).name(); // print "1B"
cout << typeid(a1).name(); // print "1B"
```
