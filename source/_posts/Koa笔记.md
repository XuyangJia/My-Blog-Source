---
title: Koa笔记
date: 2019-03-21 14:20:02
categories:
- Node
tags:
- Koa
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
~~~ js
const koa = require('koa'),
    router = require('koa-router')();

const app = new koa();
router.get('/about', async ctx => {
    ctx.response.body = '<h1>About Page</h1>';
});
router.get('/', async ctx => {
    ctx.response.body = '<h1>Index Page</h1>';
});
app.use(router.routes());
app.listen(3000, () => { console.log('app started at 3000...'); });
~~~
访问 http://127.0.0.1:3000 , => Index Page
访问 http://127.0.0.1:3000/about , => About Page
# 处理post请求
router.get('/path', async fn)用于处理get请求。要处理post请求，可以用router.post('/path', async fn)。
使用koa-bodyparser解析request的body。
~~~ js
const koa = require('koa'),
    bodyparser = require('koa-bodyparser'),
    router = require('koa-router')();

const app = new koa();
app.use(bodyparser());
router.get('/', async (ctx, next) => {
    ctx.response.body = `<h1>Index Page</h1>
        <form action="/signin" method="post">
            <p>username:<input name="name" value="koa"></p>
            <p>password:<input name="pass" type="password"></p>
            <p><input type="submit" value="Submit"></p>
        </form>`;
});
router.post('/signin', async (ctx, next) => {
    const name = ctx.request.body.name || '',
        pass = ctx.request.body.pass || '';
    console.info(`signin with name: ${name}, password: ${pass}`);
    if (name === 'koa' && pass === '123456')
        ctx.response.body = `<h1>Welcome ${name}</h1>`;
    else
        ctx.response.body = `<h1>Login field!</h1>
        <p><a href="/">Please try again!</a> </p>`;
});
app.use(router.routes());
app.listen(3000, () => { console.log('app started at 3000...'); });
~~~
# 模板引擎
网页框架base.html
~~~ html
<body>
    <h1>{{name}}</h1>
</body>
~~~
在app.js中编写代码
~~~ js
const nunjucks = require('nunjucks');

 function createEnv(path, opts) {
     const autoescape = opts.autoescape || true,
         noCache = opts.noCache || false,
         watch = opts.watch || false,
         throwOnUndefined = opts.throwOnUndefined || false,
         env = new nunjucks.Environment(
             new nunjucks.FileSystemLoader('views', {noCache, watch}),
             {autoescape, throwOnUndefined}
         );
     if (opts.filters) {
         for (const f in opts.filters) {
             env.addFilter(f, opts.filters[f]);
         }
     }
     return env;
 }

 const env = createEnv('views', {
     watch: true,
     filters: {hex: n => {return '0x' + n.toString(16);}}
 });
 var str = env.render('base.html', {name: 'Jack'});
console.log(str);
~~~
$ node app.js 输出如下：
~~~ html
<h1>Jack</h1>
~~~