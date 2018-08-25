---
title: Distributed uuid generation written with golang
date: 2018-04-17 23:38:31
tags: uuid
intro : Inspired by twitter snowflake
comments: false
---
# UUID



## 描述

UUID (Universal Unique Identifier) 是存储在 int64 中的63位整数，由3部分构成，格式如下:

```
+--------------------------------------------------------------------------+
| 1 Bit Unused | 39 Bit Timestamp |  16 Bit NodeID  |   8 Bit Sequence ID |
+--------------------------------------------------------------------------+
```

- 使用39位来存储具有毫秒精度的时间戳
- 16位用于存储节点ID
- 8位用于存储序号


## 实现细节

- 生成39位具有毫秒精度的时间戳
- 将16位的节点ID追加到后面
- 然后添加序列数，从0开始，在相同的毫秒中生成步长为1的递增序列，如果在相同的毫秒中生成的序列溢出了，则停止生成等待下一毫秒

 ## 理论性能

 1毫秒可以生成8位（0-255）256个UUID，理论上单机可以生成256000个UUID/s

 ## 关键代码实现
 ```go
 func (sf *Snowflake) UUID() (uint64, error) {
	const maskSequence = uint16(1 << BitLenSequence - 1)

	sf.mutex.Lock()
	defer sf.mutex.Unlock()

	current := currentElapsedTime(sf.startTime)
	if sf.elapsedTime < current {
		sf.elapsedTime = current
		sf.sequence = 0
	} else {
        //如果sequence位满了，需要等下一个时间片段
		sf.sequence = (sf.sequence + 1) & maskSequence
		if sf.sequence == 0 {
			sf.elapsedTime++
			overtime := sf.elapsedTime - current
			time.Sleep(sleepTime((overtime)))
		}
	}
	return sf.toID()
}
```



 ## 性能测试
```go
func BenchmarkSnowflake_UUID(b *testing.B) {

	sf := NewSnowflack()

	for n := 0; n < b.N; n++ {
		sf.UUID()
	}
}
```

```shell
$ go test -bench="."
goos: darwin
goarch: amd64
pkg: github.com/willxm/snowflake
BenchmarkSnowflake_UUID-4          50000             39036 ns/op
PASS
ok      github.com/willxm/snowflake     2.350s
```

1e9/39036=25617.3788298