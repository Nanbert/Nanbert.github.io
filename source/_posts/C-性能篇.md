---
title: C++性能篇
date: 2024-04-18 03:55:26
tags:
---

# IO
- 尽可能使用`\n`,而不是endl
- `std::ios_base::sync_with_stdio(false)`,禁止与printf/scanf的同步
- `<istream_obj>.tie(nullptr)`:Disable IO flushing when mixing istream/ostream
- `<ifstream_obj>.rdbuf()->pubsetbuf(buffer_var,buffer_size)`:increase io buffer size
- printf比ostream更快
- LZ4和zstd是很快的开元库 
# 内存
- 在局部作用域中，static const比const更快，避免了栈分配
# 操作
- 尽可能常量放在一起
- unsigned比signed更快
- `||`比`|`更好
- 乘法比除法更快
- Checking if a non-negative value x is within a range [A, B] can be optimized if B > A (useful when the condition is repeated multiple times)
- 指针用中括号操作比加法操作更好
- 尽可能的使得symbol不全局可见，即多用匿名空间，static
- Prefer signed integer for loop indexing. The compiler optimizes more aggressively such loops since integer overflow is not defined
- exceptions影响性能，尽可能用noexcept
- smart pointer影响性能
- lambda比std::function和函数指针更好
- 避免dynamic_cast
- 除了以上几点尽可能用STL
```C++
if (x >= A && x <= B)
// STEP 1: subtract A
if (x - A >= A - A && x - A <= B - A)
// -->
if (x - A >= 0 && x - A <= B - A) // B - A is precomputed
// STEP 2
// - convert "x - A >= 0" --> (unsigned) (x - A)
// - "B - A" is always positive
if ((unsigned) (x - A) <= (unsigned) (B - A))
```
# 数值转换
|From|To|Cost|
|:-:|:-:|:-:|
|Signed|Unsigned|no cost, bit representation is the same|
|Unsigned|Larger Unsigned|no cost, register extended|
|Signed|Larger Signed|1 clock-cycle, register + sign extended|
|Integer|Floating-point|4-16 clock-cycles Signed → Floating-point is faster than Unsigned → Floating-point (except AVX512 instruction set is enabled)|
|Floating-point|Integer|fast if SSE2, slow otherwise (50-100 clock-cycles)|
# Compiler intrinsic functions
编译器内提供的函数，可能缺少跨平台支持，但更好的优化，不支持的花会是很危险的
# __restrict
该关键字表明，这两个指针指向的内存不在同一快区域，可以使得编译器更好的优化
```
void matrix_mul_v1(const int* A,
const int* B,
int N,
int* C) {
// below is faster
void matrix_mul_v2(const int* __restrict A,
const int* __restrict B,
int N,
int* __restrict C) {
```
# 编译器__attribute__
设置某些属性可以帮助编译器更好的优化
- `__attribute__((visibility("hidden")))`:效果等价于匿名空间
- `__attribute__(always_inline)`:强制inline
- `__attribute__(noinline)`:强制不inline
- `__attribute__(pure)`:attribute (Clang, GCC) specifies that a function has no side effects on its parameters or program state (external global references)
- `__attribute__(const)`:attribute (Clang, GCC) specifies that a function doesn’t depend (read) on external global references
# gprof
Code Instrumentation
`g++ -pg [flags] <source_files>`
Important: -pg is required also for linking and it is not supported by clang
- Run the program (it produces the file gmon.out)
- Run gprof on gmon.out
- `gprof -q <executable> gmon.out`可以分析出函数调用图
# uftrace
火焰图
```bash
$ gcc -pg <program>.cpp
$ uftrace record <executable>
$ uftrace replay
```
# 性能工具
- callgrind(valgrind)
- cachegrind
- KCachegrind(linux)/Qcachegrind(win)
- gprof2dot
- perf
- hotspot
