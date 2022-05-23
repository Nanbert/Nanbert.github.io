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
最高层次抽象错误，可以使用特征对象`Result<(), Box<dyn Error>>`
