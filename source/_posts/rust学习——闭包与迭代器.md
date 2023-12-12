---
title: rust学习——闭包与迭代器
date: 2022-06-03 18:10:58
subtitle:
categories:
tags:
index_img: /images/rust.png
banner_img: /images/rust.png
---
## 迭代器
### IntoIterator特征
对于数组的循环实质上是：`for v in arr.into_iter()`，一个编译器语法糖
也可以使用`IntoIterator::into_iter`方法调用
- into_iter会夺走所有权
- iter是借用
- iter_mut是可变借用
### Iterator特征
特征原型如下
```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // 省略其余有默认实现的方法
}
```
- next 方法返回的是 Option 类型，当有值时返回 Some(i32)，无值时返回 None
### 消费者与适配器
#### 消费者
消费者是迭代器上的方法，会消耗迭代器中的元素
- sum:会拿走迭代器所有权，求和
- fold:类似sum,提供个初始值：`v.iter().fold(0,|sum,acm| sum+acm)`
#### 适配器
适配器会返回一个新的迭代器，是惰性的，需要一个消费者爱收尾
- map:接受一个闭包，映射每个值，`v1.iter().map(|x| x+1).sum()`
- zip:把两个迭代器压缩到一起，形成类似`Iterator<Item=(ValueFromA, ValueFromB)>`，`let folks: HashMap<_, _> = names.into_iter().zip(ages.into_iter()).collect();`
- collect:将一个迭代器中的元素收集到指定类型中，`let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();`
- filter:对每个值进行过滤，`hoes.into_iter().filter(|s| s.size == shoe_size).collect()`
- enumerate:返回一个元组（索引，值）
### 手动实现迭代器特征
```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        if self.count < 5 {
            self.count += 1;
            Some(self.count)
        } else {
            None
        }
    }
}
```
## 闭包
- 闭包的参数类型可以不标注，由编译器自行推导，但是为了可读性，最好标注
### 格式
```rust
fn main() {
|param1, param2,...| {
    语句1;
    语句2;
    返回表达式
}
```
只有一个返回表达式可以不加大括号
### 三种Fn特征
- 一个闭包实现了哪种Fn特征取决于闭包如何使用被捕获的变量,而不是取决于如何使用它们(可能同时实现了FnOnce和Fn特征)
- 所有的闭包都自动实现了 FnOnce 特征，因此任何一个闭包都至少可以被调用一次
- 没有移出所捕获变量的所有权的闭包自动实现了 FnMut 特征
- 不需要对捕获变量进行改变的闭包自动实现了 Fn 特征
#### Fn
以不可变借用捕获变量
#### FnOnce
该类型闭包表明拿走被捕获变量的所有权。只能运行一次。
#### move关键字
move会取得捕获变量的所有权，这种用法常用于闭包的生命周期大于捕获变量的生命周期
#### FnMut
以可变借用的方式捕获变量
