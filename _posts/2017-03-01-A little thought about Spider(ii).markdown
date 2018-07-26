---
layout: post
title:  "A little thought about Spider(ii)"
date:   2017-03-01 18:39:31 +0800
categories: ["Spider"]
tags: ["Python", "Spider"]
---
# 0x00
在这篇文章中，我将记录一个真实的爬虫案例。该爬虫基于Scrapy框架，爬取[Fanfiction][fanfiction-index]中有关Zootopia的所有评论数量超过100的同人文。
关于Scrapy框架的使用方法我便不多介绍，其[官方文档][scrapy-docs]中有详尽的描述。
接下来，按照上一篇文章的顺序，我们逐步解析一下这个案例：
# 0x01 关于入口地址
Scrapy框架采用面向对象方式，许多方法都已事先封装好。因此，我们只需傻瓜式地在我们的爬虫类中定义一些属性：

```python
name = "fanficzoo"    #爬虫名称
allowed_domains = ["www.fanfiction.net"]    #爬虫的活动范围
start_urls = ["https://www.fanfiction.net/search.php?ready=1&keywords=zootopia&type=story&ppage=1"]    #入口地址
```
<!-- more -->
# 0x02 关于解析方式
Fanfiction的前端代码写得比较清晰，因此可以通过Scrapy轻松解析。我们需要在文章列表中，爬取文章的标题、链接、作者、简介、内容信息。
```python
info = artical.css("div.z-padtop2::text").extract_first()

#...

yield {
    "arturl": "https://www.fanfiction.net" + artical.css("a.stitle::attr(href)").extract_first(),
    "title" : re.search(r'<img .*>(.*?)</a>', re.sub(r'</?b>', '', artical.css("a.stitle").extract_first())).group(1),
    "author": re.sub(r'-', ' ', artical.css("a::attr(href)").re(r'\/u\/\d*\/(.*)')[0]),
    "disc"  : re.search(r'z-padtop">(.*?)<div class="z-padtop2', re.sub(r'</?b>', '', artical.css("div.z-indent").extract_first())).group(1),
    "info"  : info
}
```

充分分析整个前端代码的结构之后，我们可以发现，评论（Reviews）数量就藏在内容信息之中：

```
Zootopia - Rated: K+ - English - Romance - Chapters: 2 - Words: 3,590 - Reviews: 11 - Favs: 32 - Follows: 34 - Published: May 3, 2016 - Judy H., Nick W
```

想要匹配出`Reviews: 11`，就要运用一些正则表达式的知识：

```python
info = artical.css("div.z-padtop2::text").extract_first()
isexist = re.search(r'Reviews:\s(.*?)\s', info)
```

在进行一些判断后，过滤出该页Reviews数量大于100的文章。

```python
isexist = re.search(r'Reviews:\s(.*?)\s', info)
if isexist and int(isexist.group(1)) >= 100:
```

到这里，基本的解析便介绍完毕。
# 0x03 关于寻址翻页
大量的文章不可能陈列在一页中，需要爬虫识别出标识页码的链接，并能判断出是否到了末页。一旦进入下一页，就要再次调用解析时需要的回调函数。

```python
pagelist = response.css("center").css("a")
nextpage = pagelist[-1]
if re.match(r'^Next', nextpage.css("a::text").extract_first()):
    url = response.urljoin("https://www.fanfiction.net/search.php" + nextpage.css("a::attr(href)").extract_first())
    yield scrapy.Request(url, self.parse)
```

# 0x04 一些问题和总结
+ Scrapy内置了一些固定的方法，可以直接调用某些方法的返回值。我们需要做的便是定义好用作解析、寻址等功能的回调函数。
+ 某一些文章没有`Reviews`这一参数，在使用正则表达式循环匹配的时候容易报错，因此需要加上一个`if`语句进行判断。
+ 完整代码可以在[FirSpider][firspider]中找到。

[fanfiction-index]: https://www.fanfiction.net/
[scrapy-docs]: https://doc.scrapy.org/en/latest/index.html
[firspider]: https://github.com/Songcoming/FirSpider

