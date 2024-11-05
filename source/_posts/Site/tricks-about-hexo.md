---
title: 一些关于hexo网站的小技巧
description: 一些在使用hexo建站时有用的小技巧
date: 2022-04-01 17:56:49
categories:
  - Site
tags:
  - Hexo
  - 博客
  - 技巧
---

# 一些关于 hexo 网站的小技巧

一些在使用 hexo 建站时有用的小技巧

## 自动创建 post

项目根目录执行以下命令:

```sh
hexo new post my-blog
```

执行后就会在 /source/\_post 下新增一个 my-blog.md 文件, 并且会自动生成简单的 frontmatter 信息

若想在\_post 的子目录中创建文章, 则需要带上 -p 参数

```sh
hexo new post -p blogs/my-blog
```

这样则会在 /source/\_post/blogs 目录下创建文件

## 显示菜单项

打开根目录下的\_config.yml, 或你使用的主题所在的目录下的\_config.yml

找到 menu: 配置项, 如下:

```yml
# Usage: `Key: /link/ || icon`
# Key is the name of menu item. If the translation for this item is available, the translated text will be loaded, otherwise the Key name will be used. Key is case-senstive.
# Value before `||` delimiter is the target link, value after `||` delimiter is the name of Font Awesome icon.
# When running the site in a subdirectory (e.g. yoursite.com/blog), remove the leading slash from link value (/archives -> archives).
# External url should start with http:// or https://
menu:
  home: / || fa fa-home
  #about: /about/ || fa fa-user
  #tags: /tags/ || fa fa-tags
  #categories: /categories/ || fa fa-th
  archives: /archives/ || fa fa-archive
  #schedule: /schedule/ || fa fa-calendar
  #sitemap: /sitemap.xml || fa fa-sitemap
  #commonweal: /404/ || fa fa-heartbeat

# Enable / Disable menu icons / item badges.
menu_settings:
  icons: true
  badges: false
```

取消对应菜单项的注释, 重新运行 hexo, 就可以看到相应菜单显示出来

菜单项中每一个配置的规则为:

```
Usage: `Key: /link/ || icon`
```

若配置 tags 菜单, 该菜单会显示配置的 icon 的图标, 而点击该菜单时会去寻找 public/tags 下的 index.html 页面

hexo 默认会生成 home 和 archives 两个页面, 若需要显示其他页面, 则需要创建对应的 page, 如生成 tags 页面:

```sh
hexo new page tags # 生成tags page
```

此时在项目的 source/tags 下会生成一个 index.md 文件, 打开该文件修改 frontmatter 如下:

```md
---
title: tags
date: 2020-01-01 00:00:00
type: "tags"
comments: false
---
```

再次运行 hexo, 点击 tags 菜单则会正确打开 tags 页面

## 显示 github 图标

在\_config.yml 中找到 github_banner 配置项, 设置 enable 为 true, 并添加自己的 github 链接:

```yml
# `Follow me on GitHub` banner in the top-right corner.
github_banner:
  enable: true
  permalink: https://github.com/your-name
  title: Follow me on GitHub
```

## 隐藏内容点击展开

使用 html5 提供的 details 标签, 在 summary 标签中写入隐藏时显示的文字, 在其后的 p 标签中写入详细内容, 如:

```html
<details>
  <summary>标题</summary>
  <p>详情</p>
</details>
```

展示效果如下:

<details>
  <summary>标题</summary>
  <p>详情</p>
</details>

## 自动添加 github pages 自定义域名

由于 github 的限制, 在使用自定义域名时, 每次提交代码后自定义域名都会失效, 需要重新配置

要解决这个问题, 只需要在 source 目录下添加一个 CNAME 文件, 并在其中写入你的域名即可, 如:

```
yourdomain.net
```

随后使用 hexo g 提交到 GitHub 上就会自动在 pages 中设置该自定义域名

## 使用搜索功能

安装相关插件:

```sh
npm i -S hexo-generator-searchdb
```

在 \_config.yml 配置文件中添加如下配置:

```yml
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

使用 Next 主题时自带本地搜索设置, 编辑主题中的 \_config.yml 文件中的 local_search:

```yml
# Local_search

local_search:
  eanble: true
```

## 侧边栏社交链接

侧边栏社交链接的配置有两部分, 一个是链接, 一个是连接图标, 在使用主题的 \_config.yml 中配置

链接在 social 配置项下面, 格式为 `显示文本: 链接地址 || 显示的 Font Awesome 图标`

```yml
social:
  GitHub: https://github.com/yourname || fab fa-github
  #E-Mail: mailto:yourname@gmail.com || fa fa-envelope
  #Weibo: https://weibo.com/yourname || fab fa-weibo
  #Google: https://plus.google.com/yourname || fab fa-google
  #Twitter: https://twitter.com/yourname || fab fa-twitter
  #FB Page: https://www.facebook.com/yourname || fab fa-facebook
  #StackOverflow: https://stackoverflow.com/yourname || fab fa-stack-overflow
  #YouTube: https://youtube.com/yourname || fab fa-youtube
  #Instagram: https://instagram.com/yourname || fab fa-instagram
  #Skype: skype:yourname?call|chat || fab fa-skype
```

图标的设置在 social_icons 配置项下面, 可以开启是否显示图标

```yml
social_icons:
  enable: true
  icons_only: false
  transition: false
```
