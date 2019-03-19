---
title: BeautifulSoup库的使用
date: 2019-03-19 16:52:33
categories:
- Python
tags:
- Python
- 爬虫
---
>Beautiful Soup 是一个可以从HTML或XML文件中提取数据的Python库.它能够通过你喜欢的转换器实现惯用的文档导航,查找,修改文档的方式.Beautiful Soup会帮你节省数小时甚至数天的工作时间.
> \- [Beautiful Soup 4.2.0 文档][bs4]
---
# 使用BeautifulSoup解析下面这段代码
~~~ python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""
~~~
通过解析得到一个BeautifulSoup 的对象，通过这个对象可以很容易的获得文档的结构化数据。
~~~ python
from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'lxml')
soup.title
# <title>The Dormouse's story</title>
soup.title.name
# u'title'
soup.title.parent.name
# u'head'
soup.p
# <p class="title"><b>The Dormouse's story</b></p>
soup.p['class']
# u'title'
soup.find_all('a')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
soup.find(id="link3")
# <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>
~~~
# 安装 Beautiful Soup
~~~ bash
pip install beautifulsoup4
~~~
# 解析器
Beautiful Soup支持Python标准库中的HTML解析器，还支持一些第三方解析器。
主流的解析器以及它们的优缺点:

<style>
table th:first-of-type {
	width: 130px;
}
</style>

解析器|使用方法|优势|劣势
-|-|-|-
Python标准库|BeautifulSoup(markup, "html.parser")|Python的内置标准库<br>执行速度适中<br>文档容错能力强|Python 2.7.3和3.2.2前的版本中文档容错能力差
lxml HTML 解析器|BeautifulSoup(markup, "lxml")|速度快<br>文档容错能力强|需要安装C语言库
lxml XML 解析器|BeautifulSoup(markup, ["lxml", "xml"])|速度快<br>唯一支持XML的解析器|需要安装C语言库
html5lib|BeautifulSoup(markup, "html5lib")|最好的容错性<br>以浏览器的方式解析文档<br>生成HTML5格式的文档|速度慢<br>不依赖外部扩展

推荐使用lxml作为解析器，因为xml效率更高。
# 基本用法
Beautiful Soup将复杂HTML文档转换成一个复杂的树形结构，每个节点都是Python对象，所有对象可以归纳为4种: Tag，NavigableString，BeautifulSoup，Comment。

[bs4]: https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/