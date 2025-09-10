+++
title = "Rust 引用下的成员不能发生 move"
date = "2025-04-20T00:00:00+08:00"
categories = ["学习Rust"]
tags = ["Rust", "Rust组合器"]
+++

## 引用下的成员不能发生 move

For example:

```rust
struct Foo {i: i32, string: String}

impl Foo {
    fn get_data(&self) -> i32 {
        return self.i;                  // 发生 Copy。
    }

    fn get_string(&self) -> String {
        return self.string;             // error. 引用的成员不能发生 Move。
    }

    fn move_string(self) -> String {    // 传拥有资源的标志符 self 才可以
        return self.string
    }
}
fn main() {
}
```

可见 Rust 比 C++ 在引用方面做了一些更加严格的限制。
