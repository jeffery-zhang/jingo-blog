---
title: 用于打印的CSS
description: 介绍如何使用CSS来控制打印的外观
date: 2022-04-16 12:33:12
categories:
  - Web
tags:
  - HTML
  - CSS
  - 打印
  - 媒体查询
---

# 用于打印的 CSS

介绍如何使用 CSS 来控制打印的外观

## @page

CSS 中有一个名为 @page 的规则，它通知浏览器有关您网站打印偏好的信息。通常，我使用

```css
@page {
  size: Letter portrait;
  margin: 0;
}
```

在稍后关于边距的部分，我将解释为什么我选择了 margin: 0。您应根据与公制的关系选择使用 Letter 或 A4。

设置 @page 的大小和边距并不同于设置 html 或 body 元素的宽度、高度和边距。@page 位于 DOM 之外 — 它包含了 DOM。在 Web 上，html 元素受屏幕边缘限制，但在打印时受 @page 限制。

@page 控制的设置更多或更少对应于在按 Ctrl+P 时浏览器的打印对话框中得到的设置。

这是我用来进行一些实验的示例文件。

```html
<!DOCTYPE html>
<html>
  <style>
    @page {
      /* see below for each experiment */
    }
    html {
      width: 100%;
      height: 100%;
      background-color: lightblue;

      /* grid by shunryu111 https://stackoverflow.com/a/32861765/5430534 */
      background-size: 0.25in 0.25in;
      background-image: linear-gradient(to right, gray 1px, transparent 1px),
        linear-gradient(to bottom, gray 1px, transparent 1px);
    }
  </style>
  <body>
    <h1>Sample text</h1>
    <p>sample text</p>
  </body>
</html>
```

在浏览器中看起来是这样的:

![css-for-printing](/images/css-for-printing-01.png)

然后这里是写入不同 @page 值的外观:

```css
@page {
  size: Letter portrait;
  margin: 1in;
}
```

![css-for-printing](/images/css-for-printing-02.png)

```css
@page {
  size: Letter landscape;
  margin: 1in;
}
```

![css-for-printing](/images/css-for-printing-03.png)

```css
@page {
  size: Letter landscape;
  margin: 0;
}
```

![css-for-printing](/images/css-for-printing-04.png)

设置 @page 的尺寸实际上不会将该尺寸的纸张放入打印机的进纸盘中。您需要自己完成这部分。

注意当我将尺寸设置为 A5 时，我的打印机仍保持在 Letter 尺寸，而 A5 尺寸完全适合在 Letter 尺寸内，这给人以边距的外观，尽管这并非来自于边距设置。

```css
@page {
  size: A5 portrait;
  margin: 0;
}
```

![css-for-printing](/images/css-for-printing-05.png)

但如果我告诉打印机我已加载了真实的 A5 纸张，那么打印效果就会如预期。

![css-for-printing](/images/css-for-printing-06.png)

根据我的实验，我得出的结论是，Chrome 只有在将边距设置为默认时才遵循 @page 规则。一旦您在打印对话框中更改边距，您的输出效果就会成为您的物理纸张尺寸与所选边距的结合产品。

即使您选择一个完全适合您的物理纸张的 @page 尺寸，边距仍然很重要。在这里，我创建了一个没有边距的 5x5 正方形，以及一个带边距的 5x5 正方形。html 元素的尺寸由 @page 尺寸和边距的组合所限制。

```css
@page {
  size: 5in 5in;
  margin: 0;
}
```

![css-for-printing](/images/css-for-printing-07.png)

我进行所有这些测试并不是因为我期望在 A5 或 5x5 纸张上打印，而是因为我花了一些时间来弄清楚 @page 到底是什么。现在我非常有信心始终使用 Letter 并将边距设为 0

## @media print

有一个名为 print 的媒体查询，您可以在其中编写仅在打印时应用的样式。我的生成器页面通常包含页眉、一些选项和一些帮助用户的文本，显然这些内容不应在打印时显示，因此在这里将这些元素添加 display:none

```css
/* Normal styles that appear while you are preparing the document */
header {
  display: block;
}

@media print {
  /* Disappear when you are printing the document */
  header {
    display: none;
  }
}
```

![css-for-printing](/images/css-for-printing-08.png)

## Width, height, margin, padding

您需要了解一些关于盒模型的知识，以便在不费太多功夫的情况下获得您想要的边距。

![css-for-printing](/images/css-for-printing-09.png)

我总是将 @page 的 margin 设置为 0 的原因是，我宁愿在 DOM 元素上处理边距。当我尝试使用 @page 设置 margin 为 0.5in 时，我经常会意外地得到双倍边距，导致内容比我预期的要挤压，我的单页设计溢出到第二页。

如果我想使用 @page 的 margin，那么实际的页面内容需要紧贴在 DOM 的边缘上，这对我来说更难思考，也更难在打印前进行预览。对我来说，在头脑中记住 html 占据整张物理纸张，而边距在 DOM 内而不是在 DOM 之外，会更容易一些。

