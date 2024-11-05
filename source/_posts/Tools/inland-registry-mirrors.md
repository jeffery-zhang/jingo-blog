---
title: 国内镜像源(NPM, Cargo, Rustup)
description: 记录各种包管理工具的国内镜像源 url 与设置方式
date: 2022-05-01 13:02:36
categories:
  - Tools
tags:
  - 镜像
  - NPM
  - Cargo
  - Rustup
---

# 国内镜像源(NPM, Cargo, Rustup)

记录各种包管理工具的国内镜像源 url 与设置方式

## NPM

```sh
https://registry.npmmirror.com
```

```sh
npm config set registry https://registry.npmmirror.com
```

```sh
npm i -g pnpm --registry=https://registry.npmmirror.com
```

## PIP

```sh
# 阿里云
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/

# 豆瓣
pip config set global.index-url https://pypi.douban.com/simple/

# 清华大学
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple/

# 中国科学技术大学
pip config set global.index-url https://pypi.mirrors.ustc.edu.cn/simple/
```

## Rustup

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

## Cargo

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
