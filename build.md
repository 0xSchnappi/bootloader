# Rust构建脚本build.rs

一些项目希望编译第三方的非 Rust 代码，例如 C 依赖库；一些希望链接本地或者基于源码构建的 C 依赖库；还有一些项目需要功能性的工具，例如在构建之间执行一些代码生成的工作等。

Cargo会将构建脚本编译成一个可执行文件，然后运行该文件并执行相应的任务。

build.rs条件编译主要分为两个方向，分别是bios和uefi，



## 条件编译

```json
// Cargo.toml
[features]
default = ["bios", "uefi"]
bios = ["dep:mbrman"]
uefi = ["dep:gpt"]

```

```rust
// build.rs
fn main() {
    #[cfg(not(feature = "uefi"))]
    async fn uefi_main() {}
    #[cfg(not(feature = "bios"))]
    async fn bios_main() {}

    block_on((uefi_main(), bios_main()).join());
}
```

> 1. 条件编译
>    `cfg`宏和 `feature`共同完成了条件编译，`#[cfg(not(feature = "uefi"))]`表示如果如果Cargo.toml文件中没有定义uefi，就执行 `async fn uefi_main() {}`代码。启用uefi条件编译是方式是 `uefi = []`
> 2. async关键字
>
>    async是Rust异步编程中创建Future对象使用的：
>
>    定义函数：`async fn`
>
>    定义block：`async {}`
>
>    Future不会立即执行，要想执行Future定义的函数：
>
>    使用 `await`
>
>    在异步运行时为该Future创建task
>
>    创建异步任务：
>
>    使用 `block_on`
>
>    使用 `spawn`
>
>    `async fn uefi_main() {}`代码作用就是在这里创建一个空函数，因为作者的意图是可以编译单独uefi版本，也可以单独编译bios版本，还可以同时编译uefi和bios版本。
> 3. 异步任务 `block_on`
>
>    此函数是用于创建异步任务的，，阻塞当前线程，直到异步任务完成
> 4. 依赖
>
>    `uefi = ["dep:gpt"]`编译uefi时依赖于gpt，需要到 `[dependencies]`去查看具体定义
>
>    ```json
>    [dependencies]
>    gpt = { version = "3.0.0", optional = true }[dependencies]
>    gpt = { version = "3.0.0", optional = true }
>    ```
>    这种依赖写法会自动定义一个与依赖同名的feature，也就是  `gpt `feature，也就是当我们启用 `gpt`feature时，该依赖库也会被自动引入并启用。
