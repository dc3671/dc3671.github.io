---
layout: post
title:  "相似抽取"
date:   2015-06-13 12:00:00
categories: algorithm
excerpt: Similarity Extract 相似抽取 字符串
---

* content
{:toc}


## 问题介绍

这是学校一门课的一个作业，类似于在百科页面里一个词条的文章里找到可能跟已有词条匹配的文字，从而可以链接到其他词条，问题描述如下：

Input：一个包含所有词条的集合，一个文档，相似度的阈值
Output：文档中所有在阈值范围内与词条相似的位置和长度，以及对应的词条id

## 算法简介

只是为了理清自己写作业的思路，将算法的大致流程描述如下：

1. 将所有词条根据一定的方法进行分段，可以选择直接平均分或者根据动态规划找出以词频为标准的最优分段，段的数量为阈值t加1，根据鸽巢原理，文档中的子串若与之相似，必然有一段完全匹配。

2. 将所有分段建成Trie树，树的叶结点存每个词条Entity的标号。如果要建subtrie，则叶结点左右两边各一个subtrie，按照从近到远建立

3. 以上是索引部分，查询部分需要从文档的头部开始，依次以每一个位置作为开头，在Trie树里进行查找，如果能够一直匹配到叶结点，说明有一段能完全匹配

4. 对于能完全匹配的段，取词条将其两边的剩余部分与文档中的对应部分进行ED计算，若总的ED之和小于阈值，即是一个结果。
