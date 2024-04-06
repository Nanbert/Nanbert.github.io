---
title: C++标准库
date: 2019-08-10 21:22:59
subtitle:
categories: C++
tags:
index_img: /images/STL.webp
banner_img: /images/STL.webp
---
# 顺序容器
|方法|说明|
|:-:|:-:|
|c.resize(n)|调整c的大小为n个元素。要么多出的元素被丢弃,要么新添加默认值的元素|
|c.resize(n,t)|调整c的大小为n个元素。任何新添加的元素初始为t|
|c.shrink_to_fit()|只适用于vector,string和deque。将capacity()减少与size()相同大小|
|c.capacity()|只适用于vector和string,返回c可以保存多少元素|
|c.reserve(n)|只适用于vector和string,分配至少容纳n个元素的内存空间|
|c.assign(i1)|将c中的元素全部替换为i1元素,i1可以和c类型不同,但元素必须相同,甚至可以是map与set类型|
|c.assign(n,t)|将c中的元素全部换为n个t|

# 优先队列
格式:`priority_queue<Type,Container,Functional>`
例子:`priority_queue<int,vector<int>,myCompare>`,myCompare可以替换为less<int>降序,greater<int>升序
# 数学相关
## cmath
- fabs(x) computes absolute value, |x|, C++11
- exp(x) returns e raised to the given powerx
- exp2(x) returns 2 raised to the given power , C++11
- log(x) computes natural (base e) logarithm, loge (x)
- log10(x) computes base 10 logarithm, log10(x)
- log2(x) computes base 2 logarithm, log2 (x), C++11
- pow(x, y) raises a number to the given power, x y
- sqrt(x) computes square root
- cqrt(x) computes cubic root x, C++11
- sin(x) computes sine, sin(x)
- cos(x) computes cosine, cos(x)
- tan(x) computes tangent, tan(x)
- ceil(x) nearest integer not less than the given value, ⌈x⌉
- floor(x) nearest integer not greater than the given value, ⌊x⌋
- atan2(y,x)
- atan(y)
- asin()
- acos()
- round|lround|llround(x) nearest integer(x+0.5),  (return type: floating point, long, long long respectivel
## limits
- `T numeric_limits<T>:: max()`
- `T numeric_limits<T>:: min()`
- `T numeric_limits<T>:: lowest()`
## numeric(c++20)
- `e` Euler number e
- `pi` π
- `phi`黄金分割比率
- `sqrt2`
# 随机数
- 选择随机种子seed(随机引擎的初始值)
- 定义随机引擎`<type of random engine> generator(seed)`
- 定义分布`<type of distribution> distribution(range start, range end)`
- 产生随机数`distribution(generator)`
## 产生种子
- 自定义：`unsigned seed = 2`,注意相同种子，产生的随机序列是一样的
- 使用时间
```C++
#include <random>
#include <chrono>
unsigned seed = std::chrono::system_clock::now()
.time_since_epoch().count();
std::default_random_engine generator(seed);
```
### 使用random_device
```C++
#include <random>
std::random_device rnd_device;
std::default_random_engine generator(rnd_device());
```
### 使用seed seq
```C++
#include <random>
#include <chrono>
unsigned seed1 = std::chrono::system_clock::now()
.time_since_epoch().count();
unsigned seed2 = seed1 + 1000;
std::seed_seq seq1{ seed1, seed2 };
std::default_random_engine generator1(seq);
```
## 随机引擎
三种随机引擎的对比
|Generator|Quality|Period|Randomness|C++引擎|
|:-:|:-:|:-:|:-:|:-:|
|Linear Congruential|Poor|10^9|Statistical tests|std::minstd_rand,std::minstd_rand0,std::knuth_b|
|Mersenne Twister 32/64-bit|High|10^6000|Statistical tests|std::mt19937,std::19937_64|
|Subtract-with-carry 24/48-bit|Highest|10^171|Mathematically proven|std::ranlux24_base,std::ranlux48_base,std::ranlux24,std::ranlux48|
## 分布
- `uniform int distribution<T>(range start, range end)` where T is integral type
- `uniform real distribution<T>(range start, range end)` where T is floating point type
- `normal distribution<T>(mean, std dev)`where T is floating point type
- `exponential distribution<T>(lambda)`where T is floating point type
# 时间
- `Wall-Clock/Real time` It is the human perception of the passage of time from the start to the completion of a task(整个程序的时间)
- `User/CPU time` The amount of time spent by the CPU to compute in user code(用户空间的执行时间)
- `System time` The amount of time spent by the CPU to compute system calls (including I/O calls) executed into kernel code(内核消耗的时间)
如果程序是但线程并且系统负载不高则有：
Wall-clock time = User time + System time
## Real Time
### linux系统调用
```bash
# include <time.h> //struct timeval
# include <sys/time.h> //gettimeofday()
struct timeval start, end; // timeval {second, microseconds}
::gettimeofday(&start, NULL);
... // code
::gettimeofday(&end, NULL);
long start_time = start.tv_sec * 1000000 + start.tv_usec;
long end_time = end.tv_sec * 1000000 + end.tv_usec;
cout << "Elapsed: " << end_time - start_time; // in microsec
```
### chrono
```C++
# include <chrono>
auto start_time = std::chrono::system_clock::now();
... // code
auto end_time = std::chrono::system_clock::now();
std::chrono::duration<double> diff = end_time - start_time;
cout << "Elapsed: " << diff.count(); // in seconds
cout << std::chrono::duration_cast<milli>(diff).count(); // in ms
```
## User Time
### chrono
```C++
# include <chrono>
clock_t start_time = std::clock();
... // code
clock_t end_time = std::clock();
float diff = static_cast<float>(end_time - start_time) / CLOCKS_PER_SEC;
cout << "Elapsed: " << diff; // in seconds
```
### 系统调用(包含内核时间的计算方法)
```C++
# include <sys/times.h>
struct ::tms start_time, end_time;
::times(&start_time);
... // code
::times(&end_time);
auto user_diff = end_time.tmus_utime - start_time.tms_utime;
auto sys_diff = end_time.tms_stime - start_time.tms_stime;
float user = static_cast<float>(user_diff) / ::sysconf(_SC_CLK_TCK);
float sys = static_cast<float>(sys_diff) / ::sysconf(_SC_CLK_TCK);
cout << "user time: " << user; // in seconds
cout << "system time: " << sys; // in seconds
```
# tuple
```C++
# include <tuple>
std::tuple<int, float, char> f() { return {7, 0.1f, 'a'}; }
std::tuple<int, char, float> tuple1(3, 'c', 2.2f);
auto tuple2 = std::make_tuple(2, 'd', 1.5f);
cout << std::get<0>(tuple1); // print 3
cout << std::get<1>(tuple1); // print 'c'
cout << std::get<2>(tuple1); // print 2.2f
cout << (tuple1 > tuple2); // print true
auto concat = std::tuple_cat(tuple1, tuple2);
cout << std::tuple_size<decltype(concat)>::value; // print 6
using T = std::tuple_element<4, decltype(concat)>::type; // T is int
int value1; float value2;
std::tie(value1, value2, std::ignore) = f();//creates a tuple of references to its arguments
std::swap(tuple1,tuple2);
```
# variant(c++17)
代替联合体union，类型安全(切换类型前会自动析构)，见[variant](https://en.cppreference.com/w/cpp/utility/variant)
- 构造：`std::variant<int, float> tmp`
- 获取类型个数：`std::variant_size_v<decltype(tmp)> // 2`
- 获取下标：`tmp = 3.2; tmp.index() // 1`
- 判断当前值类型：`hold_alternative<float> tmp // true`
- 获取当前值类型：`static_assert(std::is_same_v<int, variant_alternative_t<0, tmp>>) // int`
## Example
```C
std::variant<int, float, std::string> tmp;
tmp = "hi";
std::get<int>(tmp); // throw exception
int* s = std::get_if<int>(tmp); // no throw, but return null
string g = std::get<std::string>(tmp); //success
```
## visitor
It is also possible to query the index at run-time depending on the type currently being held by providing a visitor
```C++
# include <variant>
struct Visitor {
void operator()(int& value) { value *= 2; }
void operator()(float& value) { value += 3.0f; } // <--
void operator()(bool& value) { value = true; }
};
std::variant<int, float, bool> v(3.3f);
std::visit(v, Visitor{});
cout << std::get<float>(v); // 6.3f
```
# optional
```C++
# include <optional>
std::optional<std::string> find(const char* set, char value) {
for (int i = 0; i < 10; i++) {
if (set[i] == value)
return i;
}
return {}; // std::nullopt;
}
=============================
# include <optional>
char set[] = "sdfslgfsdg";
auto x = find(set, 'a'); // 'a' is not present
if (!x)
cout << "not found";
if (!x.has_value())
cout << "not found";
auto y = find(set, 'l');
cout << *y << " " << y.value(); // print '4' '4'
x.value_or(-1); // returns '-1'
y.value_or(-1); // returns '4'
```
# any
```C++
# include <any>
std::any var = 1; // int
cout << var.type().name(); // print 'i'
cout << std::any_cast<int>(var);
// cout << std::any_cast<float>(var); // exception!!
var = 3.14; // double
cout << std::any_cast<double>(var);
var.reset();
cout << var.has_value(); // print 'false'
```
# stacktrace(c++23)
```C++
# include <print>
# include <stacktrace> // the program must be linked with the library
// -lstdc++_libbacktrace
// (-lstdc++exp with gcc-14 trunk)
void g() {
auto call_stack = std::stacktrace::current();
for (const auto& entry : call_stack)
std::print("{}\n", entry);//entry.description()entry.source_file(),entry.source_line()
}
void f() { g(); }
int main() { f(); }
```
# filesystem(c++17)
- Follow the Boost filesystem library
- Based on POSIX
- Fully-supported from clang 7, gcc 8, etc.
- Work on Windows, Linux, Android, etc.
## path
### member function
- root_name()
- relative_path()
- parent_path()
- filename()
- extension()
### method
- exists(path)
- file_size(path)
- is_directory(path)
- is_empty(path)
- is_regular_file(path):return true if path is no dir,hard link,soft link
- copy(path1, path2)
- copy_file(src_path, src_path, [fs::copy_options::recursive])
- create_directory(path)
- remove(path):remove a file or empty directory
- remove_all(path)
- rename(old_path,new_path)
## iterator
```C++
namespace fs = std::filesystem;
for(auto& path : fs::directory_iterator("/usr/tmp/"))
cout << path << '\n';
for(auto& path : fs::recursive_directory_iterator("/usr/tmp/"))
cout << path << '\n';
```
## example
```C++
# include <filesystem> // required
namespace fs = std::filesystem;
fs::path p1 = "/usr/tmp/my_file.txt";
cout << p1.exists(); // true
cout << p1.parent_path(); // "/usr/tmp/"
cout << p1.filename(); // "my_file"
cout << p1.extension(); // "txt"
cout << p1.is_directory(); // false
cout << p1.is_regular_file(); // true
fs::create_directory("/my_dir/");
fs::copy(p1.parent_path(), "/my_dir/", fs::copy_options::recursive);
fs::copy_file(p1, "/my_dir/my_file2.txt");
fs::remove(p1);
fs::remove_all(p1.parent_path());
```
