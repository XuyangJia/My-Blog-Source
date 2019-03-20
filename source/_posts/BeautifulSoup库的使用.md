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
> \- [Beautiful Soup 4 文档][bs4]
本文为学习BeautifulSoup时所做的笔记，详细内容请查看[官方文档][bs4]
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
## Tag 标签
Tag对象即XML或HTML原生文档中的标签。Tag对象有两个最重要的属性**name**和**attributes**
~~~ python
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>')
tag = soup.b
type(tag)
# <class 'bs4.element.Tag'>
tag.name
# u'b'
tag['class']
# u'boldest'
~~~
**多值属性：**最常见的多值的属性是 class (一个tag可以有多个CSS的class)，Beautiful Soup中多值属性的返回类型是list。
~~~ python
css_soup = BeautifulSoup('<p class="body strikeout"></p>')
css_soup.p['class']
# ["body", "strikeout"]

css_soup = BeautifulSoup('<p class="body"></p>')
css_soup.p['class']
# ["body"]
~~~
如果某个属性看起来好像有多个值,但在任何版本的HTML定义中都没有被定义为多值属性,那么Beautiful Soup会将这个属性作为字符串返回
~~~ python
id_soup = BeautifulSoup('<p id="my id"></p>')
id_soup.p['id']
# 'my id
~~~
*如果转换的文档是XML格式,那么tag中不包含多值属性*
## NavigableString 可遍历的字符串
~~~ python
tag.string
# u'Extremely bold'
type(tag.string)
# <class 'bs4.element.NavigableString'>
~~~
NavigableString对象支持**遍历文档树**和**搜索文档树**中定义的大部分属性, 但并非全部.字符串不支持**.contents**或**.string**属性或**find()**方法.
如果需要在Beautiful Soup之外使用NavigableString对象,需要调用*unicode()*方法,将该对象转换成普通的Unicode字符串,否则就算Beautiful Soup已方法已经执行结束,该对象的输出也会带有对象的引用地址.这样会浪费内存.
## BeautifulSoup
BeautifulSoup对象表示的是一个文档的全部内容.支持**遍历文档树**和**搜索文档树**中定义的大部分功能。
大部分时候,可以把它当作**Tag**对象,但不是真正的**Tag**对象，所以它没有name和attribute属性。
## Comment 注释及特殊字符串
Comment对象是一个特殊类型的**NavigableString**对象。
~~~ python
markup = "<b><!--Hey, buddy. Want to buy a used parser?--></b>"
soup = BeautifulSoup(markup)
comment = soup.b.string
type(comment)
# <class 'bs4.element.Comment'>
comment
# u'Hey, buddy. Want to buy a used parser'
~~~
# 操作文档树
## 标签选择器
操作文档树最简单的方法就是告诉它你想获取的tag的name,如果想获取**head**标签,只要用**soup.head**.
可以在文档树的tag中多次调用这个方法,这是个获取tag的小窍门.
~~~ python
soup.head
# <head><title>The Dormouse's story</title></head>
soup.body.b
# <b>The Dormouse's story</b>
~~~
## .contents 和 .children
tag的.contents 属性可以将tag的直接子节点以列表的方式输出 <class 'list'>
tag的.children 属性可以获取tag的直接子节点的列表迭代器 <class 'list_iterator'>
~~~ python
head_tag = soup.head
head_tag
# <head><title>The Dormouse's story</title></head>
head_tag.contents
[<title>The Dormouse's story</title>]
title_tag = head_tag.contents[0]
title_tag
# <title>The Dormouse's story</title>
title_tag.contents
# [u'The Dormouse's story']

for child in title_tag.children:
    print(child)
    # The Dormouse's story
~~~
## .descendants
tag的.descendants 属性可以对所有tag的子孙节点进行递归循环 <class 'generator'>
~~~ python
for child in head_tag.descendants:
    print(child)
    # <title>The Dormouse's story</title>
    # The Dormouse's story
~~~
## .string
如果tag只有一个 NavigableString 类型子节点,那么这个tag可以使用 .string 得到子节点
如果一个tag仅有一个子节点,那么这个tag也可以使用 .string 方法,输出结果与当前唯一子节点的 .string 结果相同
如果tag包含了多个子节点,tag就无法确定 .string 方法应该调用哪个子节点的内容, .string 的输出结果是 None
~~~ python
title_tag.string
# u'The Dormouse's story'
head_tag.contents
# [<title>The Dormouse's story</title>]
head_tag.string
# u'The Dormouse's story'
print(soup.html.string)
# None
~~~
## .strings 和 stripped_strings
如果tag中包含多个字符串,可以使用 .strings 来循环获取
输出的字符串中可能包含了很多空格或空行,使用 .stripped_strings 可以去除多余空白内容
~~~ python
for string in soup.stripped_strings:
    print(repr(string))
    # u"The Dormouse's story"
    # u"The Dormouse's story"
    # u'Once upon a time there were three little sisters; and their names were'
    # u'Elsie'
    # u','
    # u'Lacie'
    # u'and'
    # u'Tillie'
    # u';\nand they lived at the bottom of a well.'
    # u'...'
~~~
全部是空格的行会被忽略掉,段首和段末的空白会被删除
## .parent 父节点
通过 .parent 属性来获取某个元素的父节点
文档的顶层节点`<html>`的父节点是 BeautifulSoup 对象
BeautifulSoup 对象的 .parent 是None
~~~ python
title_tag = soup.title
title_tag
# <title>The Dormouse's story</title>
title_tag.parent
# <head><title>The Dormouse's story</title></head>
title_tag.string.parent
# <title>The Dormouse's story</title>
print(soup.parent)
# None
~~~
## .parents 父辈节点
.parents 属性可以递归得到元素的所有父辈节点
~~~ python
link = soup.a
link
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
for parent in link.parents:
    if parent is None:
        print(parent)
    else:
        print(parent.name)
# p
# body
# html
# [document]
# None
~~~
## .next_sibling 和 .previous_sibling
在文档树中,使用 .next_sibling 和 .previous_sibling 属性来查询兄弟节点
**实际文档中的tag的 .next_sibling 和 .previous_sibling 属性通常是字符串或空白**
~~~ python
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a>
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>
~~~
如果以为第一个`<a>`标签的 .next_sibling 结果是第二个`<a>`标签,那就错了,真实结果是第一个`<a>`标签和第二个`<a>`标签之间的顿号和换行符
~~~ python
link = soup.a
link
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

link.next_sibling
# u',\n'
link.next_sibling.next_sibling
# <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
~~~
## .next_siblings 和 .previous_siblings
通过 .next_siblings 和 .previous_siblings 属性可以对当前节点的兄弟节点迭代输出
## .next_element 和 .previous_element
.next_element 属性指向解析过程中下一个被解析的对象(字符串或tag),结果可能与 .next_sibling 相同,但通常是不一样的
## .next_elements 和 .previous_elements
通过 .next_elements 和 .previous_elements 的迭代器就可以向前或向后访问文档的解析内容,就好像文档正在被解析一样

# 搜索文档树
Beautiful Soup定义了很多搜索方法,find() 和 find_all()是最重要的两个,其它方法的参数和用法类似
## 过滤器
搜索方法需要接受一个过滤器作为参数，过滤器的类型有5种:**字符串**,**正则表达式**,**列表**,**True**,**方法**
### 字符串
下面的例子用于查找文档中所有的`<b>`标签
~~~ python
soup.find_all('b')
# [<b>The Dormouse's story</b>]
~~~
### 正则表达式
如果传入正则表达式作为参数,Beautiful Soup会通过正则表达式的 match() 来匹配内容
下面例子中找出所有以b开头的标签,这表示`<body>`和`<b>`标签都应该被找到
~~~ python
import re
for tag in soup.find_all(re.compile("^b")):
    print(tag.name)
# body
# b
~~~
### 列表
如果传入列表参数,Beautiful Soup会将与列表中任一元素匹配的内容返回.下面代码找到文档中所有`<a>`标签和`<b>`标签
~~~ python
soup.find_all(["a", "b"])
# [<b>The Dormouse's story</b>,
#  <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
~~~
### True
True 可以匹配任何值,下面代码查找到所有的tag,但是不会返回字符串节点
~~~ python
for tag in soup.find_all(True):
    print(tag.name)
# html
# head
# title
# body
# p
# b
# p
# a
# a
# a
# p
~~~
### 方法
如果没有合适过滤器,那么还可以定义一个方法,方法只接受一个元素参数
如果这个方法返回 True 表示当前元素匹配并且被找到,如果不是则反回 False
~~~ python
def has_class_but_no_id(tag):
    return tag.has_attr('class') and not tag.has_attr('id')
soup.find_all(has_class_but_no_id)
# [<p class="title"><b>The Dormouse's story</b></p>,
#  <p class="story">Once upon a time there were...</p>,
#  <p class="story">...</p>]
~~~
## find_all
find_all( name , attrs , recursive , text , **kwargs )
find_all() 方法搜索当前tag的所有tag子节点,并判断是否符合过滤器的条件.这里有几个例子
### name 参数
name 参数可以查找所有名字为 name 的tag,字符串对象会被自动忽略掉.
~~~ python
soup.find_all("title")
# [<title>The Dormouse's story</title>]
~~~
搜索 name 参数的值可以使任一类型的**过滤器**
### keyword 参数
如果一个指定名字的参数不是搜索内置的参数名,搜索时会把该参数当作指定名字tag的属性来搜索,如果包含一个名字为 id 的参数,Beautiful Soup会搜索每个tag的”id”属性.
~~~ python
soup.find_all(id='link2')
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
~~~
如果传入 href 参数,Beautiful Soup会搜索每个tag的”href”属性:
~~~ python
soup.find_all(href=re.compile("elsie"))
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]
~~~
搜索指定名字的属性时可以使用的参数值包括**字符串**, **正则表达式**, **列表**, **True**.

