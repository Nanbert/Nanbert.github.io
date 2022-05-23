---
title: rust学习——泛型与特征
date: 2022-05-21 14:14:14
subtitle:
categories:
tags:
cover:
---
## 泛型
- 泛型函数：`fn largest<T> (list:&[T]) -> T`
- 泛型结构体：`struct Point<T>{}`
- 泛型枚举：`enum Option<T>{}`
- 泛型方法：`impl<T> Point<T>`
- 多重泛型：
```rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}
```
- 为具体泛型实现方法：`impl Point<f32>{}`
### const泛型
- 语法：`fn display_array<T: std::fmt::Debug,const N: usize>(arr:[T;N])`
- 限制大小：
```rust
// 目前只能在nightly版本下使用
#![allow(incomplete_features)]
#![feature(generic_const_exprs)]

fn something<T>(val: T)
where
    Assert<{ core::mem::size_of::<T>() < 768 }>: IsTrue,
    //       ^-----------------------------^ 这里是一个 const 表达式，换成其它的 const 表达式也可以
{
    //
}

fn main() {
    something([0u8; 0]); // ok
    something([0u8; 512]); // ok
    something([0u8; 1024]); // 编译错误，数组长度是1024字节，超过了768字节的参数长度限制
}
```
## 特征
- **如果你想要为类型A实现特征T,那么A或T至少一个在当前作用域中定义**
- **如果使用一个特征的方法，那么需要引入该特征到当前作用域**
### 定义：
```rust
pub trait Summary{
	fn summarize(&self) -> String;
}
```
### 为某个类型实现特征
```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
pub struct Post {
    pub title: String, // 标题
    pub author: String, // 作者
    pub content: String, // 内容
}

impl Summary for Post {
    fn summarize(&self) -> String {
        format!("文章{}, 作者是{}", self.title, self.author)
    }
}
```
### 默认实现
```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
impl Summary for Post{}
```
### 特征多个方法混用
```
pub trait Summary {
    fn summarize_author(&self) -> String;
	//可以调用其他方法
    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
//只需实现一个特征即可
impl Summary for Weibo{
	fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}
```
### 特征用作函数参数
#### 形式
- `pub fn notify(item1: &impl Summary,item2: &impl Summary)`
- `pub fn notify<T: Summary>(item1:&T,item2:&T)`
**以上两种是有区别的，第一个item1和item2可以是不同类型，第二个必须是一个类型**
#### 多个约束
- `pub fn notify(item:&(impl Summary+Display)) {}`
- `pub fn notify<T: Summary+Display>(item:&T)`
#### 使用where简化
```rust
fn main() {
fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{}
}
```
#### 有条件的实现方法或特征
```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```
### 返回特征
`fn returns_summarizable() -> impl Summary`
**该形式返回的值只能是某个具体的类型，即所有return值必须相同类型，如果要使用不同类型但实现同一特征的，需要使用特征对象**
### 常用手动实现标准库特征
#### std::ops::Add--自定义加操作
```rust
use std::ops::Add;

// 为Point结构体派生Debug特征，用于格式化输出
#[derive(Debug)]
struct Point<T: Add<T, Output = T>> { //限制类型T必须实现了Add特征，否则无法进行+操作。
    x: T,
    y: T,
}

impl<T: Add<T, Output = T>> Add for Point<T> {
    type Output = Point<T>;

    fn add(self, p: Point<T>) -> Point<T> {
        Point{
            x: self.x + p.x,
            y: self.y + p.y,
        }
    }
}

fn add<T: Add<T, Output=T>>(a:T, b:T) -> T {
    a + b
}

fn main() {
    let p1 = Point{x: 1.1f32, y: 1.1f32};
    let p2 = Point{x: 2.1f32, y: 2.1f32};
    println!("{:?}", add(p1, p2));

    let p3 = Point{x: 1i32, y: 1i32};
    let p4 = Point{x: 2i32, y: 2i32};
    println!("{:?}", add(p3, p4));
}
```
## 特征对象
通过`&`引用或`Box<T>`智能指针来创建特征对象
- `fn draw1(x:Box<dyn Draw>)`
- `fn draw2(x:&dyn Draw)`
**dyn关键子只用在类型声明，创建具体类型时无需使用dyn，如`Vec<Box<dyn Draw>>`,直接v.push(Box::new(x))**
- 特征对象占用两个指针空间大小，一个ptr,一个vptr（相当于c++的虚函数多态,c++我也不懂，嘎嘎,简单就是该特征对应具体实例类型的同一方法的不同实现）
### 特征对象的限制
只有对象安全的特征才能拥有特征对象，
- 特征方法的返回类型不是Self
- 方法没有任何泛型参数
如果误用会报错`the trait xxx cannot be made into an object`
## 特征冷门概念
### 关联类型
就是在特征定义的语句块中，申明一个自定义类型，如下:
```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        // --snip--
    }
}

fn main() {
    let c = Counter{..}
    c.next()
}
```
如上不使用如下泛型的原因，纯粹可读性更好
```rust
pub trait Iterator<Item>{
	fn next(&mut self) -> Option<Item>;
}
```
同时type不仅可以指类型，还可以指代特征，如下：
```rust
pub trait CacheableItem: Clone + Default + fmt::Debug + Decodable + Encodable {
  type Address: AsRef<[u8]> + Clone + fmt::Debug + Eq + Hash;
  fn is_null(&self) -> bool;
}
```
再例如：
```rust
fn main() {
trait Container<A,B> {
    fn contains(&self,a: A,b: B) -> bool;
}

fn difference<A,B,C>(container: &C) -> i32
  where
    C : Container<A,B> {...}

//简单方法
trait Container{
    type A;
    type B;
    fn contains(&self, a: &Self::A, b: &Self::B) -> bool;
}

fn difference<C: Container>(container: &C) {}
```
### 默认泛型类型参数
```rust
trait Add<RHS=Self> {
    type Output;

    fn add(self, rhs: RHS) -> Self::Output;
}
//使用默认参数,注意这和不使用泛型一样，因此这一般用来扩展而无需改变原代码
impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}
不使用默认参数
impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```
### 同名方法
#### 特征方法与本身方法重名
假设Human类型有两个特征Pilot和Wizard,并这两个特征都有fly方法，Human类型本身也有fly方法。假设有个实例person:
- person.fly():调用的是类型本身的方法
- Pilot::fly(&person):调用Pilot特征上的方法
- Wizard::fly(&person):调用Wizard特征上的方法
#### 特征方法与关联函数重名
```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", Dog::baby_name());//Spot
	println!("A baby dog is called a {}", <Dog as Animal>::baby_name());//puppy
}
```
使用了完全限定语法`<Type as Trait>::function(receiver_if_method, next_arg, ...);`
第一个receiver只有方法才有，有三种（self,&self,&mut self）,一般不这么写，编译器帮我们做了，只有分不清时才这么写
### 特征的特征约束
即你要实现特征A必须先实现B:`trait A:B{}`
### newtype--绕过特征或类型必须在当前作用域
- rust中无法直接为外部类型实现外部特征，可以使用`newtype`:
```rust
struct Array(Vec<i32>);

use std::fmt;
impl fmt::Display for Array {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "数组是：{:?}", self.0)
    }
}
fn main() {
    let arr = Array(vec![1, 2, 3]);
    println!("{}", arr);
}
```
