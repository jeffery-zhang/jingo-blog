---
title: Tailwindcss 使用细节整理
description: 实际使用中遇到的一些 Tailwindcss 操作的细节和问题的记录
date: 2022-10-08 14:21:52
categories:
  - Web
tags:
  - Tailwindcss
  - Tailwindcss 细节
---

# Tailwindcss 使用细节整理

实际使用中遇到的一些 Tailwindcss 操作的细节和问题的记录

## 类名中使用任意值

### 插入动态值

有时官方的 utilities 不能满足像素级别的设计, 可以使用方括号的形式直接设置像素值或颜色值等:

```html
<div class="w-[125px] h-[64px]">
  <!-- ... -->
</div>

<div class="bg-[#99caff] text-[26px]">
  <!-- ... -->
</div>
```

### Theme 函数调用任意值

类名中可以使用 theme 函数来调用任意已定义的设计:

```html
<div class="grid grid-cols-[fit-content(theme(spacing.32))]">
  <!-- ... -->
</div>
```

### 调用 css 变量

将 css 变量作为任意值时, 不需要封装在 var() 函数中, 直接使用变量名即可:

```html
<div class="bg-[--custom-color]">
  <!-- ... -->
</div>
```

### 任意属性

方括号还可以像写内联样式一样编写任意 css 属性和值

```html
<div class="[mask-type:luminance]">
  <!-- ... -->
</div>

<!-- 可以使用 css 修饰符 -->
<div class="[top:120px] hover:[top-200px]">
  <!-- ... -->
</div>
```

### 处理歧义

Tailwindcss 会根据传入的值自动处理有歧义的值, 如共用 text- 的颜色和大小:

```html
<div class="text-[#99caff] text-[26px]">
  <!-- ... -->
</div>
```

但如果使用 css 变量时可能会无法分别这种歧义, 这时可以在值前添加 css 数据类型来进行区分:

```html
<!-- Will generate a font-size utility -->
<div class="text-[length:var(--my-var)]">...</div>

<!-- Will generate a color utility -->
<div class="text-[color:var(--my-var)]">...</div>
```

## 类名检测注意事项

### 在 js 中操作类名

当在 js 中进行了 tailwindcss 类名的操作时, 需要将 js 文件也作为监测对象配置在 content 中, 如:

```js
btn.addEventListener("click", function () {
  const div = document.getElementById("myDiv");
  div.setAttribute("class", "bg-red-100");
});
```

若 tailwindcss 没有配置 js 内容文件, 则这个类名可能不会生效, 需要在 tailwindcss.config.js 中进行配置:

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    // ...
    "./src/**/*.js",
  ],
  // ...
};
```

### 不要使用动态类名

尽量不要使用动态类名, 如:

```jsx
function (props) {
  return <div className={`text-${props.error ? 'red' : 'green'}`}></div>
}

// 优先使用完整类名
function (props) {
  return <div className={props.error ? 'text-red' : 'text-green'}></div>
}
```

### 安全列表和丢弃类

如果使用动态类名, 可以在安全列表中将用到的类名提前加入该类名:

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    // ...
  ],
  safelist: [
    "bg-red-500",
    "text-3xl",
    "lg:text-4xl",
    // 安全列表还支持正则表达式
    {
      pattern: /bg-(red|green|blue)-(100|200|300)/,
      variants: ["lg", "hover", "focus", "lg:hover"],
    },
  ],
  // ...
};
```

与之相对的还有丢弃类, 表示即便一个类名被 tailwindcss 检测到也不会生成在产物中:

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    // ...
  ],
  // 不能使用正则表达式
  blocklist: ["container"],
  // ...
};
```

## 配置主题

在 theme 配置中可以定制各种 css 属性, 同时覆盖或扩展现有主题:

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  theme: {
    colors: {
      primary: "#0933ff",
    },
    extend: {
      screens: {
        "3xl": "1600px",
      },
    },
  },
};
```

在项目中可以使用这些值:

```html
<div class="text-primary md:text-md 3xl:text-lg">Hola</div>
```

详细可配置项参阅[主题](https://tailwind.nodejs.cn/docs/theme#-6)

### 引用主题其他值或默认主题

```js
/** @type {import('tailwindcss').Config} */
const defaultTheme = require("tailwindcss/defaultTheme");

module.exports = {
  theme: {
    fill: ({ theme }) => ({
      gray: theme("colors.gray"), // 使用链式调用获取实际值
    }),
    extend: {
      fontFamily: {
        sans: ["Lato", ...defaultTheme.fontFamily.sans], // 获取默认主题的值
      },
    },
  },
};
```

## 插件

可用于注册新样式, 包括工具类, 基础样式, 组件等

基本语法:

```js
const plugin = require("tailwindcss/plugin");

module.exports = {
  theme: {
    borderRadius: {
      sm: "4px",
    },
  },
  plugins: [
    plugin(function ({
      addUtilities,
      addComponents,
      addBase,
      addVariant,
      theme,
    }) {
      // 添加静态工具类
      addUtilities({
        ".content-auto": {
          "content-visibility": "auto",
        },
      });

      // 添加组件
      addComponents({
        ".btn": {
          background: "blue",
          color: "#fff",
          // theme 函数可以引用其他配置值
          "border-radius": theme("borderRadius.sm"),
          // 可直接使用伪类
          "&:hover": {
            background: "gray",
          },
        },
      });

      // 添加基础样式
      addBase({
        h1: {
          fontSize: "32px",
        },
      });

      // 添加变体
      addVariant("optional", "&:optional");
      addVariant("hocus", ["&:hover", "&:focus"]);
      addVariant("inverted-colors", "@media (inverted-colors: inverted)");
    }),
  ],
};
```
