---
title: Common cache algorithms written with go 
date: 2018-04-14 23:25:35
tags: cache
intro : 常见的cache算法，比如 LRU LFU FIFO
comments: false
---

# cache algorithms
常见的cache算法包括 LRU, LFU, FIFO 等。在代码中要实现这些算法，最简单的就是实现 Get(), Set() 方法。

- FIFO (First In First Out)

FIFO 是先进先出算法，数据结构可以用双向链表实现，新来的元素都插入到链表的尾部。如果容量满了，则删除头部元素再插入到尾部。访问元素的时候，如果存在返回value否则返回nil。以下是golang代码实现

```go
package cache

import (
	"container/list"
	"sync"
)

type FIFOCache struct {
	mu       sync.Mutex
	list     *list.List
	capacity int64
}

func (f *FIFOCache) Get(k interface{}) (interface{}, error) {
}

func (f *FIFOCache) Set(k, v interface{}) error {

}

```

- LFU (Least Frequently Used)

LFU 是最近最不常使用算法，通过为每个元素维护一个计数器，插入的时候计数器默认为1。如果容量满了，则删除计数器最小的元素，再插入新的。访问数据的时候，如果存在返回value并计数器+1否则放回nil。以下是golang代码实现

- LRU (Least Recently Used)

LRU 是最近最少使用算法，数据结构可以用双向链表实现，新来的元素插入到链表头部。如果容量满了，则删除尾部再插入到头部。访问数据的时候，如果存在返回value并移动该元元素到头部否则返回nil。以下是golang代码实现
 



