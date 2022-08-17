---
title: rust学习笔记-所有权
date: 2021-08-22 09:12:10
tags: rust
category: 编程语言
---
### 所有权的基本规则
 - Each value in Rust has a variable that’s called its owner.
 - There can only be one owner at a time.
 - When the owner goes out of scope, the value will be dropped.

可以从基本规则出发，推导其他的规则。

### 变量owner的move
```rust
    let s = String::from("abc");
    let a = s; //所有权转移
    println!("{}", s) // 这行代码会报错， 因为s已经失效
```
因为一个变量的owner goes out of scope时, 变量被被释放掉, 所以一个变量不允许同时有两个owner, 否则会有double free的问题。

但是对于基本数据类型情况有些不一样。
```rust
fn main() {
    let x = 1;
    let y = x; 这里并没有所有权转移， 而是拷贝, x和y是两个变量
    println!("{},{} ", x, y) // 这里并不会报错, x仍然有效
}
```
基本数据类型的 `let y = x`是直接拷贝, 所有并没有所有权的转移。

### 所以权与函数

```rust
fn main() {
    let s = String::from("abc");
    take_ownership(s); // s的所有权在这里转移给 some_string
    println!("{}", s); // s已经失效, 所以这行代码错误
}

fn take_ownership(some_string:String){
    println!("{}", some_string) //some_string在函数执行完会被释放掉
}
```
同理, 如果是基本类型作为函数参数, 因为是值拷贝, 也不需要考虑所有权的问题。

### 引用
**引用的基本规则**:
- At any given time, you can have either one mutable reference or any number of immutable references.
- References must always be valid.

引用并不获取变量的所有权, 所以
- 对于一个变量, 可以有多个引用的变量
- 引用变量goes out of scope时, 不会释放对应内存
- 引用的变量s 如果是只读变量, 那么引用也是只读变量。
```rust
fn main() {
    let s = String::from("abc"); //默认是只读变量, 不允许s.push_back等写操作。
    let x = &s; //引用s, 所有权并没有move
    println!("{},{}", s,x) //两个变量都有效
}
```

### 可写引用
```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
let r3 = &mut s; // BIG PROBLEM

println!("{}, {}, and {}", r1, r2, r3); //
```
r1和和r2认为&s是只读的, 但是r3使用了可写引用, 所以这里报错。
稍微修改一下
```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
println!("{} and {}", r1, r2);
// r1 and r2 are no longer used after this point

let r3 = &mut s; // no problem
println!("{}", r3);
```
r3初始化后并没有再使用r1和r2, 所以这段代码没问题。


```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &mut s;

    s.push_str("acv"); //调用s的push_xx函数, 其实也是mutable borrow
    println!("{}", r1);
}
```
这里mutable borrow了两次, 所以会报错
### 总结
- 一个变量只能有一个owner, owner goes out of scope, 变量会释放。
- 基本数据类型是拷贝, 所以不存在owner转移的问题。
- 引用并不获取变量的所有权。在一个scope里, 可以有多个普通引用和一个写引用。
