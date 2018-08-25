---
title: Code a simple blockchain
date: 2018-04-21 16:29:54
tags: blockchain
intro : Blockchain principle
comments: false
---

# Blockchain
本文介绍区块链的数据结构以及最简单的单机实现（不包括pow, dpos等共识算法）

首先定义一个最基本的block结构，其中包括区块高度，时间戳，交易信息，上个区块的Hash和这个区块的Hash
```go
type Block struct {
	Index     int
	Timestamp int64
	Data      string
	Hash      string
	PrevHash  string
}
```
因为Hash函数是一种单向加密的函数，无法逆向计算出原始值（除非通过暴力穷尽），而且彩虹表对这种复杂Hash根本没办法。所有这个机制是保证区块链难以篡改的原因之一，其他还有共识机制等。


接下来实现其中最重要的方法生产区块
```go
// Calculate the block's hash
func CalculateHash(block Block) string {
	record := string(block.Index) + string(block.Timestamp) + string(block.Data) + block.PrevHash
	h := sha256.New()
	h.Write([]byte(record))
	hashed := h.Sum(nil)
	return hex.EncodeToString(hashed)
}

// Generate a new block by previous block
func GenerateBlock(oldBlock Block, data string) (Block, error) {
	var newBlock Block
	t := time.Now()
	newBlock.Index = oldBlock.Index + 1
	newBlock.Timestamp = t.Unix()
	newBlock.Data = data
	newBlock.PrevHash = oldBlock.Hash
	newBlock.Hash = CalculateHash(newBlock)

	return newBlock, nil
}
```
注：CalculateHash() 方法是计算下一个区块的哈希值，GenerateBlock（）是生成区块。


区块链的第一个区块叫做创世区块，是由系统初始化的时候生成的，自然它的PrevHash值就是空。
```go
t := time.Now()
genesisBlock := core.Block{0, t.Unix(), "", "", ""}
```