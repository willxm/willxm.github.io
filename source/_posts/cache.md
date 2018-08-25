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
	"fmt"
	"sync"
)

type fifoItem struct {
	Key   interface{}
	Value interface{}
}

type FIFOCache struct {
	amu      sync.Mutex
	rmu      sync.Mutex
	list     *list.List
	table    map[interface{}]*list.Element
	capacity int
}

func NewFIFOCache(cap int) *FIFOCache {
	return &FIFOCache{
		capacity: cap,
		list:     list.New(),
		table:    make(map[interface{}]*list.Element),
	}
}

func (f *FIFOCache) Count() int {
	return f.list.Len()
}

func (f *FIFOCache) Contains(key interface{}) bool {
	_, ok := f.table[key]
	return ok
}

func (f *FIFOCache) Get(key interface{}) interface{} {
	if element, ok := f.table[key]; ok {
		return element.Value.(*fifoItem).Value
	}
	return nil
}

func (f *FIFOCache) Set(key, value interface{}) {
	f.amu.Lock()
	defer f.amu.Unlock()
	if f.capacity > 0 && f.Count() == f.capacity && !f.Contains(key) {
		f.Remove(f.list.Front().Value.(*fifoItem).Key)
	}
	if element, ok := f.table[key]; ok {
		element.Value.(*fifoItem).Value = value
	} else {
		element := f.list.PushBack(&fifoItem{Key: key, Value: value})
		f.table[key] = element
	}
}

func (f *FIFOCache) Remove(key interface{}) {
	f.rmu.Lock()
	defer f.rmu.Unlock()
	if element, ok := f.table[key]; ok {
		f.list.Remove(element)
		delete(f.table, key)
	}
}

func (f *FIFOCache) Traverse() {
	data := f.list.Front()
	for {
		fmt.Println(data.Value.(*fifoItem))
		if data.Next() != nil {
			data = data.Next()
		} else {
			return
		}
	}
}

```

- LFU (Least Frequently Used)

LFU 是最近最不常使用算法，通过为每个元素维护一个计数器，插入的时候计数器默认为1。如果容量满了，则删除计数器最小的元素，再插入新的。访问数据的时候，如果存在返回value并计数器+1否则放回nil。以下是golang代码实现

- LRU (Least Recently Used)

LRU 是最近最少使用算法，数据结构可以用双向链表实现，新来的元素插入到链表头部。如果容量满了，则删除尾部再插入到头部。访问数据的时候，如果存在返回value并移动该元元素到头部否则返回nil。以下是golang代码实现

```go
package cache

import (
	"container/list"
	"fmt"
	"sync"
)

type lruItem struct {
	Key   interface{}
	Value interface{}
}

type LRUCache struct {
	amu      sync.Mutex
	rmu      sync.Mutex
	list     *list.List
	table    map[interface{}]*list.Element
	capacity int
}

func NewLRUCache(cap int) *LRUCache {
	return &LRUCache{
		capacity: cap,
		list:     list.New(),
		table:    make(map[interface{}]*list.Element),
	}
}

func (l *LRUCache) Count() int {
	return l.list.Len()
}

func (l *LRUCache) Contains(key interface{}) bool {
	_, ok := l.table[key]
	return ok
}

func (l *LRUCache) Get(key interface{}) interface{} {
	l.rmu.Lock()
	defer l.rmu.Unlock()
	if element, ok := l.table[key]; ok {
		l.list.MoveToBack(element)
		return element.Value.(*lruItem).Value
	}
	return nil
}

func (l *LRUCache) Set(key, value interface{}) {
	l.amu.Lock()
	defer l.amu.Unlock()
	if l.capacity > 0 && l.Count() == l.capacity && !l.Contains(key) {
		l.Remove(l.list.Front().Value.(*lruItem).Key)
	}
	if element, ok := l.table[key]; ok {
		element.Value.(*lruItem).Value = value
	} else {
		element := l.list.PushBack(&lruItem{Key: key, Value: value})
		l.table[key] = element
	}
}

func (l *LRUCache) Remove(key interface{}) {
	l.rmu.Lock()
	defer l.rmu.Unlock()
	if element, ok := l.table[key]; ok {
		l.list.Remove(element)
		delete(l.table, key)
	}
}

func (l *LRUCache) Traverse() {
	data := l.list.Front()
	for {
		fmt.Println(data.Value.(*lruItem))
		if data.Next() != nil {
			data = data.Next()
		} else {
			return
		}
	}
}

```
 