```css
@page {
  size: Letter portrait;
  margin: 0;
}
html,
body {
  width: 8.5in;
  height: 11in;
}
```

在多页面打印生成器中，您需要一个表示每个页面的单独 DOM 元素。由于您不能有多个 html 或 body，您需要另一个元素。我喜欢使用 article。即使对于单页面生成器，您也可以始终使用 article。

由于每个 article 代表一页，我不希望在 html 或 body 上有任何边距或填充。我们将逻辑推进一步 — 对我来说，让 article 占据整张物理页面并将边距放在其中会更容易一些。

```css
@page {
  size: Letter portrait;
  margin: 0;
}
html,
body {
  margin: 0;
}

article {
  width: 8.5in;
  height: 11in;
}
```

当我谈到在我的 article 中添加边距时，我不是使用 margin 属性，而是使用 padding。这是因为 margin 在盒模型中是在元素外部和周围的。如果您使用 0.5in 的边距，您将不得不将 article 设置为 7.5×10，以便 article 加上 2× 边距等于 8.5×11。如果您想要调整边距，您将不得不调整其他尺寸。

相反，padding 放在元素内部，因此我可以将 article 定义为 8.5×11，带有 0.5in 的内边距，article 内的所有元素将保持在页面上。

当您设置 box-sizing: border-box 时，关于元素尺寸的许多直觉更容易理解。这样可以锁定 article 的外部尺寸，同时调整内部 padding。这是我的代码片段：

```css
html {
  box-sizing: border-box;
}
*,
*:before,
*:after {
  box-sizing: inherit;
}
```

将这些代码合并起来:

```css
@page {
  size: Letter portrait;
  margin: 0;
}

html {
  box-sizing: border-box;
}
*,
*:before,
*:after {
  box-sizing: inherit;
}

html,
body {
  margin: 0;
}

article {
  width: 8.5in;
  height: 11in;
  padding: 0.5in;
}
```

![css-for-printing](/images/css-for-printing-10.png)

## 元素定位

一旦您设置好了 article 和边距，article 内部的空间就完全由您自行决定如何使用。设计您的文档时，可以根据项目选择使用任何您认为合适的 HTML/CSS。有时这意味着使用 flex 或 grid 布局元素，因为您在输出方面有一些灵活性。有时这意味着创建特定尺寸的方块以适合某个品牌的贴纸纸张。有时这意味着绝对定位所有内容到毫米，因为用户需要将特殊的预标记纸张放入打印机，在其上打印您的数据，而您无法控制那种特殊纸张。

我在这里不是为了教授如何撰写一般的 HTML 教程，所以您需要能够做到这一点。我只能说的是，请注意您正在处理的是一张纸的有限空间，而不是像浏览器窗口那样可以滚动和缩放到任意长度或比例。如果您的文档将包含任意数量的项目，请准备通过创建更多的 article 来进行分页。

## 多页文档和重复元素

```html
<table>
  <thead>
    <tr>
      <th>Sample text</th>
      <th>Sample text</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <td>2</td>
      <td>4</td>
    </tr>
    ...
  </tbody>
</table>
```

如果只是打印一个普通的没有复杂样式的 table 很简单，但在许多实际情况下并不是那么简单。我重新创建的文档通常每页顶部都有抬头，底部有页脚，以及其他需要在每页明确重复的自定义元素。如果只是在多页之间打印一个很长的表格，您在中间页上就没有太多机会将其他元素放在它的上方、下方或周围。

因此，我使用 JavaScript 生成页面，将表格分割为多个较小的部分。总体的方法如下：

- 将 article 元素视为可丢弃的，随时准备从内存中的对象重新生成它们。所有用户输入和配置应该在单独的页眉/选项框中进行，而不是在 article 中。
- 编写一个名为 new_page 的函数，用于创建具有必要重复的页眉/页脚等的新 article 元素。
- 编写一个名为 renderpages 的函数，从基本数据创建 article，每次填满上一个 article 时调用 newpage。我通常使用 offsetTop 查看内容何时到达页面的较远处，尽管您肯定可以使用更智能的技术来实现每页的完美适配。
- 每当基本数据更改时调用 render_pages。

```js
function delete_articles() {
  for (const article of Array.from(document.getElementsByTagName("article"))) {
    document.body.removeChild(article);
  }
}

function new_page() {
  const article = document.createElement("article");
  article.innerHTML = `
    <header>...</header>
    <table>...</table>
    <footer>...</footer>
    `;
  document.body.append(article);
  return article;
}

function render_pages() {
  delete_articles();

  let page = new_page();
  let tbody = page.query("table tbody");
  for (const line_item of line_items) {
    // I usually pick this threshold by experimentation but you can probably
    // do something more rigorously correct.
    if (tbody.offsetTop + tbody.offsetParent.offsetTop > 900) {
      page = new_page();
      tbody = page.query("table tbody");
    }
    const tr = document.createElement("tr");
    tbody.append(tr);
    // ...
  }
}
```

