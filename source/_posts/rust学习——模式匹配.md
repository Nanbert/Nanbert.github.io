---
title: rust学习——模式匹配
date: 2022-04-11 17:33:59
subtitle:
categories:
tags:
cover:
---
## match
- match的匹配必须要穷举所有可能，用`_`来代表未列出的所有可能
- match的每一个分支必须是一个表达式，并且所有分支的表达式最终返回值的类型必须相同
- `X|Y`用来表示逻辑或
- match本身也是表达式可以用来赋值

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
和if let类似
### matches!宏
该宏将一个表达式跟模式进行匹配，然后返回true或false  
`v.iter().filter(|x| x == MyEnum::Foo);`：无法编译通过，无法将x直接跟一个枚举成员进行比较,应该如下：
`v.iter().filter(|x| matches!(x,MyEnum::Foo));`
更多例子:
`assert!(matches!(foo, 'A'..='Z' | 'a'..='z'));`
`assert!(matches!(bar, Some(x) if x > 2));`
