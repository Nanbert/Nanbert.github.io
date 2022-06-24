---
title: rust学习——测试
date: 2022-06-18 19:04:00
subtitle:
categories:
tags:
cover:
---
## 介绍
lib型的项目会生产出以下代码，这就是一个基本的test demo
```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```
上面代码中的 #[cfg(test)] 标注(本质是条件编译)可以告诉 Rust 只有在 cargo test 时才编译和运行模块 tests，其它时候当这段代码是空气即可
### 单元测试
单元测试目标是测试某一个代码单元(一般都是函数)，验证该单元是否能按照预期进行工作，例如测试一个 add 函数，验证当给予两个输入时，最终返回的和是否符合预期。 在 Rust 中，单元测试的惯例是将测试代码的模块跟待测试的正常代码放入同一个文件中
```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(add_two(2), 4);
    }
}
```
### 集成测试
集成测试的代码在一个单独目录tests下，cargo会自动在此目录下寻找集成测试文件，cargo会对该文件夹下每个文件进行自动编译。
- 需要将待测试的包引入到当前包的作用域
- 无需再使用`#[cfg(test)]`
- 可以通过选项--test来运行指定的集成测试用例
- tests文件夹下每一个文件都是一个独立的包
- tests文件夹下的子目录中的文件，不会被当作独立的包，通常用来作共享模块(如：不是创建`tests/common.rs`,而是`tests/common/mod.rs`)
- 目前来说只支持lib类型的包集成测试
### 基准测试benchmark
目前只有nightly版支持：`rustup install nightly`
```rust
#![feature(test)]

extern crate test;

fn fibonacci_u64(number: u64) -> u64 {
    let mut last: u64 = 1;
    let mut current: u64 = 0;
    let mut buffer: u64;
    let mut position: u64 = 1;

    return loop {
        if position == number {
            break current;
        }

        buffer = last;
        last = current;
        current = buffer + current;
        position += 1;
    };
}
#[cfg(test)]
mod tests {
    use super::*;
    use test::Bencher;

    #[test]
    fn it_works() {
       assert_eq!(fibonacci_u64(1), 0);
       assert_eq!(fibonacci_u64(2), 1);
       assert_eq!(fibonacci_u64(12), 89);
       assert_eq!(fibonacci_u64(30), 514229);
    }

    #[bench]
    fn bench_u64(b: &mut Bencher) {
        b.iter(|| {
            for i in 100..200 {
                test::black_box(fibonacci_u64(i));
            }
        });
    }
}
```
cargo bench运行
必须使用black_box,否则编译器会优化掉（因为结果没有用到），结果就会0ns
#### 三方库criterion.rs

## 断言
- assert!,assert_eq!,assert_ne!,它们会在所有模式下运行
- debug_assert!,debug_assert_eq!,debug_assert_ne!,它们只会在Debug模式下运行
### assert_eq！
判断两个表达式返回的值是否相等
- `assert_eq!(a,b)`
- `assert_eq!(a, b, "我们在测试两个数之和{} + {}，这是额外的错误信息", a, b)`
### assert_ne!
判断两个表达式是否不等
### assert!
用于判断bool表达式是否为true
## 常用属性
### [should_panic]
```rust
#[should_panic]
fn s(){
	panic!("sk");
}
```
还可以接受报错信息，与报错信息不一样的认为失败，只要前缀匹配即可
```rust

#![allow(unused)]
impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!(
                "Guess value must be greater than or equal to 1, got {}.",
                value
            );
        } else if value > 100 {
            panic!(
                "Guess value must be less than or equal to 100, got {}.",
                value
            );
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "Guess value must be less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```
### [ignore]
必须使用`--ignored`选项才运行的测试
## 自定义Result<T,E>
可以手动写
```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```
## cargo test命令
该命令可以运行单元测试，集成测试，文档测试
### --分割命令行参数
- 第一种是提供给 cargo test 命令本身的，这些参数在 -- 之前指定,`cargo test --help`来查看帮助列表
- 第二种是提供给编译后的可执行文件的，在 -- 之后指定,可以用`cargo test -- --help`来查看帮助列表
### 第一种选项
- --no-run:只生成编译文件，不运行
- --test:指定运行集成测试
#### 指定测试函数
- 当运行指定一个测试时，直接跟函数名
- 名称过滤,如`cargo test add`运行所有前缀为add的测试函数，同时可以通过模块名进行过滤：如`cargo test tests::add`

### 第二种选项
- test-threads:指定运行测试的线程数，当你想顺序执行测试可以指定为1
- show-output:默认情况test函数里println不输出，指定该选项才能看到输出
- ignored:运行被忽略的测试函数
