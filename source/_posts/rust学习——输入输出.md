---
title: rust学习——输入输出
date: 2022-04-13 23:43:19
subtitle:
categories:
tags:
cover:
---

## 常用宏
- print!:将格式化文本输出到标准输出，不带换行符
- println!:同上，但是添加换行符
- format!:将格式化文本输出到String`let s1 = format!("{},world",s)`
- eprint!:不带换行符的标准错误
- eprintln!:带换行符的标准错误
## 占位符
- `{}`用于实现了`std::fmt::Display`特征的类型
- `{:?}`用于实现了`std::fmt::Debug`特征的类型
- `{:#?}`和`{:?}`基本一样，只不过更美观
- **位置参数:**`println!("{1}{}{0}{}",1,2);//=>2112`
- **具名参数:**`println!("{a}{c}",c=3,a="a")//=>a3`
- 转义左大括号:`{{`,不是用反斜杠
- 转义右大括号:`}}`,不是用反斜杠
### 格式化参数
以下以`{}`为例，`{:?}`就是在各个例子最后加个问号
#### 字符串填充
字符串格式化默认使用空格进行填充，若想自定义填充符，需要指明对齐，并且进行左对齐。
```rust
fn main() {
    //-----------------------------------
    // 以下全部输出 "Hello x    !"
    // 为"x"后面填充空格，补齐宽度5
    println!("Hello {:5}!", "x");
    // 使用参数5来指定宽度
    println!("Hello {:1$}!", "x", 5);
    // 使用x作为占位符输出内容，同时使用5作为宽度
    println!("Hello {1:0$}!", 5, "x");
    // 使用有名称的参数作为宽度
    println!("Hello {:width$}!", "x", width = 5);
    //-----------------------------------

    // 使用参数5为参数x指定宽度，同时在结尾输出参数5 => Hello x    !5
    println!("Hello {:1$}!{}", "x", 5);
}
```
#### 数字填充和正负号
数字格式化默认也是使用空格进行填充，若想自定义填充符，需要指明对齐，但与字符串左对齐不同的是，数字是右对齐。
```rust
fn main() {
    // 宽度是5 => Hello     5!
    println!("Hello {:5}!", 5);
    // 显式的输出正号 => Hello +5!
    println!("Hello {:+}!", 5);
    // 宽度5，使用0进行填充 => Hello 00005!
    println!("Hello {:05}!", 5);
    // 负号也要占用一位宽度 => Hello -0005!
    println!("Hello {:05}!", -5);
}
```
#### 精度和字符串长度
```rust
fn main() {
    let v = 3.1415926;
    // 保留小数点后两位 => 3.14
    println!("{:.2}", v);
    // 带符号保留小数点后两位 => +3.14
    println!("{:+.2}", v);
    // 不带小数 => 3
    println!("{:.0}", v);
    // 通过参数来设定精度 => 3.1416，相当于{:.4}
    println!("{:.1$}", v, 4);

    let s = "hi我是Sunface孙飞";
    // 保留字符串前三个字符 => hi我
    println!("{:.3}", s);
    // {:.*}接收两个参数，第一个是精度，第二个是被格式化的值 => Hello abc!
    println!("Hello {:.*}!", 3, "abcdefg");
}
```
#### 对齐
```rust
fn main() {
    // 以下全部都会补齐5个字符的长度
    // 左对齐 => Hello x    !
    println!("Hello {:<5}!", "x");
    // 右对齐 => Hello     x
    println!("Hello {:>5}!", "x");
    // 居中对齐 => Hello   x  !
    println!("Hello {:^5}!", "x");

    // 对齐并使用指定符号填充 => Hello x&&&&!
    // 指定符号填充的前提条件是必须有对齐字符
    println!("Hello {:&<5}!", "x");
}
```
#### 进制
- `#b`:二进制
- `#o`:八进制
- `#x`:小写十六进制
- `#X`:大写十六进制
- `x`:不带前缀的小写十六进制
```rust
fn main() {
    // 二进制 => 0b11011!
    println!("{:#b}!", 27);
    // 八进制 => 0o33!
    println!("{:#o}!", 27);
    // 十进制 => 27!
    println!("{}!", 27);
    // 小写十六进制 => 0x1b!
    println!("{:#x}!", 27);
    // 大写十六进制 => 0x1B!
    println!("{:#X}!", 27);

    // 不带前缀的十六进制 => 1b!
    println!("{:x}!", 27);

    // 使用0填充二进制，宽度为10 => 0b00011011!
    println!("{:#010b}!", 27);
}
```
#### 指数
```rust
fn main() {
    println!("{:2e}", 1000000000); // => 1e9
    println!("{:2E}", 1000000000); // => 1E9
}
```
#### 指针地址
```rust
let v= vec![1, 2, 3];
println!("{:p}", v.as_ptr()) // => 0x600002324050
```
### rust1.58捕获环境中的值
该功能反人类,直接捕获变量名,只能是普通的变量，更复杂的类型(比如表达式)不行
```rust
fn main() {
let (width, precision) = get_format();
for (name, score) in get_scores() {
  println!("{name}: {score:width$.precision$}");
}
```
## Debug特征
大多数rust类型都实现了Debug特征或支持派生该特征:
```
#[derive(Debug)]
struct Person {
    name: String,
    age: u8
}
fn main() {
    let i = 3.1415926;
    let s = String::from("hello");
    let v = vec![1, 2, 3];
    let p = Person{name: "sunface".to_string(), age: 18};
    println!("{:?}, {:?}, {:?}, {:?}", i, s, v, p);
}
```
## 手动实现Display特征
标准库Display特征实现没那么多,可以自己实现:
```rust
struct Person {
    name: String,
    age: u8,
}

use std::fmt;
impl fmt::Display for Person {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(
            f,
            "大佬在上，请受我一拜，小弟姓名{}，年芳{}，家里无田又无车，生活苦哈哈",
            self.name, self.age
        )
    }
}
fn main() {
    let p = Person {
        name: "sunface".to_string(),
        age: 18,
    };
    println!("{}", p);
}
```

