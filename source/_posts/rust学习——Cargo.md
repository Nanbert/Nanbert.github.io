---
title: rust学习——Cargo
date: 2022-04-09 18:01:07
subtitle:
categories:
tags:
cover:
---

## 常用命令
- 创建项目:`cargo new <项目名>`
- 编译并运行项目：`cargo run`
- 编译项目：`cargo build`
- 检查项目是否编译通过:`cargo check`
## 常用选项
- --release:编译优化，可能增加编译时间
## Cargo.toml
- 作用：是项目数据描述文件。存储了项目的所有元配置信息。
### [package]
|关键字|作用|例子|
|name|项目名|`name = "world_hello"`|
|version|项目版本|`version = "0.1.0"`|
|edition|rust大版本|`edition = "2021"`|
### [dependencies]
- 主要有三种依赖：
  1) 基于Rust官方仓库`crates.io`,通过版本来说明：`rand = "0.3"`或`hammer = { version = "0.5.0" }`
  2) 基于项目源代码的git仓库地址，通过url：`color = { git = "https://github.com/bjz/color-rs" }`
  3) 基于本项目的绝对路径或相对路径：`geometry = { path = "crates/geometry" }`
### [source]更换镜像
- 更改中科大
```bash
[source.crates-io]
replace-with = 'ustc'

[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
```
## Cargo.lock
- 功能作用：根据toml文件生成的项目依赖详细清单
- 该不该上传git：一个可运行项目就上传，一个依赖库则添加到`.gitignore`

