---
title: Go sync.Map source code analysis
date: 2018-04-27 00:41:45
tags: sync.Map
intro : Concurrency safety of map
comments: false
---

# Go sync.Map
以下基于 go version go1.10 darwin/amd64 源码

在go1.9版本之前，map不是并发安全的，需要通过Mutex加锁来实现。在1.9之后引入了新的sync.Map

## 数据结构 
```go
type Map struct {
    //在read中读取不到的时候，加锁
  	mu Mutex
  
    //只读的map，使用atomic保证原子性
  	read atomic.Value // readOnly
    //写入数据的map
  	dirty map[interface{}]*entry
    //用于记录miss数量
  	misses int
  }
```

```go
type readOnly struct {
  	m       map[interface{}]*entry
  	amended bool // true if the dirty map contains some key not in m.
  }
```

```go
  type entry struct {
  	p unsafe.Pointer // *interface{}
  }
  
```
