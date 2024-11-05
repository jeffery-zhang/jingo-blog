---
title: 用tauri将前端页面打包成桌面应用
description: 详细介绍如何使用tauri将web前端项目打包为一个桌面APP
date: 2022-04-02 13:41:13
categories:
  - Web
tags:
  - Tauri
  - Rust
  - 前端应用
  - 桌面应用
---

## tauri 介绍

Tauri 是一款使用 rust 开发的应用构建工具包, 让你能够为使用 Web 技术的所有主流桌面操作系统构建软件

## 安装 tauri

### windows

1. 安装 rust

   前往[https://www.rust-lang.org/zh-CN/tools/install](https://www.rust-lang.org/zh-CN/tools/install)下载并安装 rust 运行环境

2. 在项目中加入 tauri

   安装 tauri cli 工具, 并执行创建命令

   ```bash
   pnpm add -D @tauri-apps/cli

   pnpm tauri init
   ```

   接下来 tauri 会向你提出几个问题, 回答之后会在你的项目根目录加入一个 src-tauri 文件夹, 其中包含了 tauri 的核心运行代码和配置文件, 比较关键的文件如下:

   - Cargo.toml

     Cargo 清单, 声明了当前应用依赖的 rust 包和元数据等内容

   - tauri.conf.json

     项目构建配置文件, 可高度自定义 tauri 应用的各方面, 可参考[Tauri 的 API 配置](https://tauri.app/zh-cn/v1/api/config)

   - src/main.rs

     rust 程序入口, 其中的 main 函数就是运行时的入口函数

3. 开发环境

   在 tauri.conf.json 中可以配置开发环境监听的本地端口和开启开发环境之前需要执行的命令(如一些项目中通常会执行 pnpm dev 来开启本地开发环境)

   ```json
   {
     "build": {
       "beforeDevCommand": "pnpm dev",
       "devPath": "http://localhost:8888"
     }
   }
   ```

   执行上述步骤后, 现在你可以执行以下命令来开启开发环境

   ```bash
   pnpm tauri dev
   ```

   执行完毕后会打开一个窗口, 此时就能看到你的前端界面了

4. 打包应用

   打包之前需要先进行一些配置, 比如配置构建应用之前需要执行的打包操作, 以及打包好的文件输出的目录:

   ```json
   {
     "build": {
       "beforeBuildCommand": "pnpm build:pack",
       "distDir": "../dist"
     }
   }
   ```

   应用打包后的名称和版本:

   ```json
   {
     "package": {
       "productName": "myApp",
       "version": "0.1.0"
     }
   }
   ```

   以及窗口相关的配置, 具体配置可参考[Tauri 的 API 配置](https://tauri.app/zh-cn/v1/api/config)

   打包命令

   ```bash
   pnpm tauri build
   ```

   打包成功后可以在项目中的 src-tauri/target/release 目录中找到应用的可执行文件

5. 打包时的错误处理

   墙内使用 tauri 打包时可能会遇到一些 github 上的资源下载失败的问题

   - Downloading https://github.com/wixtoolset/wix3/releases/download/wix3112rtm/wix311-binaries.zip

     墙内由于访问 github 非常慢, 下载依赖包大概率失败, 解决办法是直接去下载这个 zip 包, 随后将其中所有文件解压到 C:\Users\用户\AppData\Local\tauri\WixTools 这个目录中, 如果不存在这个目录则手动创建一个

   - Downloading https://github.com/tauri-apps/binary-releases/releases/download/nsis-3/nsis-3.zip

     同理, 这个从 github 上下载的依赖包也大概率失败, 直接下载 zip 包将所有文件解压到 C:\Users\用户\AppData\Local\tauri\NSIS 中

     NSIS3 中还有两个依赖的插件会从 github 自动下载, 如果构建应用时下载失败, tauri 会自动删除本地的 NSIS 包

     所以我们还需要手动去下载 NSIS3 依赖的两个插件, 其下载地址如下:

     - https://github.com/tauri-apps/binary-releases/releases/download/nsis-plugins-v0/NSIS-ApplicationID.zip
     - https://github.com/tauri-apps/nsis-tauri-utils/releases/download/nsis_tauri_utils-v0.1.1/nsis_tauri_utils.dll

     其中, NSIS-ApplicationID.zip 包将其解压后, 复制目录中的 release\ApplicationID.dll 文件到 C:\Users\用户\AppData\Local\tauri\NSIS\Plugins\x86-unicode 目录中, nsis_tauri_utils.dll 下载完成后同样复制到这个目录中, 随后再次执行 tauri build 就能正常打包出可执行文件了
