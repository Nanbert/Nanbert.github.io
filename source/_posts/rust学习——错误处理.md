---
title: rust学习——错误处理
date: 2022-05-22 12:49:51
subtitle:
categories:
tags:
cover:
---
## Result
### 定义
```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
### unwrap和expect
- unwrap如果成功，就将Ok(T)中的值取出来，如果失败就直接panic
- expect和unwrap类似，但遇到错误可以自定义提示信息
### Option和Result组合器方法
#### or()和and()
- s1.or(s2),若s1为some,则表达式为s1。。。
- s1.and(s2),若其中有一个为None,则表达式为None
#### or_else()和and_then()
和or()及and()类似，但接受一个闭包。
```rust
    let fn_some = |_| Some("some2"); // 类似于: let fn_some = |_| -> Option<&str> { Some("some2") };

    let n: Option<&str> = None;
    let fn_none = |_| None;

    assert_eq!(s1.and_then(fn_some), s2); // Some1 and_then Some2 = Some2
    assert_eq!(s1.and_then(fn_none), n);  //
```
#### filter()
接受一个返回bool的闭包，为None或返回false则为None
- `s1.filter(fn_is_even),n`
#### map()和map_err()
将值映射为另一个，map()是用来映射Ok或Some,map_err()是用来映射Err,
#### map_or()和map_or_else()
和map差不多，不同的是map_or()遇到None返回一个默认值,map_or_else()则接受一个闭包
#### ok_or()和ok_or_else()
将Option类型转换成Result类型，ok_or()接受一个错误作为默认类型，ok_or_else()则接受一个闭包
## 错误传播`?`
- `?`需要一个变量来承载正确的值，只有错误值是直接返回，正确值不行，所以如下代码是不对的
```rust
fn first(arr: &[i32]) -> Option<&i32> {
   arr.get(0)?
}
```
只有以下两种使用形式：
  - `let v = xxx()?;`
  - `xxx()?.yyy()?;`
- 一个例子理解：
```rust
let mut f = File::open("hello.txt")?;
//等价于
let mut f = match f {
	Ok(file) => file,
	//向上传播
	Err(e) => return Err(e),
};
```
- 可以链式调用
- 可以应用于Option
## 自定义错误
- 最高层次抽象错误，可以使用特征对象`Result<(), Box<dyn Error>>`
- 自定义错误类型只需实现Debug和Display特征
### std::error::Error特征
### 使用From特征将标准库错误转成自定义
```rust
use std::fs::File;
use std::io::{self, Read};
use std::num;

#[derive(Debug)]
struct AppError {
    kind: String,
    message: String,
}

impl From<io::Error> for AppError {
    fn from(error: io::Error) -> Self {
        AppError {
            kind: String::from("io"),
            message: error.to_string(),
        }
    }
}

impl From<num::ParseIntError> for AppError {
    fn from(error: num::ParseIntError) -> Self {
        AppError {
            kind: String::from("parse"),
            message: error.to_string(),
        }
    }
}

fn main() -> Result<(), AppError> {
    let mut file = File::open("hello_world.txt")?;

    let mut content = String::new();
    file.read_to_string(&mut content)?;

    let _number: usize;
    _number = content.parse()?;

    Ok(())
}
```
#### thiserror便捷的库
该库可以便捷的转换标准库的错误
```rust
use std::fs::read_to_string;

fn main() -> Result<(), MyError> {
  let html = render()?;
  println!("{}", html);
  Ok(())
}

fn render() -> Result<String, MyError> {
  let file = std::env::var("MARKDOWN")?;
  let source = read_to_string(file)?;
  Ok(source)
}

#[derive(thiserror::Error, Debug)]
enum MyError {
  #[error("Environment variable not found")]
  EnvironmentVariableNotFound(#[from] std::env::VarError),
  #[error(transparent)]
  IOError(#[from] std::io::Error),
}
```
