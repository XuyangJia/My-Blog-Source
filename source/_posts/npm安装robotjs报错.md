---
title: npm安装robotjs报错
date: 2019-05-17 16:11:39
categories:
- npm
tags:
- ERR
---

# robotjs不支持Python 3.x 需要安装Python 2.x

使用npm install或者yarn 安装依赖时，可能会出现如下类似的错误：
~~~
MSBUILD : error MSB4132: The tools version "2.0" is unrecognized. Available tools versions are "4.0".
~~~
或者：
~~~
MSBUILD : error MSB4132: The tools version "2.0" is unrecognized. Available tools versions are "12.0", "4.0".
~~~
该类错误发生在node-gyp在构建时未能找到所需版本的构建工具，解决方法如下：
~~~bash
npm install --global --production windows-build-tools
npm config set msvs_version 2015 --global
~~~

参考链接 https://www.cnblogs.com/xzysaber/p/8649228.html