使用多个指定名字的参数可以同时过滤tag的多个属性:
~~~ python
soup.find_all(href=re.compile("elsie"), id='link1')
# [<a class="sister" href="http://example.com/elsie" id="link1">three</a>]
~~~
可以通过 find_all() 方法的 attrs 参数定义一个字典参数来搜索包含特殊属性的tag:
~~~ python
data_soup.find_all(attrs={"data-foo": "value"})
# [<div data-foo="value">foo!</div>]
~~~
### 按CSS搜索
按照CSS类名搜索tag的功能非常实用,但标识CSS类名的关键字 class 在Python中是保留字,使用 class 做参数会导致语法错误
从Beautiful Soup的4.1.1版本开始,可以通过 **class_** 参数搜索有指定CSS类名的tag
**class_** 参数同样接受上面的5种过滤器
tag的 class 属性是多值属性。按照CSS类名搜索tag时,可以分别搜索tag中的每个CSS类名
~~~ python
css_soup = BeautifulSoup('<p class="body strikeout"></p>')
css_soup.find_all("p", class_="strikeout")
# [<p class="body strikeout"></p>]

css_soup.find_all("p", class_="body")
# [<p class="body strikeout"></p>]
~~~
搜索 class 属性时也可以通过CSS值完全匹配
完全匹配 class 的值时,如果CSS类名的顺序与实际不符,将搜索不到结果
### text 参数
通过 text 参数可以搜搜文档中的字符串内容.与 name 参数的可选值一样, text 参数接受 字符串 , 正则表达式 , 列表, True 
~~~ python
soup.find_all(text="Elsie")
# [u'Elsie']

