---
layout: post
book: true
background-image: http://ot1cc1u9t.bkt.clouddn.com/17-7-16/91630214.jpg
category: 算法/数据结构
title: BloomFilter
tags:
- BloomFilter
---

>Bloom Filter是由Bloom在1970年提出的一种多哈希函数映射的快速查找算法。通常应用在一些需要快速判断某个元素是否属于集合，但是并不严格要求100%正确的场合。

场景
===
假设要你写一个网络蜘蛛（web crawler）。由于网络间的链接错综复杂，蜘蛛在网络间爬行很可能会形成“环”。为了避免形成“环”，就需要知道蜘蛛已经访问过那些URL。给一个URL，怎样知道蜘蛛是否已经访问过呢？稍微想想，就会有如下几种方案：

1. 将访问过的URL保存到数据库。
2. 用HashSet将访问过的URL保存起来。那只需接近O(1)的代价就可以查到一个URL是否被访问过了。
3. URL经过MD5或SHA-1等单向哈希后再保存到HashSet或数据库。
4. Bit-Map方法。建立一个BitSet，将每个URL经过一个哈希函数映射到某一位。

方法1~3都是将访问过的URL完整保存，方法4则只标记URL的一个映射位。

以上方法在数据量较小的情况下都能完美解决问题，但是当数据量变得非常庞大时问题就来了。

***方法1的缺点***

数据量变得非常庞大后关系型数据库查询的效率会变得很低。而且每来一个URL就启动一次数据库查询,代价略大。

***方法2的缺点***

太消耗内存。随着URL的增多，占用的内存会越来越多。就算只有1亿个URL，每个URL只算50个字符，就需要5GB内存。

***方法3的缺点***

由于字符串经过MD5处理后的信息摘要长度只有128Bit，SHA-1处理后也只有160Bit，因此方法3比方法2节省了好几倍的内存。

***方法4的缺点***

消耗内存是相对较少的，但缺点是单一哈希函数发生冲突的概率太高。Hash表冲突的时候会把冲突的键里建一个链表，将多个值放到链表里，但链表的随机访问很慢。若要降低冲突发生的概率到1%，就要将BitSet的长度设置为URL个数的100倍。

实质上上面的算法都忽略了一个重要的隐含条件：允许小概率的出错，不一定要100%准确！也就是说少量url实际上没有没网络蜘蛛访问，而将它们错判为已访问的代价是很小的——大不了少抓几个网页。

Bloom Filter的算法 
===
上面方法4的思想已经很接近Bloom Filter了。方法四的致命缺点是冲突概率高，为了降低冲突的概念，Bloom Filter使用了多个哈希函数，而不是一个。

Bloom Filter算法如下
---
创建一个 m 位BitSet，先将所有位初始化为0，然后选择k个不同的哈希函数。第i个哈希函数对字符串str 哈希的结果记为h(i，str)，且h(i，str)的范围是 0 到 m-1。

***记录字符串过程***

下面是每个字符串处理的过程，首先是将字符串str“记录”到BitSet中的过程：

对于字符串str，分别计算 h(1，str)，h(2，str)…… h(k，str)。然后将BitSet的第 h(1，str)、h(2，str)…… h(k，str)位设为1。

这样就将字符串str映射到BitSet中的k个二进制位了。

***检查字符串是否存在的过程***

下面是检查字符串str是否被BitSet记录过的过程：

对于字符串str，分别计算 h(1，str)，h(2，str)…… h(k，str)。然后检查 BitSet的第 h(1，str)、h(2，str)…… h(k，str)位是否为1，若其中任何一位不为1则可以判定 str一定没有被记录过。若全部位都是1，则“认为”字符串str存在(也有可能不存在，因为其它字符串计算出来的值在某个位置上可能和当前字符串有交叉)。

***删除字符串过程***

字符串加入了就被不能删除了，因为删除会影响到其他字符串。

> Bloom Filter跟单哈希函数Bit-Map不同之处在于：Bloom Filter使用了k个哈希函数，每个字符串跟k个bit对应。从而降低了冲突的概率。