数据结构
---
## 数组

## 链表
## 树
### 二叉树



## 哈希
业界Java并行库[HashMap和ConcurrentHashMap](https://my.oschina.net/pingpangkuangmo/blog/817973) 

### 相关论文

[Split-Ordered Lists: Lock-Free Extensible Hash Tables](atl/hash/Split-Ordered_Lists.pdf)


这篇文章介绍了的hash结构，有如下特点
* 无锁化实现，拉链式解决冲突
* 桶的大小总是 2^n
* 所有的元素被组成一个全局的拉链
* 桶的分隔符用一个虚拟(dummy)节点进行分割(split)
* rehash策略只改变桶的大小，不修改 item

以两倍的方式扩容，如原桶长度是4，现在桶长度是8，那么
* 桶0中的元素会被分到桶0和桶4中
* 桶1中的元素会被分到桶1和桶5中
* 桶2中的元素会被分到桶2和桶6中
* 桶3中的元素会被分到桶3和桶7中

我们将原来桶0的元素可以分为按照最高位两类 &(2^n -1)，分别插入到对应的桶下即可。


具体请参考其开源实现 [split-ordered-list](https://github.com/mpastyl/split-ordered-list)

论文
[A SevenDimensional Analysis of Hashing Methods and its Implications on Query Processing](atl/hash/p96-richter.pdf)

评估了hash因子对查询的影响

## Trie
### 技术文章

[Trie](http://zh.wikipedia.org/wiki/Trie) 是一种检索结构


[从Trie树到双数组Trie树](https://cloud.tencent.com/developer/article/1057813)



[爆炸式字典树](https://cloud.tencent.com/developer/news/380648)


[Benchmark of major hash maps implementations](https://tessil.github.io/2016/08/29/benchmark-hopscotch-map.html)

### 相关论文
[HAT-trie: A Cache-conscious Trie-based Data Structure for Strings](atl/hash/HAT_trie.pdf)

[hat-trie](https://tessil.github.io/2017/06/22/hat-trie.html)

提出了一种 hash 主要思想为

## 图
