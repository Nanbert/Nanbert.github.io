---
title: rust学习——控制
date: 2022-04-11 02:54:17
subtitle:
categories:
tags:
cover:
---
## if
- if语句块是表达式
- 赋值时，每个分支返回的类型要一样
## for
- 不声明一个变量，循环指定次数
```rust
for _ in 0..10{
 //...
}
```
## loop
- loop也是一个表达式
## break
- break可以单独使用，也可以带一个返回值，有些类似return
```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {}", result);
}
```
## 循环标签
- break和continue都支持循环标签,用于控制多层循环
```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {}", count);
        let mut remaining = 10;

        loop {
            println!("remaining = {}", remaining);
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {}", count);
}
```
