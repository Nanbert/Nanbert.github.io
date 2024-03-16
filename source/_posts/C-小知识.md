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
## variant(c++17)
代替联合体union，类型安全(切换类型前会自动析构)，见![variant](https://en.cppreference.com/w/cpp/utility/variant)
- 构造：`std::variant<int, float> tmp`
- 获取类型个数：`std::variant_size_v<decltype(tmp)> // 2`
- 获取下标：`tmp = 3.2; tmp.index() // 1`
- 判断当前值类型：`hold_alternative<float> tmp // true`
- 获取当前值类型：`static_assert(std::is_same_v<int, variant_alternative_t<0, tmp>>) // int`
### 获取值
例子：
```C
std::variant<int, float, std::string> tmp;
tmp = "hi";
std::get<int>(tmp); // throw exception
int* s = std::get_if<int>(tmp); // no throw, but return null
string g = std::get<std::string>(tmp); //success
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
