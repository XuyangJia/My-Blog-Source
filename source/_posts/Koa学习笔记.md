---
title: Koa学习笔记
date: 2019-03-21 14:20:02
tags:
---
Koa 就是一种简单好用的 Web 框架。它的特点是优雅、简洁、表达力强、自由度高。本身代码只有1000多行，所有功能都通过插件实现，很符合 Unix 哲学。
# 基本用法
## 架设 HTTP 服务
hello world 应用
~~~ js
// server.js
const Koa = require('koa');
const app = new Koa();
app.use(async ctx => {
    ctx.body = 'Hello World';
});
app.listen(3000);
~~~
运行这个脚本。
~~~ bash
$ node server.js
~~~
访问 http://127.0.0.1:3000 ，现在就可以看到"Hello World"了。
## 网页模板
~~~ js
const fs = require('fs');
const main = ctx => {
  ctx.response.type = 'html';
  ctx.response.body = fs.createReadStream('./template.html');
};
~~~
访问 http://127.0.0.1:3000 ，看到的就是模板文件的内容了。
# 路由