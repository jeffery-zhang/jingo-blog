---
title: 使用 next-on-pages 部署 nextjs 项目时遇到的一些问题
description: 使用 next-on-pages 将 nextjs 项目部署到 cloudflare 上时遇到的一些问题
date: 2024-01-23 17:19:01
categories:
  - Site
tags:
  - next-on-pages
  - nextjs
  - cloudflare
  - wrangler
---

# 使用 next-on-pages 部署 nextjs 项目时遇到的一些问题

使用 next-on-pages 将 nextjs 项目部署到 cloudflare 上时遇到的一些问题

## 创建一个 next-on-pages 项目

对于已有的 nextjs 项目, 可以在项目中添加 @cloudflare/next-on-pages 依赖:

```sh
npm i -D @cloudflare/next-on-pages
```

并且在构建时需要修改 scripts:

```json
{
  "scripts": {
    "build": "npx @cloudflare/next-on-pages@1",
    // or
    "build": "npx next-on-pages"
  }
}
```

部署到 cloudflare pages 上可以使用 wrangler 来进行部署

```sh
npm i -D wrangler
```

部署命令:

```json
{
  "scripts": {
    "deploy": "npm run build && wrangler pages deploy"
  }
}
```

对于新建的项目, 则推荐直接使用 wrangler 来创建 nextjs 项目

```sh
npm create cloudflare@latest my-pages
# 根据后续提示选择nextjs框架
```

脚手架会创建一个基于 create-next-app 默认模板的 nextjs 项目, 随后开发流程和 nextjs 一样

## Windows 系统构建报错

在 windows 上用 next-on-pages 构建 nextjs 项目时有时会出错, 通常抛出的是 nodejs 的 child_process 执行命令时的错误, 没有明确的错误说明

但在使用 next-on-pages 也会提示在 windows 上不稳定:

```sh
⚡️ Warning: It seems like you're on a Windows system, the Vercel CLI (run by @cloudflare/next-on-pages
⚡️ to build your application) seems not to work reliably on Windows so if you experience issues during
⚡️ the build process please try switching to a different operating system or running
⚡️ @cloudflare/next-on-pages under the Windows Subsystem for Linux
```

所以可以将项目迁移到 linux 系统或 WSL 中来构建, 同时也需要将 npm 等基本环境安装到 WSL 中

如果不想安装 WSL, 也可以将项目上传到 github 然后连接到 cloudflare pages 空间, 利用云服务器来进行构建

## 如何在 cloudflare 上连接 github 仓库

1. 进入 cloudflare dashboard, 打开 Workers & Pages 的 Overview 页面, 点击 Create

![issues-about-next-on-pages-01](/images/issues-about-next-on-pages-01.png)

2. 选择 Pages 并点击 Connect to Git

![issues-about-next-on-pages-02](/images/issues-about-next-on-pages-02.png)

cloudflare 目前支持 github 和 gitlab 自动部署, 如果之前已经在 cloudflare 连接过 github 账号, 此时就已经能看到自己的 github 账号下的所有公开仓库了, 选择想要部署的项目然后点击 Begin Setup

3. 进入项目构建配置页面, 选择项目分支, 预设的开发框架, 输入构建命令和输出静态文件的目录(相对于根目录), 还可以选择添加环境变量或者改变根目录所在位置

当选择 nextjs 作为开发框架时, cloudflare 通常会自动配置好命令和输出目录

![issues-about-next-on-pages-03](/images/issues-about-next-on-pages-03.png)

都配置好以后点击 Save and Deploy 就会开始自动构建和部署

## 构建 nextjs 之前需要注意的细节

需要注意的是, 在 cloudflare 上部署 nextjs 项目时, 项目中使用的 api 路由需要额外配置运行时为 edge

项目中 /api 下的所有 route.ts 中均需要加上如下代码:

```ts
export const runtime = "edge";
```

同时 nextjs 构建时也会强制执行 ts 类型检查, 需要注意项目中的 route.ts, layout.ts, page.ts 等文件中不要导出无关的代码
