---
title: 使用github pages和hexo部署个人博客网站
description: 详细介绍如何从零开始搭建个人博客网站，使用 Hexo 创建博客网站，并将其部署到 GitHub Pages，并最终绑定自定义域名。
date: 2022-04-01 16:40:03
categories:
  - Site
tags:
  - Hexo
  - GitHub Pages
  - 博客
  - 网站
---

# 使用 github pages 和 hexo 部署个人博客网站

在本篇博客中，我将详细介绍如何从零开始搭建个人博客网站，使用 Hexo 创建博客网站，并将其部署到 GitHub Pages，并最终绑定自定义域名。让我们一起来跟随下面的步骤进行：

## 步骤一：使用 Hexo 创建博客网站

1. 安装 Node.js 和 Git：
   首先确保您的电脑上安装了 Node.js 和 Git。打开命令行工具，运行以下命令安装 Hexo：

```
  npm install -g hexo-cli
```

2. 初始化项目：
   创建一个新的 Hexo 项目，并进入项目目录：

```
  hexo init my-blog
  cd my-blog
```

3. 添加新文章：
   创建新的文章，使用 Markdown 格式编写文章内容：

```
  hexo new "My First Post"
```

## 步骤二：部署到 GitHub Pages

1. 创建 GitHub 仓库：
   在 GitHub 上创建一个新的仓库，用于存储博客网站的代码。

2. 编辑配置文件：
   打开 \_config.yml 文件，修改 deploy 配置为 GitHub Pages：

```
  deploy:
  type: git
  repo: https://github.com/your-username/your-repo.git
  branch: gh-pages
```

3. 安装 hexo-deployer-git：
   根目录下执行如下命令：

```
  npm install hexo-deployer-git --save
```

之后就可以让 hexo 使用 git 来部署页面到 github 仓库中

4. 部署网站：
   运行以下命令部署网站到 GitHub Pages：

```
  hexo deploy
```

## 步骤三：绑定自定义域名

7. 配置 CNAME 文件：
   在博客项目目录中创建 CNAME 文件，输入您的自定义域名 domain.com。

8. 绑定域名：
   进入 GitHub 仓库设置，将您的自定义域名 domain.com 添加到仓库的 Custom Domain，点击 Save 后，github 会自动开始检查域名是否可用。

9. 配置 DNS 记录：
   在您的 DNS 控制面板中，添加一条 CNAME 记录，将域名指向 GitHub Pages 提供的域名 your-username.github.io.

通过以上步骤，您将成功从零开始搭建个人博客网站，使用 Hexo 创建博客，并将其部署到 GitHub Pages，并绑定自定义域名。祝您的博客网站一切顺利！如有任何问题或需要帮助，请随时告诉我。希朿您顺利搭建个人博客网站！
