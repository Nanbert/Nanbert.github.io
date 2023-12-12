---
title: rust学习——标准库
date: 2022-05-19 13:37:35
subtitle:
categories:
tags:
index_img: /images/rust.png
banner_img: /images/rust.png
---
## Vec
### 两种初始化
- `let v = vec![1,2,3];`
- `let mut v = Vec::new();v.push(1);`
### 添加元素
- `v.push(1)`
### 读取元素
- 下标：`let third= &v[2]`
- get方法：
```rust
match v.get(2) {
    Some(third) => println!("第三个元素是 {}", third),
    None => println!("去你的第三个元素，根本没有！"),
}
```
### 迭代遍历
和数组一样
`for i in &mut v`
## HashMap
要引入`use std::collections::HashMap`
### 初始化
- `let mut my_gems:HashMap<&str,i32> = HashMap::new()`
- 从其他类型转换：
```rust
    let teams_list = vec![
        ("中国队".to_string(), 100),
        ("美国队".to_string(), 10),
        ("日本队".to_string(), 50),
    ];

    let teams_map: HashMap<_,_> = teams_list.into_iter().collect();

    println!("{:?}",teams_map)
```
### 所有权问题
key和value若没实现Copy特征，所有权将转移到HashMap中
### 插入更新值
- 插入或更新：`kv.insert(key,value)`
- 查询，不存在则插入,返回的是&mut引用：`let v = scores.entry(key).or_insert(value)`
### 通过get方法获取元素
- 返回一个Option<&xx>类型
### 遍历
```rust
for (key,value) in &scores{

}
```
### 键的限制
只有实现`std::cmp::Eq`特证的类型才能用作键
## std::mem
### std::mem::drop
手动将某值从内存中移除
### std::mem::size_of_val()
取得所占内存大小,也可以获得动态的切片大小
### mem::transmute<T,U>
将类型T直接转换成类型U,只要他们字节数相同
- 应用举例:
  - 将裸指针转变成函数指针:
  ```rust
	fn foo() -> i32 {
		0
	}

	let pointer = foo as *const ();
	let function = unsafe { 
		// 将裸指针转换为函数指针
		std::mem::transmute::<*const (), fn() -> i32>(pointer) 
	};
	assert_eq!(function(), 0);
  ```
  - 延长生命周期，或缩短一个静态生命周期寿命
  ```rust
	struct R<'a>(&'a i32);

	// 将 'b 生命周期延长至 'static 生命周期
	unsafe fn extend_lifetime<'b>(r: R<'b>) -> R<'static> {
		std::mem::transmute::<R<'b>, R<'static>>(r)
	}

	// 将 'static 生命周期缩短至 'c 生命周期
	unsafe fn shorten_invariant_lifetime<'b, 'c>(r: &'b mut R<'static>) -> &'b mut R<'c> {
		std::mem::transmute::<&'b mut R<'static>, &'b mut R<'c>>(r)
	}
  ```
###`mem::transmute_copy<T,U>
从T中拷贝出U类型所需的字节数，然后转换成U
## 数值类型
### 最大最小值
`let a = i8::MAX`




