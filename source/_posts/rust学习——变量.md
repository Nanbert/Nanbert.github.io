---
title: rust学习——变量
date: 2022-04-09 19:20:58
subtitle:
categories:
tags:
cover:
---
## 常量
- 常量值的类型必须标注
## 数值类型
### 整形
- rust默认使用i32
### 浮点形
- rust默认使用f64
- 保留指定位数：`13.14_f32.round()`默认取整,必须下划线指定类型
### NaN
- 所有跟NaN交互的操作，会返回一个NaN
- 可以用`x.is_nan()`来判读
### 序列
- 序列支持字符或整形数字
- `1..5`1到4
- `1..=5`1到5
- `(1..5).rev`逆序
## 字符类型
- 字符只能用`''`括起来，占4个字节，可以表示所有Unicode值
## 单元类型
- 单元类型就是`()`，main函数,prinln!()就返回该类型
- 没有返回值的函数与这个不同，它们被称为发散函数
- 可以用作map的值，用来占位
## 函数
- 格式:`fn funcname(i:type,j:type) -> type{}`
- 发散函数用来定义导致程序崩溃的函数或死循环的函数
```bash
fn dead_end() -> !{
	panic!("dead!!");
}
fn forever() ->!{
	loop{
	};
}
```
## 所有权原则
谨记一下两点规则：
- Rust中每一个值都有且只有一个所有者（变量）
- 当所有者（变量）离开作用域范围时，这个值将被丢弃
### 基本类型
基本类型的赋值都是深拷贝，在栈上占用新空间,因为它们拥有`Copy`特性
### 堆上的数据
堆上的数据要想不发生所有权转移，就需要深拷贝，需要调用`clone`特性
### tips
- 不可变引用`&T`含有`Copy`特性
## 引用
- 可变引用同一作用域只能存在一个
- 可变引用与不可变引用不能同时存在
- 引用的作用域为最后一次使用的位置
## 结构体
- 结构体不支持结构体内部某个字段标记为可变，只能结构体的实例可以标记为可变
- 结构体必须有名称
### 定义
```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```
### 实例化
#### 标准的做法
```rust
let mut user1 = User {
	email: String::from("someone@example.com"),
	username: String::from("someusername123"),
	active: true,
	sign_in_count: 1,
};
```
#### 从现有的实例进行部分赋值，其余重新赋值：
```rust
let user2 = User {
	email: String::from("another@example.com"),
	..user1
}
```
- `..user1`必须在结构体的尾部使用
- 这个user1某些字段可能发生所有权转移，导致user1无法使用，但是其他字段可以继续使用访问
### 构建函数
- 当函数参数和结构体字段同名时，可以直接使用缩略的方式进行初始化
```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```
### 元组结构体
- 结构体必须要有名称，但是字段可以没名称,字段没名称的称为元组结构体
- 可以像元组一样通过`.x`来访问各个字段
```rust
struct Point(i32,i32,i32);
let origin=Point(0,0,0);
```
### 引用类型的字段
- 结构体的字段借用其他数据，必须注明生命周期
### 单元结构体
- 没有字段和属性，只关心行为的结构体
```rust
struct AlwaysEqual;

let subject = AlwaysEqual;

// 我们不关心 AlwaysEqual 的字段数据，只关心它的行为，因此将它声明为单元结构体，然后再为它实现某个特征
impl SomeTrait for AlwaysEqual {

}
```
### 结构体的输出
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
	println!("{:?}",rect1)
	println!("{:#?}",rect1)
}
```
## 枚举
- 任何类型的数据都可以放入枚举成员中
- 枚举也可以通过`impl`定义自己的函数
```bash
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let m1 = Message::Quit;
    let m2 = Message::Move{x:1,y:1};
    let m3 = Message::ChangeColor(255,255,0);
}
```
## 数组
- 可以直接赋值，自动推断类型与数组长度
- 完整定义
  - `let a: [类型;长度]=...`
  - `let b = [初始化值;长度]`
- 数组切片类型为`&[类型]`，数组本身类型为`[类型;长度]`
- 遍历
```rust
//不变引用
for item in &container{...}
//可变引用
for item in &mut container{...}
//获取元素的索引
for (i,v) in array.iter.enumerate(){
	println!("第{}个元素是{}",i+1,v);
}
```
- 由于数组本身类型包含长度，所以泛型需要使用const泛型(一般直接使用数组切片即可，不需要这么烦):
```rust
fn display_array<T: std::fmt::Debug, const N: usize>(arr: [T; N]) {
    println!("{:?}", arr);
}
fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    display_array(arr);

    let arr: [i32; 2] = [1, 2];
    display_array(arr);
}
```
其中N就是泛型，usize表示它基于值类型usize
## trick
- 数字字面量可插入下划线提高可读性：`const MAX_POINTS: u32 = 100_000;`
- 数字字面量也可以用下划线表面类型：`let a=23_u32`
