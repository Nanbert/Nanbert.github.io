---
title: C++小知识
date: 2019-07-18 17:14:17
subtitle:
categories: C++
tags:
index_img:
---
1.  
八进制整型常量以"0"开头如:0123
十六进制以"0x(X)"开头
长整型以"L(l)"结尾
无符号型以"U(u)"结尾
2.默认数据类型为int型,默认实型为double型,若要表示float型在结尾加"F(f)"
3.  
extern可以使用后面的定义的数据:
extern int h, k;
   .
   .
   .
int h=1,k=2;
4.  
register+类型+变量名,存放在寄存器,加快调用
auto+类型+变量名,自动分配存储,默认就是这个可以省略
5.  
内部函数,在前面加static,仅在包含文件中有效.
外部函数在前面加extern,默认就是这个.
6.  
const与指针
`const double *cptr; //cptr指针指向一个类型为double的常量值`
`double *const cura; //cura是一个const指针,即无法改变指向的对象` 
7.  
指针与函数
返回指针的函数称为指针函数，本质上是一个返回指针的函数，示例如下:
`int *test(a,b)`
函数指针就是指向函数的指针。示例如下:
`int (*test)(int a,int b)`
8.  
数组与指针
数组指针是一个指针变量，它指向一个数组。示例如下:
`int (*ap)[2]`
指针数组是一个包含指针元素的数组，它的元素可以指向相同类型的不同对象
`int *ap[2]`
ps:从以上可以看出指针从左至右结合，
9.  
指针与应用  
.初始化要求不同，引用创建的同时必须初始化，而指针不必，可以重新赋值。
.可修改性不同，引用一旦被初始化，就不可更改，而指针可以指向另一个变量
.不存在NULL引用，指针可以，指针更灵活，但风险也大
.指针是一个实体，引用是一个别名
.内存上，指针有分配的存储空间，而应用则不用
10.
c++无符号右移高位都补0
11.
`auto dp = vector<vector<int>>(length, vector<int>(length));`可以这样声明数组
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