通常在页面上包含一个"第 X 页，共 Y 页"的计数器是很有意义的。由于直到所有页面生成后才知道页数，所以我无法在循环中完成这个操作。我在最后调用类似下面的函数：

```js
function renumber_pages() {
  let pagenumber = 1;
  const pages = document.getElementsByTagName("article");
  for (const page of pages) {
    page.querySelector(".pagenumber").innerText = pagenumber;
    page.querySelector(".totalpages").innerText = pages.length;
    pagenumber += 1;
  }
}
```

## Portrait / Landscape 模式

我已经展示了 @page 规则有助于告知浏览器的默认打印设置，但用户可以在需要时进行覆盖。如果您将 @page 设置为纵向模式，而用户将其覆盖为横向模式，您的布局和分页可能看起来不正确，特别是如果您正在硬编码任何页面阈值。

您可以通过为纵向和横向分别创建 style 元素，并使用 JavaScript 在它们之间进行切换来满足用户需求。可能有更好的方法来实现这一点，但类似 @page 这样的 at-rules 的行为与常规 CSS 属性不同，所以我不太确定。您还应该保存一些变量，以帮助 render_pages 函数正确执行操作。

您也可以停止硬编码阈值，但那样我就要听从我自己的建议。

```html
<select onchange="return page_orientation_onchange(event);">
  <option selected>Portrait</option>
  <option>Landscape</option>
</select>
```

```html
<style id="style_portrait" media="all">
  @page {
    size: Letter portrait;
    margin: 0;
  }
  article {
    width: 8.5in;
    height: 11in;
  }
</style>

<style id="style_landscape" media="not all">
  @page {
    size: Letter landscape;
    margin: 0;
  }
  article {
    width: 11in;
    height: 8.5in;
  }
</style>
```

```js
let print_orientation = "portrait";

function page_orientation_onchange(event) {
  print_orientation = event.target.value.toLocaleLowerCase();
  if (print_orientation == "portrait") {
    document.getElementById("style_portrait").setAttribute("media", "all");
    document.getElementById("style_landscape").setAttribute("media", "not all");
  }
  if (print_orientation == "landscape") {
    document.getElementById("style_landscape").setAttribute("media", "all");
    document.getElementById("style_portrait").setAttribute("media", "not all");
  }
  render_printpages();
}

function render_printpages() {
  if (print_orientation == "portrait") {
    // ...
  } else {
    // ...
  }
}
```

## 数据源

有几种方法可以将数据放在页面上。有时，我将所有数据打包到 URL 参数中，因此 JavaScript 只需执行 const urlparams = new URLSearchParams(window.location.search); 然后一堆 urlparams.get("title")。这样做有一些优势：

- 页面加载非常快。
- 通过更改 URL 来调试和实验非常容易。
- 生成器可以脱机工作。

这也有一些缺点：

- URL 变得非常长而混乱，人们不能舒适地通过电子邮件相互发送它们。请参见本文章顶部的示例链接。
- 如果 URL 确实通过电子邮件发送，即使以后数据库中的源记录发生变化，这些数据也会“被锁定”。
- 浏览器对 URL 长度有限制。这些限制非常高，但并非无限，而且可能因客户端而异。

有时候，我相反使用 JavaScript 通过 API 获取我们的数据库记录，因此 URL 参数只包含记录的主键和可能的模式设置。

这有一些优点：

- URL 长度更短。
- 数据始终是最新的。

以及一些缺点：

- 用户必须等待一会儿，直到数据被获取。
- 您需要编写更多的代码。

有时我会在 article 上设置 contenteditable，以便用户在打印前进行小的更改。我也喜欢使用用户可以在打印前点击的实际复选框输入框。这些功能增加了一些便利性，但在大多数情况下，更明智的做法是让用户首先更改数据库中的源记录。此外，它们限制了您将 article 元素视为可丢弃的能力。

## 完整代码

```html
<!DOCTYPE html>
<html>
  <style>
    @page {
      size: Letter portrait;
      margin: 0;
    }
    html {
      box-sizing: border-box;
    }
    *,
    *:before,
    *:after {
      box-sizing: inherit;
    }

    html,
    body {
      margin: 0;
      background-color: lightblue;
    }

    header {
      background-color: white;
      max-width: 8.5in;
      margin: 8px auto;
      padding: 8px;
    }

    article {
      background-color: white;
      padding: 0.5in;
      width: 8.5in;
      height: 11in;

      /* For centering the page on the screen during preparation */
      margin: 8px auto;
    }

    @media print {
      html,
      body {
        background-color: white !important;
      }
      body > header {
        display: none;
      }
      article {
        margin: 0 !important;
      }
    }
  </style>

  <body>
    <header>
      <p>Some help text to explain the purpose of this generator.</p>
      <p><button onclick="return window.print();">Print</button></p>
    </header>

    <article>
      <h1>Sample page 1</h1>
      <p>sample text</p>
    </article>

    <article>
      <h1>Sample page 2</h1>
      <p>sample text</p>
    </article>
  </body>
</html>
```
