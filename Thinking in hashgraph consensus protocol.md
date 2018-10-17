---
title: hashgraph共识协议读书笔记
grammar_cjkRuby: true
---
## 基本概念
- event - 实质上是block 区块。包含指向本节点最新event的hash指针和指向gossip来源event的hash指针，是"gossip about gossip"的设计实现
- hashgraph - 一种数据结构，记录了以什么顺序，谁gossip了谁
- Gossip about gossip - hashgraph在gossip协议的基础上进行传播扩散。消息内容是gossip本身的历史
- 可见 seeing - 区块x可见区块y意味着y是x的祖先区块
- 强可见 strongly seeing - 区块x强可见区块y意味着y是x的祖先区块且x到y的路径跨过了2/3以上的节点
- 轮次 round - 分为创建轮(create round)和接受轮(receive round)
- 创建轮(create round) - 
- 接受轮(receive round) - 
- 见证人 witness - 
- 知名见证人 famous witness - 

## 问题
- 虚拟投票是指在单节点中依托全节点event数据（hash graph）完成全网的投票，而无需进行大量的网络通信。单节点中的全节点hash graph是怎么形成的呢？
- 同步网络与异步网络中的共识协议，其区别是什么？