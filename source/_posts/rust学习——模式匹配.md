---
title: rust学习——模式匹配
date: 2022-04-11 17:33:59
subtitle:
categories:
tags:
index_img: /images/rust.png
banner_img: /images/rust.png
---
## match
- match的匹配必须要穷举所有可能，用`_`来代表未列出的所有可能
- match的每一个分支必须是一个表达式，并且所有分支的表达式最终返回值的类型必须相同
- `X|Y`用来表示逻辑或
- match本身也是表达式可以用来赋值
- match匹配不成功是不会发生所有权的转移
- `_`不会取走所得权

### 格式
```rust
match target {
    模式1 => 表达式1,
    模式2 => {
        语句1;
        语句2;
        表达式2
    },
    _ => 表达式3,
}
```
### 从模式中提取绑定值
假设action为一个绑定值的枚举类型，可以像下面这么使用
```rust
match action {
	Action::Say(s) => {
		println!("{}", s);
	},
	Action::MoveTo(x, y) => {
		println!("point from (0, 0) move to ({}, {})", x, y);
	},
	Action::ChangeColorRGB(r, g, _) => {
		println!("change color into '(r:{}, g:{}, b:0)', 'b' has been ignored",
			r, g,
		);
	}
}
```
### if let匹配
- 当只要匹配一个条件,且忽略其他条件时就用if let,否则都用match
```rust
let v = Some(3u8);
match v{
	Some(3) => println!("three"),
	_ =>(),
}
```
等价于
```rust
if let Some(3) = v {
    println!("three");
}
```
- 提取Option值
`let Some(x) = some_option_value;`会编译出错，因为没覆盖None  
`if let Some(x) = some_option_value{}`应该这样，但是x的作用域仅限花括号内
### while let匹配
和if let类似,一个十分有用的例子：
```rust
while let Some(top) = stack.pop() {
    println!("{}", top);
}
```
### matches!宏
该宏将一个表达式跟模式进行匹配，然后返回true或false  
`v.iter().filter(|x| x == MyEnum::Foo);`：无法编译通过，无法将x直接跟一个枚举成员进行比较,应该如下：
`v.iter().filter(|x| matches!(x,MyEnum::Foo));`
更多例子:
`assert!(matches!(foo, 'A'..='Z' | 'a'..='z'));`
`assert!(matches!(bar, Some(x) if x > 2));`
## 解构结构体
```
let p = Point {x:0,y:7};
//更常用 let Point {x,y} = p;
let Point {x:a,y:b} = p;
assert_eq!(0,a);
assert_eq!(7,b);
```
## `_`
- 任何模式匹配的结构赋值都可能发生所有权的转移，但`_`不会发生
## `..`忽略剩余的值
```rust
let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x),
}
```
## 匹配守卫-额外的if条件
例子：
```rust
let num = Some(4);
match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
}
```
- 匹配守卫条件会作用于所有的模式，无法使用括号来改变优先级，因为这不是运算符
`4 | 5 | 6 if y`用括号(这仅是方便理解，无法用括号改变优先级)表示就是`(4| 5 | 6) if y`
## @绑定
### 绑定部分字段
- 匹配到的模式绑定到一个变量中,允许为一个字段绑定到另外一个变量
```rust
enum Message {
    Hello { id: i32 },
}
let msg = Message::Hello { id: 5 };
match msg {
    Message::Hello { id: id_variable @ 3..=7 } => {
		println!("Found an id in range: {}", id_variable)
	},
	_ =>{...},
```
上例可以这么理解通过在`3..=7`之前指定`id_variable @`,我们捕获了任何匹配此范围的值并同时将该值绑定到变量id_variable
### 前绑定后结构
- 下例字段都是基本形没发生所有权转移，所以这么用,没有copy特性的用起来还挺蛋疼
- 可以在绑定新变量的同时，对目标进行解构
```rust
    let p @ Point {x: px, y: py } = Point {x: 10, y: 23};
    println!("x: {}, y: {}", px, py);
    println!("{:?}", p);
```
### 1.53版本后新增
绑定可以使用括号，下例不加括号会没有绑定到所有模式，只绑定到了1,编译不通过:
```rust
fn main() {
    match 1 {
        num @ (1 | 2) => {
            println!("{}", num);
        }
        _ => {}
    }
}
```
## tips
- 使用下划线开头可以使编译器忽略未使用的变量
- 可以使用序列
