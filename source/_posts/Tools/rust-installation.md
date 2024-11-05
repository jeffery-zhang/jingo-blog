---
title: Rust安装和更新
description: 在Windows中安装和更新rust, 并且配置墙内镜像源
date: 2022-05-21 15:12:12
categories:
  - Coding
tags:
  - Rust
  - Rustup
  - Cargo
  - 镜像源
---

# Rust 安装和更新

在 Windows 中安装和更新 rust, 并且配置墙内镜像源

## 安装 rustup

rustup 工具链需要 C++ 编译支持, 首先需要确保安装了 C++ 编译环境, 可优先下载安装[Visual Studio](https://visualstudio.microsoft.com/zh-hans/visual-cpp-build-tools/)

确保安装了 C++ 编译环境后, 下载并安装[Rustup](https://www.rust-lang.org/tools/install)

## 配置 rustup 工具链镜像

修改环境变量 RUSTUP_DIST_SERVER (默认值为 [https://static.rust-lang.org](https://static.rust-lang.org)) 和 RUSTUP_UPDATE_ROOT (默认值为 [https://static.rust-lang.org/rustup](https://static.rust-lang.org/rustup))的值为墙内镜像源:

```sh
# 清华大学
RUSTUP_DIST_SERVER=https://mirrors.tuna.tsinghua.edu.cn/rustup

# 中国科学技术大学
RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup

# 上海交通大学
RUSTUP_DIST_SERVER=https://mirrors.sjtug.sjtu.edu.cn/rust-static/
```

## 配置 Cargo 镜像源

打开 C:\Users\<用户名>\.cargo\config 文件, 如果没有则新建一个, 输入以下内容:

```toml
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"

# 替换成要使用的镜像
replace-with = 'rsproxy'

# 中国科学技术大学
[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
# 如果所处的环境中不允许使用 git 协议，可以把上述地址改为 https 协议
#registry = "https://mirrors.ustc.edu.cn/crates.io-index"

# 清华大学
[source.tuna]
registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"

# 上海交通大学
[source.sjtu]
registry = "https://mirrors.sjtug.sjtu.edu.cn/git/crates.io-index"

# rustcc 社区
[source.rustcc]
registry = "git://crates.rustcc.cn/crates.io-index"

# rsproxy
[source.rsproxy]
registry = "https://rsproxy.cn/crates.io-index"
[source.rsproxy-sparse]
registry = "sparse+https://rsproxy.cn/index/"
[registries.rsproxy]
index = "https://rsproxy.cn/crates.io-index"

[net]
git-fetch-with-cli=true
```

如果更换 cargo 源后执行 cargo build 命令出现如下错误:

```sh
blocking waiting for file lock on package cache lock
```

可以删除 C:\Users\<用户名>\.cargo\.package-cache 文件

## rust 升级

检查是否有更新和升级 rustup

```sh
rustup check

rustup update
```
