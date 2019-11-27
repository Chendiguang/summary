# 初识Rust

安装Rust和替换国内cargo源

## [安装](https://www.rust-lang.org/tools/install)

在windows和类Unix上安装的情况大同小异。推荐使用```rustup```，这个会安装所有的依赖管理工具。

* 依赖

    rust必须由c++的编译环境。从导入包的语法来看，这两者是很相似的。

* windows

    直接下载[RUSTUP-INIT.exe](https://win.rustup.rs/)执行

* Unix

    直接在shell执行

    ```bash
    curl -sSf https://static.rust-lang.org/rustup.sh | sh
    ```

## 替换国内源

编译依赖包的问题都会涉及到源，尤其是在国内编程。在这一点上，cargo是一个比较优雅的管理工具，在go里面是mod。

* 替换源

    初次替换源，需要新建$HOME/.cargo/config(与平台无关)文件，然后在里面添加一下内容。

    ```bash
    [source.crates-io]
    registry = "https://github.com/rust-lang/crates.io-index"
    replace-with = 'ustc'
    [source.ustc]
    registry = "git://mirrors.ustc.edu.cn/crates.io-index"
    ```

* crate管理的依赖

    rust的依赖在cargo.toml里面添加后，直接执行：```cargo update或cargo build```即可。这是一个依赖清单。

    ```toml
    [package]
    name = "guessing_game"
    version = "0.1.0"
    authors = ["pipshq <1334415215@qq.com>"]
    edition = "2018"

    # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

    [dependencies]  # 在此添加依赖
    rand = "0.5.5"
    ```

* Cargo.lock文件

    crate更新完成后，可以在Cargo.lock文件观察到新的更新。这个文件```由cargo维护，且不应手动编辑```。
