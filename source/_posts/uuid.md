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
| 1 Bit Unused | 41 Bit Timestamp |  10 Bit NodeID  |   12 Bit Sequence ID |
+--------------------------------------------------------------------------+
```

- 使用41位来存储具有毫秒精度的时间戳
- 10位用于存储节点ID
- 12位用于存储序号


## 实现细节

- 生成41位具有毫秒精度的时间戳
- 将10位的节点ID追加到后面
- 然后添加序列数，从0开始，在相同的毫秒中生成步长为1的递增序列，如果在相同的毫秒中生成的序列溢出了，则停止生成等待下一毫秒

 ## 理论性能

 1毫秒可以生成12位（0-4095）4096个UUID，理论上单机可以生成4096000个UUID/s

 ## 关键代码实现

coding


 ## 性能测试

 coding