soup.find_all(text=["Tillie", "Elsie", "Lacie"])
# [u'Elsie', u'Lacie', u'Tillie']
~~~
### limit 参数
find_all() 方法返回全部的搜索结构,如果文档树很大那么搜索会很慢.如果我们不需要全部结果,可以使用 limit 参数限制返回结果的数量
~~~ python
soup.find_all("a", limit=2)
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
~~~
### recursive 参数
调用tag的 find_all() 方法时,Beautiful Soup会检索当前tag的所有子孙节点,如果只想搜索tag的直接子节点,可以使用参数 recursive=False
~~~ python
soup.html.find_all("title")
# [<title>The Dormouse's story</title>]

soup.html.find_all("title", recursive=False)
# []
~~~
### 简写方法
find_all()几乎是Beautiful Soup中最常用的搜索方法。
它的简写方法是把BeautifulSoup对象和 tag对象当作一个方法来使用,这个方法的执行结果与调用这个对象的 find_all() 方法相同。
下面的两种写法是等价的:
~~~ python
soup.find_all("a")
soup("a")

soup.title.find_all(text=True)
soup.title(text=True)
~~~
## find()
find( name , attrs , recursive , text , **kwargs )
使用find()方法相当于使用find_all方法并设置参数limit=1.
唯一的区别是 find_all() 方法的返回结果是值包含一个元素的列表,而 find() 方法直接返回结果.
find_all() 方法没有找到目标是返回空列表, find() 方法找不到目标时,返回 None
~~~ python
soup.find_all('title', limit=1)
# [<title>The Dormouse's story</title>]

