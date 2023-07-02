---
title: rust学习——异步
date: 2022-06-20 21:00:58
subtitle:
categories:
tags:
index_img: /img/rust.png
banner_img: /img/rust.png
---
## 基本知识与背景
- 异步编程是一个并发编程模型，允许我们同时并发运行大量的任务，却仅仅需要几个甚至一个os线程或cpu核心
- 几个并发模型:
  - 多线程：最原始，上下文切换损耗大，线程池有所缓解，不适用于io密集型
  - 事件驱动：回调-callback，javascript使用，存在回调地狱（非线性的控制流和结果处理导致了数据流向和错误传播变得难以掌控，可维护性和可读性大幅降低）
  - 协程：go语言使用，抽象层次过高，无法接触底层细节,自定义程度低
  - actor模型：erlang使用，容易实现，通过消息传递进行通信和数据传递，和分布式系统概念相似，但遇到流控制、失败重试场景就不好用
  - async：性能高，支持底层，又有一定抽象，内部机制过于复杂，使用起来不够简单
### rust中的async
- future在rust中是惰性的，future可以简单理解为在未来某个时间点被执行的任务
- rust没有内置异步调用所必须的运行时，需要三方库，如tokio
- 运行时同时支持单线程和多线程
### 原始多线程与async的选择
原始多线程适用于：
- 少量任务并发
- cpu密集型任务，如并行计算
- 部分io任务，可以用线程池
- 无所谓时，使用简单
async适用于：
- io密集型，高并发
### async的库支持
- 所必须的特征(例如Future)、类型和函数，由标准库提供实现
- 关键字async/await由Rust语言提供，并进行了编译器层面的支持
- 众多实用的类型、宏和函数由官方开发的futures包提供(不是标准库)，它们可以用于任何async应用中。
- async代码的执行、IO操作、任务创建和调度等等复杂功能由社区的async运行时提供，例如tokio和async-std
### 一个例子认识async
未改前(虽然用到async函数，但完全是同步)：
```rust
use futures::executor::block_on;

struct Song {
    author: String,
    name: String,
}

async fn learn_song() -> Song {
    Song {
        author: "曲婉婷".to_string(),
        name: String::from("《我的歌声里》"),
    }
}

async fn sing_song(song: Song) {
    println!(
        "给大家献上一首{}的{} ~ {}",
        song.author, song.name, "你存在我深深的脑海里~ ~"
    );
}

async fn dance() {
    println!("唱到情深处，身体不由自主的动了起来~ ~");
}

fn main() {
    let song = block_on(learn_song());
    block_on(sing_song(song));
    block_on(dance());
}
```
改之后:
```rust
use futures::executor::block_on;

struct Song {
    author: String,
    name: String,
}

async fn learn_song() -> Song {
    Song {
        author: "曲婉婷".to_string(),
        name: String::from("《我的歌声里》"),
    }
}

async fn sing_song(song: Song) {
    println!(
        "给大家献上一首{}的{} ~ {}",
        song.author, song.name, "你存在我深深的脑海里~ ~"
    );
}

async fn dance() {
    println!("唱到情深处，身体不由自主的动了起来~ ~");
}

async fn learn_and_sing() {
    // 这里使用`.await`来等待学歌的完成，但是并不会阻塞当前线程，该线程在学歌的任务`.await`后，完全可以去执行跳舞的任务
    let song = learn_song().await;

    // 唱歌必须要在学歌之后
    sing_song(song).await;
}

async fn async_main() {
    let f1 = learn_and_sing();
    let f2 = dance();

    // `join!`可以并发的处理和等待多个`Future`，若`learn_and_sing Future`被阻塞，那`dance Future`可以拿过线程的所有权继续执行。若`dance`也变成阻塞状态，那`learn_and_sing`又可以再次拿回线程所有权，继续执行。
    // 若两个都被阻塞，那么`async main`会变成阻塞状态，然后让出线程所有权，并将其交给`main`函数中的`block_on`执行器
    futures::join!(f1, f2);
}

fn main() {
    block_on(async_main());
}
```
- async标记的语法块会转成实现了Future特征的状态机，当Future执行并遇到阻塞时，它会让出当前线程的控制权，其他的future就可以在该线程中运行
- 需要在Cargo.toml中添加内容:
```rust
[dependencies]
futures = "0.3"
```
- `async fn`创建一个异步函数，异步函数的返回值是一个Future,Future直接调用不会输出任何结果，需要一个执行器（上面例子就是block_on）
- block_on执行器简单粗暴，会阻塞当前线程直到任务完成
- .await不会阻塞当前线程，而是异步等待future完成，该线程还可以执行其他future
## future特征
抽象化定义：它是一个能产出值的异步计算（值可能为空`()`）
简化版：
```rust
trait SimpleFuture {
    type Output;
    fn poll(&mut self, wake: fn()) -> Poll<Self::Output>;
}

enum Poll<T> {
    Ready(T),
    Pending,
}
```
若当前poll中，future可以被完成，则返回Poll::Ready,反之则返回Pending,并且安排一个wake函数，，当future准备好时，wake通知执行器执行
### Unpin
绝大多数标准库类型都实现了Unpin,但生成的Future没有实现Unpin,默认是!Unpin的，具体实践吧
## async的生命周期
如果拥有引用类型的参数，那么返回的Future的生命周期就会被这些参数的生命周期所限制，引用参数必须活得比future长
## async move
就是移到future内，不可以共享了
## .await与多线程
async语句块中的变量必须可以在线程间传递，所以必须实现Send和Sync特征，
不能使用Mutex锁，而是要四用futures::lock
## Stream特征
该特征类似于Future特征，也与迭代器类似，future包中的mpsc就实现了Stream特征：
```rust
fn main() {
async fn send_recv() {
    const BUFFER_SIZE: usize = 10;
    let (mut tx, mut rx) = mpsc::channel::<i32>(BUFFER_SIZE);

    tx.send(1).await.unwrap();
    tx.send(2).await.unwrap();
    drop(tx);

    // `StreamExt::next` 类似于 `Iterator::next`, 但是前者返回的不是值，而是一个 `Future<Output = Option<T>>`，
    // 因此还需要使用`.await`来获取具体的值
    assert_eq!(Some(1), rx.next().await);
    assert_eq!(Some(2), rx.next().await);
    assert_eq!(None, rx.next().await);
}
```
- next返回一个**future**特征，因此还需要await取
## join!宏
join!宏会返回一个元组，里面对应future执行结束后的值，宏中的future是并发：
```rust
use futures::join;

async fn enjoy_book_and_music() -> (Book, Music) {
    let book_fut = enjoy_book();
    let music_fut = enjoy_music();
    join!(book_fut, music_fut)
}
```
## try_join!
由于join!必须等待它管理的所有 Future 完成后才能完成，如果你希望在某一个 Future 报错后就立即停止所有 Future 的执行，可以使用 try_join!，特别是当 Future 返回 Result 时:
```rust
use futures::{
    future::TryFutureExt,
    try_join,
};

async fn get_book() -> Result<Book, ()> { /* ... */ Ok(Book) }
async fn get_music() -> Result<Music, String> { /* ... */ Ok(Music) }

async fn get_book_and_music() -> Result<(Book, Music), String> {
    let book_fut = get_book().map_err(|()| "Unable to get book".to_string());
    let music_fut = get_music();
    try_join!(book_fut, music_fut)
}
```
- 有一点需要注意，传给 try_join! 的所有 Future 都必须拥有相同的**错误类型**。如果错误类型不同，可以考虑使用来自 futures::future::TryFutureExt 模块的 map_err和err_info方法将错误进行转换
## select!
join!必须等所有Future结束后，才能集中处理结果，但是select!任意一个future结束，都可以立即被处理:
```rust
use futures::{
    future::FutureExt, // for `.fuse()`
    pin_mut,
    select,
};

async fn task_one() { /* ... */ }
async fn task_two() { /* ... */ }

async fn race_tasks() {
    let t1 = task_one().fuse();
    let t2 = task_two().fuse();

    pin_mut!(t1, t2);

    select! {
        () = t1 => println!("任务1率先完成"),
        () = t2 => println!("任务2率先完成"),
    }
}
```
### default和complete
- complete分支当所有的Future和Stream完成后才会被执行，它往往配合loop使用，loop用于循环完成所有的Future
- default分支，若没有任何Future或Stream处于Ready状态，则该分支会被立即执行
(ps:我这里有个疑惑，当所有future都执行完了，不就没有符合ready状态的，此时两分支不就是相同，但这个疑惑可以从另一角度避开，complete就是所有future都执行完了才匹配，default就是某个时刻没有ready的future来匹配，毕竟你自己还不懂底层，谁知道这两个是不是等价的,且只有这两个状态，所以下面代码，default我猜一般应该是continue语句搭配)
```rust
use futures::future;
use futures::select;
pub fn main() {
    let mut a_fut = future::ready(4);
    let mut b_fut = future::ready(6);
    let mut total = 0;

    loop {
        select! {
            a = a_fut => total += a,
            b = b_fut => total += b,
            complete => break,
            default => panic!(), // 该分支永远不会运行，因为`Future`会先运行，然后是`complete`
        };
    }
    assert_eq!(total, 10);
}
```
### FusedFuture和Unpin特征
这两个特征是使用select所必须的，在本小节第一段代码中，分别用`.fuse()`和`pin_mut!`来为Future添加,以下是为什么：
- Unpin，由于 select 不会通过拿走所有权的方式使用Future，而是通过可变引用的方式去使用，这样当 select 结束后，该 Future 若没有被完成，它的所有权还可以继续被其它代码使用。
- FusedFuture的原因跟上面类似，当 Future 一旦完成后，那 select 就不能再对其进行轮询使用。Fuse意味着熔断，相当于 Future 一旦完成，再次调用poll会直接返回Poll::Pending。
- Stream使用的是FusedStream,对其调用next可以获取实现了FusedFuture特征的Future
### 一个例子
```rust
use futures::{
    future::{Fuse, FusedFuture, FutureExt},
    stream::{FusedStream, Stream, StreamExt},
    pin_mut,
    select,
};

async fn get_new_num() -> u8 { /* ... */ 5 }

async fn run_on_new_num(_: u8) { /* ... */ }

async fn run_loop(
    mut interval_timer: impl Stream<Item = ()> + FusedStream + Unpin,
    starting_num: u8,
) {
    let run_on_new_num_fut = run_on_new_num(starting_num).fuse();
    let get_new_num_fut = Fuse::terminated();
    pin_mut!(run_on_new_num_fut, get_new_num_fut);
    loop {
        select! {
            () = interval_timer.select_next_some() => {
                // 定时器已结束，若`get_new_num_fut`没有在运行，就创建一个新的
                if get_new_num_fut.is_terminated() {
                    get_new_num_fut.set(get_new_num().fuse());
                }
            },
            new_num = get_new_num_fut => {
                // 收到新的数字 -- 创建一个新的`run_on_new_num_fut`并丢弃掉旧的
                run_on_new_num_fut.set(run_on_new_num(new_num).fuse());
            },
            // 运行 `run_on_new_num_fut`
            () = run_on_new_num_fut => {},
            // 若所有任务都完成，直接 `panic`， 原因是 `interval_timer` 应该连续不断的产生值，而不是结束
            //后，执行到 `complete` 分支
            complete => panic!("`interval_timer` completed unexpectedly"),
        }
    }
}
```
- fuse返回类型的set()方法好像是一个换值操作，就是这个变量重新赋值一个future
- Fuse::terminated()是一个空的Future
- FuturesUnordered类型：它会将 run_on_new_num_fut 的每一个拷贝都运行到完成，而不是像之前那样一旦创建新的就终止旧的，例子大体类似如下：
```rust
use futures::{
    future::{Fuse, FusedFuture, FutureExt},
    stream::{FusedStream, FuturesUnordered, Stream, StreamExt},
    pin_mut,
    select,
};

async fn get_new_num() -> u8 { /* ... */ 5 }

async fn run_on_new_num(_: u8) -> u8 { /* ... */ 5 }


// 使用从 `get_new_num` 获取的最新数字 来运行 `run_on_new_num`
//
// 每当计时器结束后，`get_new_num` 就会运行一次，它会立即取消当前正在运行的`run_on_new_num` ,
// 并且使用新返回的值来替换
async fn run_loop(
    mut interval_timer: impl Stream<Item = ()> + FusedStream + Unpin,
    starting_num: u8,
) {
    let mut run_on_new_num_futs = FuturesUnordered::new();
    run_on_new_num_futs.push(run_on_new_num(starting_num));
    let get_new_num_fut = Fuse::terminated();
    pin_mut!(get_new_num_fut);
    loop {
        select! {
            () = interval_timer.select_next_some() => {
                 // 定时器已结束，若`get_new_num_fut`没有在运行，就创建一个新的
                if get_new_num_fut.is_terminated() {
                    get_new_num_fut.set(get_new_num().fuse());
                }
            },
            new_num = get_new_num_fut => {
                 // 收到新的数字 -- 创建一个新的`run_on_new_num_fut` (并没有像之前的例子那样丢弃掉旧值)
                run_on_new_num_futs.push(run_on_new_num(new_num));
            },
            // 运行 `run_on_new_num_futs`, 并检查是否有已经完成的
            res = run_on_new_num_futs.select_next_some() => {
                println!("run_on_new_num_fut returned {:?}", res);
            },
            // 若所有任务都完成，直接 `panic`， 原因是 `interval_timer` 应该连续不断的产生值，而不是结束
            //后，执行到 `complete` 分支
            complete => panic!("`interval_timer` completed unexpectedly"),
        }
    }
}
```
## 一些疑难解决方法
### 在async语句块中使用?
在async fn中使用?是没问题的，因为可以指明返回值，但在语句块中要指明返回值，需要像下面这么做：
```rust
async fn foo() -> Result<u8, String> {
    Ok(1)
}
async fn bar() -> Result<u8, String> {
    Ok(1)
}
pub fn main() {
    let fut = async {
        foo().await?;
        bar().await?;
		Ok::<(), String>(())
    };
}
```
### async与send
async中并不是一定不能包含send，但它不能影响.await,可以加大括号提前drop掉，但是目前`std::mem::drop`是不能解决的：
```rust
async fn foo() {
    {
        let x = NotSend::default();
    }
    bar().await;
}
```
### async fn递归
```rust
use futures::future::{BoxFuture, FutureExt};

fn recursive() -> BoxFuture<'static, ()> {
    async move {
        recursive().await;
        recursive().await;
    }.boxed()
}
```
### 特征中使用async
每一次特征中的async函数被调用时，都会产生一次堆内存分配,具有一定的开销
```rust

use async_trait::async_trait;

#[async_trait]
trait Advertisement {
    async fn run(&self);
}

struct Modal;

#[async_trait]
impl Advertisement for Modal {
    async fn run(&self) {
        self.render_fullscreen().await;
        for _ in 0..4u16 {
            remind_user_to_join_mailing_list().await;
        }
        self.hide_for_now().await;
    }
}

struct AutoplayingVideo {
    media_url: String,
}

#[async_trait]
impl Advertisement for AutoplayingVideo {
    async fn run(&self) {
        let stream = connect(&self.media_url).await;
        stream.play().await;

        // 用视频说服用户加入我们的邮件列表
        Modal.run().await;
    }
}
```
