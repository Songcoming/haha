---
layout: post
title:  "A little thought about Spider(i)"
date:   2017-02-27 19:22:11 +0800
tags:   ["Python", "Spider"]
categories: ["Spider"]
---
# 0x00
这段时间需要做一个项目，其中一个模块便是从一些特定网站收集信息。因此，爬虫成了实现该功能的不二工具。由于是第一次真正着手学习爬虫，因此我将在这里记录下学习过程和一点思考。
# 0x01
让我们先从最简单的爬虫做起。首先我们需要知道的，便是爬虫需要具备的基本功能：
* 向服务器发送请求，接收响应
* 解析接收的文件，并从中获取需要的信息

这大概便是爬虫的根基所在。至于识别进入下一链接以及查询重复链接等功能，则暂时不需讨论。<!-- more -->
用伪代码来描述以上功能：

```python
def parse(response):
    # parse HTML/XML...

if __name__ == '__main__':
    url = "http://example.com"  # 入口URL
    response = request(url)     # 获得响应结果

    with open("/your/filepath/filename.txt", "a", encoding = "UTF-8") as f:
        f.write(parse(response))
```

# 0x02
但是能够满足日常需求的爬虫不可能如此简单。很多时候，我们都需要结合特定情况来定制爬虫。
以下我们举一个例子，来爬取百度贴吧明星板块排名前列的贴吧名称、关注数、贴子数及贴吧描述。
在这里我没有使用爬虫框架，只是手动实现一只爬虫，以加深对其思想的理解：
### 要有一个入口

入口并不难找，[娱乐明星贴吧列表的第一页][first-page]就是一个不错的入口。要想对该链接发起请求，可以引用python自带的包。但是在这里我选择了“made for human”的[Requests][Requests-docs]包。总之，选择一个你喜欢的工具，来实现对目标的请求：

```python
import Requests

payload = {'cn': '', 'ci': 0, 'pcn': '娱乐明星', 'pci': 0, 'ct': 1, 'st': 'new', 'pn': page} 
r = requests.get('http://tieba.baidu.com/f/index/forumpark', params = payload)
```

`r`将作为一个重要句柄来获取你需要的内容（`r.text()`）。
### 要有一个解析方案

当我们完整地获得了响应内容，便要提取出我们需要的信息。Python有各种各样的`HTML/XML`解析模块供你选择。但是在这里我没有使用解析工具，而是手动实现解析：

```python
re_name  = re.compile(r'<p class="ba_name">(.*?)</p>')
re_m_num = re.compile(r'<span class="ba_m_num">(.*?)</span>')
re_p_num = re.compile(r'<span class="ba_p_num">(.*?)</span>')
re_desc  = re.compile(r'<p class="ba_desc">(.*?)</p>')

#...

ba_name  = "Ba_name:     " + re.search(re_name,  bacontent).group(1) + "\n"
ba_m_num = "Ba_m_number: " + re.search(re_m_num, bacontent).group(1) + "\n"
ba_p_num = "Ba_p_number: " + re.search(re_p_num, bacontent).group(1) + "\n"
ba_desc  = "Ba_desc:     " + re.search(re_desc,  bacontent).group(1) + "\n"
```

### 进阶问题

我们知道，几百个贴吧不可能同时陈列在一页中，因此这里涉及到进入下一页的问题。如果仔细分析网站的URL：

```
http://tieba.baidu.com/f/index/forumpark?cn=&ci=0&pcn=%E5%A8%B1%E4%B9%90%E6%98%8E%E6%98%9F&pci=0&ct=1&st=new&pn=1
```

我们会发现，`pn=1`这一参数，标识了贴吧列表的分页。接下来就是一个for循环的问题。

### 一些小问题

* 在解析过程中，我们总会不可避免地使用正则表达式。但由于标签的高度重复，我们需要注意贪婪与懒惰匹配模式的正确使用。

# 0x03
完整代码可在[TestSpider][testspider-url]中找到。下一篇我们要讨论一个爬虫框架的使用案例。

[first-page]: http://tieba.baidu.com/f/index/forumpark?cn=&ci=0&pcn=%E5%A8%B1%E4%B9%90%E6%98%8E%E6%98%9F&pci=0&ct=1&st=new&pn=1
[Requests-docs]: http://docs.python-requests.org/zh_CN/latest/user/quickstart.html
[testSpider-url]: https://github.com/Songcoming/TestSpider
