---
title: 2分钟搭建 github 文件下载加速服务
description: 利用 Cloudflare Worker 提供的功能和优势，通过简单的步骤实现加速 GitHub 文件下载。
date: 2024-06-13 16:31:03
categories:
  - Site
tags:
  - Github
  - Proxy
  - Cloudflare
  - Worker
---

# 2 分钟搭建 github 文件下载加速服务

今天发现一个加速 GitHub 下载的方法，体验非常不错！简单操作，仅花了 2 分钟就搞定了，速度也确实快了不少。

## 创建 Cloudflare Worker

使用 Cloudflare Worker 创建无服务器反代程序, 可以利用 Cloudflare 的国内 CDN 加速完成代理, 虽然免费版有一天 10 万次请求限制, 但自用也绰绰有余了

首先进入 Workers 和 Pages 页面, 点击创建, 随后选择从模板开始中的 Hello World 模板

![deploy-github-download-proxy](/images/deploy-github-download-proxy-01.png)

进去后取个名字直接点击部署

## 编辑代码

点击编辑代码进入编辑器, 复制以下内容:

```js
"use strict";

/**
 * static files (404.html, sw.js, conf.js)
 */
const ASSET_URL = "https://hunshcn.github.io/gh-proxy/";
// 前缀，如果自定义路由为example.com/gh/*，将PREFIX改为 '/gh/'，注意，少一个杠都会错！
const PREFIX = "/";
// git使用cnpmjs镜像、分支文件使用jsDelivr镜像的开关，0为关闭，默认开启
const Config = {
  jsdelivr: 1,
  cnpmjs: 1,
};

/** @type {RequestInit} */
const PREFLIGHT_INIT = {
  status: 204,
  headers: new Headers({
    "access-control-allow-origin": "*",
    "access-control-allow-methods":
      "GET,POST,PUT,PATCH,TRACE,DELETE,HEAD,OPTIONS",
    "access-control-max-age": "1728000",
  }),
};

const exp1 =
  /^(?:https?:\/\/)?github\.com\/.+?\/.+?\/(?:releases|archive)\/.*$/i;
const exp2 = /^(?:https?:\/\/)?github\.com\/.+?\/.+?\/(?:blob|raw)\/.*$/i;
const exp3 = /^(?:https?:\/\/)?github\.com\/.+?\/.+?\/(?:info|git-).*$/i;
const exp4 =
  /^(?:https?:\/\/)?raw\.(?:githubusercontent|github)\.com\/.+?\/.+?\/.+?\/.+$/i;
const exp5 =
  /^(?:https?:\/\/)?gist\.(?:githubusercontent|github)\.com\/.+?\/.+?\/.+$/i;
const exp6 = /^(?:https?:\/\/)?github\.com\/.+?\/.+?\/tags.*$/i;

/**
 * @param {any} body
 * @param {number} status
 * @param {Object<string, string>} headers
 */
function makeRes(body, status = 200, headers = {}) {
  headers["access-control-allow-origin"] = "*";
  return new Response(body, { status, headers });
}

/**
 * @param {string} urlStr
 */
function newUrl(urlStr) {
  try {
    return new URL(urlStr);
  } catch (err) {
    return null;
  }
}

addEventListener("fetch", (e) => {
  const ret = fetchHandler(e).catch((err) =>
    makeRes("cfworker error:\n" + err.stack, 502)
  );
  e.respondWith(ret);
});

function checkUrl(u) {
  for (let i of [exp1, exp2, exp3, exp4, exp5, exp6]) {
    if (u.search(i) === 0) {
      return true;
    }
  }
  return false;
}

/**
 * @param {FetchEvent} e
 */
async function fetchHandler(e) {
  const req = e.request;
  const urlStr = req.url;
  const urlObj = new URL(urlStr);
  let path = urlObj.searchParams.get("q");
  if (path) {
    return Response.redirect("https://" + urlObj.host + PREFIX + path, 301);
  }
  // cfworker 会把路径中的 `//` 合并成 `/`
  path = urlObj.href
    .substr(urlObj.origin.length + PREFIX.length)
    .replace(/^https?:\/+/, "https://");
  if (
    path.search(exp1) === 0 ||
    path.search(exp5) === 0 ||
    path.search(exp6) === 0 ||
    (!Config.cnpmjs && (path.search(exp3) === 0 || path.search(exp4) === 0))
  ) {
    return httpHandler(req, path);
  } else if (path.search(exp2) === 0) {
    if (Config.jsdelivr) {
      const newUrl = path
        .replace("/blob/", "@")
        .replace(/^(?:https?:\/\/)?github\.com/, "https://cdn.jsdelivr.net/gh");
      return Response.redirect(newUrl, 302);
    } else {
      path = path.replace("/blob/", "/raw/");
      return httpHandler(req, path);
    }
  } else if (path.search(exp3) === 0) {
    const newUrl = path.replace(
      /^(?:https?:\/\/)?github\.com/,
      "https://github.com.cnpmjs.org"
    );
    return Response.redirect(newUrl, 302);
  } else if (path.search(exp4) === 0) {
    const newUrl = path
      .replace(/(?<=com\/.+?\/.+?)\/(.+?\/)/, "@$1")
      .replace(
        /^(?:https?:\/\/)?raw\.(?:githubusercontent|github)\.com/,
        "https://cdn.jsdelivr.net/gh"
      );
    return Response.redirect(newUrl, 302);
  } else {
    return fetch(ASSET_URL + path);
  }
}

