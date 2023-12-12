---
title: rust学习——Cargo
date: 2022-04-09 18:01:07
subtitle:
categories:
tags:
index_img: /images/rust.png
banner_img: /images/rust.png
---
## Package(项目)、Crate(包)和Module(模块)
- Package项目:包含独立的Cargo.toml文件，包含多个包，但只能包含一个库类型的包，可以包含多个二进制可执行类型的包
- Crate包:是个独立的可编译单元，编译后会生成一个可执行文件或一个库,是一个模块树
### cargo new到底干了啥
- 不加任何选项，cargo为我们创建了一个名称是`my-project`的package,同时创建了Cargo.toml,虽然该文件没有提到`src/main.rs`为程序的入口，但Cargo有个惯例:**src/main.rs是二进制包的根文件，该二进制包的包名跟所属Package相同，这里都是my-project**,所有代码执行都从该文件中的main函数开始
- 与`src/main.rs`一样，如果一个Package包含src/lib.rs,意味这包含一个库类型的同名包，该包的根文件为`src/lib.rs`
### rust工程项目结构
```rust
.
├── Cargo.lock
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── main.rs
│   └── bin/
│       ├── named-executable.rs
│       ├── another-executable.rs
│       └── multi-file-executable/
│           ├── main.rs
│           └── some_module.rs
├── benches/
│   ├── large-input.rs
│   └── multi-file-bench/
│       ├── main.rs
│       └── bench_module.rs
├── examples/
│   ├── simple.rs
│   └── multi-file-example/
│       ├── main.rs
│       └── ex_module.rs
└── tests/
    ├── some-integration-tests.rs
    └── multi-file-test/
        ├── main.rs
        └── test_module.rs
```
- 唯一库包：`src/lib.rs`
- 默认二进制包：`src/main.rs`，编译后生成的可执行文件与Package同名
- 其余二进制包：`src/bin/main1.rs...`它们会生成与文件同名的二进制可执行文件
- 集成测试文件：tests目录
- 基准性能测试文件:benches目录
- 项目示例：examples目录
### 包根crate root
src/main.rs和src/lib.rs被成为包根，这是由于两个文件的内容形成了一个模块**crate**，假设lib.rs的内容如下：
```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```
则模块树则如下:
```shell
crate
 └── eat_at_restaurant
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```
### 模块
- 父模块完全无法访问子模块的私有项，但是子模块却可以访问父模块，爷爷。。。的模块私有项
- 绝对路径，从包根开始以crate(`/`)为开头
- 相对路径，从当前模块开始以self(`.`),super(`..`),或当前模块标识符开头
### 模块与文件分离
src/lib.rs:
```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```
与下面其实等价：
src/front_of_house.rs:
```rust
pub mod hosting{
	pub fn add_to_waitlist(){}
}
```
src/lib.rs
```rust
mod front_of_house;

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```
**注意：**`mod front_of_house`把该文件中模块内容加载进来，我们可以认为模块front_of_house的定义还在src/lib.rs,只不过具体内容在src/front_of_house.rs
### 模块可见性
- `pub`可见性无限制
- `pub(crate)`表示在当前包可见
- `pub(self)`在当前模块可见
- `pub(super)`在父模块可见
- `pub(in <path>)`表示在某个路径代表的模块中可见，其中path必须是父模块或祖先模块
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
- 功能作用：根据toml文件生成的项目依赖详细清单,锁住构建时的版本信息
- 该不该上传git：一个可运行项目就上传，一个依赖库则添加到`.gitignore`
## cargo命令
### 更新依赖
- 更新所有依赖:`cargo update`
- 只更新`regex`:`cargo update-p regex`
### 测试`cargo test`
- 功能：他会在`src/`底下寻找单元测试，也会在`tests/`目录下寻找集成测试,同时还会编译`examples/`下的示例文件，以及文档中的示例
## cargo缓存
- 构建时，cargo会将已下载的依赖放在`CARGO_HOME`目录下，默认是`$HOME/.cargo/`
### 文件
- config.toml:全局配置文件
- credentials.toml:提供私有化登陆证书，用于package注册中心，例如crates.io
- .crates.toml,.crates2.json:包含了cargo install安装包的package信息
### 目录
- bin:包含了通过cargo install或rustup下载的包编译出的可执行文件。
- git存储了`Git`的资源文件
  - git/db:当一个包依赖某个git仓库时，Cargo会将该仓库克隆到git/db目录下，如果未来需要还会对其进行更新
  - git/checkouts:若指定了git源和commit，那相应的仓库就会从git/db中checkout到该目录下，因此同一个仓库的不同checkout共存成为了可能性
- registry:包含了注册中心（如crates.io）的元数据和packages
  - registry/index:一个git仓库，包含了注册中心的所有可用包的元数据（版本、依赖等）
  - registry/cache:保存了已下载的依赖，以gzip的压缩格式，后缀名`.crate`
  - registry/src:若一个已下载的`.crate`档案被一个package所需要，该档案会被解压到`registry/src`文件夹下，最终rustc在其中找到所需要的.rs文件
### ci的考虑
应该保留缓存下列目录：
- bin
- registry/index
- registry/cache
- git/db
## 构建缓存-target目录
- target目录结构取决于是否使用--target标志为特定的平台构建，不使用则使用宿主机架构
- target下的每个子目录包含了相应的发布配置profile的构建结果，release和debug是自带的profile，见下表

|目录|描述|
|:-:|:-:|
|target/debug/|包含了 dev profile 的构建输出(cargo build 或 cargo build --debug)|
|target/release/|release profile 的构建输出，cargo build --release|
|target/foo/|自定义 foo profile 的构建输出，cargo build --profile=foo|

出于历史原因：
- dev和test profile的构建结果都放在debug下
- release和bench profile则放在release目录下
### 常见目录说明

|目录|描述|
|:-:|:-:|
|target/debug/|包含编译后的输出，例如二进制可执行文件、库对象|
|target/debug/examples/|包含示例对象|
|target/doc/|包含cargo doc生成的文档|
|target/package/|cargo package或cargo publish生成的输出|
|target/debug/deps|依赖和其他输出成果|
|target/debug/incremental|rustc增量编译的输出，该缓存可以用于提升后续的编译速度|
|target/debug/build/|构建脚本的输出|

### 依赖信息文件
每个编译成果旁边有个依赖信息文件，文件后缀是`.d`。类似Makefile
