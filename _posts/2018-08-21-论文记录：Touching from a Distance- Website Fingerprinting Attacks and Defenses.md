---

layout: post
title: "论文记录：Touching from a Distance: Website Fingerprinting Attacks and Defenses"
date: 2018-08-21 15:03:00 +0800
tags: ["Encrypted Traffic Analysis", "Website Fingerprint"]
categories: ["Lectures"]

---

## 0x00
>本系列笔记是用来记录论文阅读过程中产生的问题与思考的随笔性质文本，结构可能比较松散，无法完全体现原论文的精髓之处，仅供自己日后温习参考之用。

* 题目：Touching from a Distance: Website Fingerprinting Attacks and Defenses
* 作者：Xiang Cai (Stony Brook University); Xin Cheng Zhang (Stony Brook University); Brijesh Joshi (Stony Brook University); Rob Johnson (Stony Brook University)
* 出处：CCS '12 Proceedings of the 2012 ACM conference on Computer and communications security
Pages 605-616 
* 关键词：Anonymity; website fingerprinting attacks

## 0x01 提出问题

* Web站点的识别攻击已有对应的防御之策。例如：HTTPOS和TOR会将请求拆分并打乱顺序。
* 以上两种防范方法皆可攻破。我们提出了能够识别TOR和SSH流量归属的方法。同时，我们还可以提出更好的防御方法。

## 0x02 解决方法
*  我们首先使用计算`编辑距离（Damerau-Levenshtein edit distance）`的方式来建立网页（Web pages）分类识别系统，随后我们使用`隐藏马尔科夫模型（Hidden Markov Models）`来建立网站（Web sites）分类识别系统。最后，我们改良了`BUFLO模式`，并以此作为一个新的防御方法。<!-- more -->

## 0x03 特征提取细节
### 网页分类
我们从每个网页中提取若干trace，每个trace为一个l维的向量，向量的元素为该trace中每个数据包的长度数值（有符号，用以标记方向）。随后，我们计算不同trace之间的编辑距离。所谓编辑距离，即为两个字串之间，由一个转成另一个所需的最少编辑操作次数。一般情况下，这些编辑操作包含`插入`、`替换`和`删除`。而在本文中，还包含`相邻字符的交换`。

为计算两个trace之间的相似度，我们使用一个与编辑距离有关的式子作为训练SVM的核，训练一个分类器。该分类器能够帮助我们对网页进行分类。

值得注意的是，为了防止ACK包对训练产生影响，我们在SSH分类中滤除了不大于84 bytes的包，在TOR分类中滤除了40 bytes和52 bytes的包。
### 网站分类

我们建立一个隐藏马尔科夫模型，每一个网页代表了模型的一种状态，而状态转化率代表了用户从一个网页被导航至另一个网页的概率。我们定义O为模型可能判定出的所有结果的集合，s为这个网页真实的归属。通过该模型，我们可以计算Pr[o|s],也即不同的结果出现的可能性。`（该部分涉及马尔可夫链，我能够理解的内容不多）`

值得注意的是，由于一些网站有大量相似的页面（如IMDB的每一个电影主页），我们可以使用一个网页作为其他相似页面的模板，来输入模型进行判定。

## 0x04 有拥塞控制功能的BUFLO
BUFLO是一个数据包填充系统。每p毫秒，它都将发送一个d bytes的数据包，这样的过程将持续t毫秒。如果数据包本身不足d bytes，那么系统将其自动填充至d bytes。该系统存在三个缺点`高开销`、`低可行性`以及`未知的安全性`。

因此，在改良的BUFLO系统中，我们设置了参数T，来代表清空输出序列的时间间隔。每隔T时间，系统查看输出序列的状况和网络状况，并借此作出拥塞控制。这个T控制了发送效率，因此需要将其设置为一个合理的数值，使整个系统机能保证高效，又不至于发送太多的无效填充数据。