/**
 * @param {Request} req
 * @param {string} pathname
 */
function httpHandler(req, pathname) {
  const reqHdrRaw = req.headers;

  // preflight
  if (
    req.method === "OPTIONS" &&
    reqHdrRaw.has("access-control-request-headers")
  ) {
    return new Response(null, PREFLIGHT_INIT);
  }

  const reqHdrNew = new Headers(reqHdrRaw);

  let urlStr = pathname;
  if (urlStr.startsWith("github")) {
    urlStr = "https://" + urlStr;
  }
  const urlObj = newUrl(urlStr);

  /** @type {RequestInit} */
  const reqInit = {
    method: req.method,
    headers: reqHdrNew,
    redirect: "manual",
    body: req.body,
  };
  return proxy(urlObj, reqInit);
}

/**
 *
 * @param {URL} urlObj
 * @param {RequestInit} reqInit
 */
async function proxy(urlObj, reqInit) {
  const res = await fetch(urlObj.href, reqInit);
  const resHdrOld = res.headers;
  const resHdrNew = new Headers(resHdrOld);

  const status = res.status;

  if (resHdrNew.has("location")) {
    let _location = resHdrNew.get("location");
    if (checkUrl(_location)) resHdrNew.set("location", PREFIX + _location);
    else {
      reqInit.redirect = "follow";
      return proxy(newUrl(_location), reqInit);
    }
  }
  resHdrNew.set("access-control-expose-headers", "*");
  resHdrNew.set("access-control-allow-origin", "*");

  resHdrNew.delete("content-security-policy");
  resHdrNew.delete("content-security-policy-report-only");
  resHdrNew.delete("clear-site-data");

  return new Response(res.body, {
    status,
    headers: resHdrNew,
  });
}
```

修改后点击保存并部署

然后就能通过自动生成的 workers.dev 域名访问该页面了

![deploy-github-download-proxy](/images/deploy-github-download-proxy-02.png)

在输入框中输入 github 上的文件下载链接即可直接加速下载该文件了

## 使用自定义域名

要使用自定义域名访问该页面, 需要用 Cloudflare 托管你的域名

在 Workers 和 Pages 页面中找到你部署的 Worker, 点击进入后进入设置页面, 在域和路由选项处点击添加, 然后输入你想要的域名并部署即可, 之后就可以通过该自定义域名访问代理页面了

## 原项目

[https://github.com/hunshcn/gh-proxy](https://github.com/hunshcn/gh-proxy)
