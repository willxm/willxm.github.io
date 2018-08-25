---
title: 'MIT 6.824 Lab 1: MapReduce'
date: 2018-08-18 01:32:59
tags: MapReduce
intro : MapReduce 学习笔记
comments: false
---

# MapReduce

MapReduce 是一个编程模型，也是一个处理和生成超大数据集的算法模型的相关实现。用户首先创建一 个 Map 函数处理一个基于 key/value pair 的数据集合，输出中间的基于 key/value pair 的数据集合;然后再创建 一个 Reduce 函数用来合并所有的具有相同中间 key 值的中间 value 值。