soup.find('title')
# <title>The Dormouse's story</title>
~~~
soup.head.title 是**tag的名字**方法的简写.这个简写的原理就是多次调用当前tag的find()方法
~~~ python
soup.head.title
# <title>The Dormouse's story</title>

soup.find("head").find("title")
# <title>The Dormouse's story</title>
~~~
## find_parents() 和 find_parent()
find_all() 和 find() 只搜索当前节点的所有子节点,子孙节点等. 
find_parents() 和 find_parent() 用来搜索当前节点的父辈节点,搜索方法与普通tag的搜索方法相同,搜索文档搜索文档包含的内容. 
搜索父辈节点的方法实际上就是对 .parents 属性的迭代搜索.
## find_next_siblings() 和 find_next_sibling()
.next_siblings 属性对当tag的所有后面解析的兄弟tag节点进行迭代
find_next_siblings() 方法返回所有符合条件的后面的兄弟节点
find_next_sibling() 只返回符合条件的后面的第一个tag节点.
## find_all_next() 和 find_next()
.next_elements 属性对当前tag的之后的tag和字符串进行迭代
find_all_next() 方法返回所有符合条件的节点
find_next() 方法返回第一个符合条件的节点
## find_all_previous() 和 find_previous()
.previous_elements 属性对当前tag的之前的tag和字符串进行迭代
find_all_previous() 方法返回所有符合条件的节点
find_previous() 方法返回第一个符合条件的节点

# CSS选择器
在Tag或BeautifulSoup对象的 .select() 方法中传入字符串参数,即可使用CSS选择器的语法找到tag
~~~ python
soup.select("title")
# [<title>The Dormouse's story</title>]
~~~
## 通过tag标签逐层查找
~~~ python
soup.select("html head title")
# [<title>The Dormouse's story</title>]
~~~
## 通过tag标签逐层查找
~~~ python
soup.select("html head title")
# [<title>The Dormouse's story</title>]
~~~
## 找到兄弟节点标签
~~~ python
soup.select("#link1 ~ .sister")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie"  id="link3">Tillie</a>]

soup.select("#link1 + .sister")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
~~~
## 通过CSS的类名查找
~~~ python
soup.select(".sister")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("[class~=sister]")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
~~~
## 通过tag的id查找
~~~ python
soup.select("#link1")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select("a#link2")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
~~~
## 通过是否存在某个属性来查找
~~~ python
soup.select('a[href]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
~~~
## 通过属性的值来查找
~~~ python
soup.select('a[href="http://example.com/elsie"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select('a[href^="http://example.com/"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select('a[href$="tillie"]')
# [<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select('a[href*=".com/el"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]
~~~
## 通过语言设置来查找
~~~ python
multilingual_soup.select('p[lang|=en]')
# [<p lang="en">Hello</p>,
~~~
# 修改文档树
~~~ python
~~~


[bs4]: https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/