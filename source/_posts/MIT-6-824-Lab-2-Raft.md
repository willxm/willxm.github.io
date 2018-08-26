---
title: 'MIT 6.824 Lab 2: Raft'
date: 2018-08-26 18:00:46
tags: Raft
intro : Raft learning notes
comments: false
---

# Raft
Raft 是一种用来管理日志复制的一致性算法。它和 Paxos 的性能和功能是一样的，但是它和 Paxos 的结构不一样；这使得 Raft 更容易理解并且更易于建立实际的系统。为了提高理解性，Raft 将一致性算法分为了几个部分，例如领导选取（leader selection），日志复制（log replication）和安全性（safety），同时它使用了更强的一致性来减少了必须需要考虑的状态。