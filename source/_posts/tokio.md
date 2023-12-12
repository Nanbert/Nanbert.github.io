---
title: tokio
date: 2022-06-27 21:04:09
subtitle:
categories:
tags:
index_img: /images/tokio.png
banner_img: /images/tokio.png
---
## 不适用场景
- 并行运行CPU密集型的任务（并行计算）
- 读取大量的文件
- 发送少量HTTP请求
## async main
异步main函数的意义：
- .await只能在async函数中使用
- 异步运行时本身需要初始化，`#[tokio::main]`宏将`async fn main`隐式转换为`fn main`，同时还对整个异步运行时进行了初始化：
```rust
#[tokio::main]
async fn main() {
    println!("hello");
}
```
转化为：
```rust
fn main() {
    let mut rt = tokio::runtime::Runtime::new().unwrap();
    rt.block_on(async {
        println!("hello");
    })
}
```
