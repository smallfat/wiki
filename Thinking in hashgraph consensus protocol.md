---
title: hashgraph共识协议读书笔记
grammar_cjkRuby: true
---
## 基本概念
- event - 实质上是block 区块。包含指向本节点最新event的hash指针和指向gossip来源event的hash指针，是"gossip about gossip"的设计实现。Hashgraph上的每一个顶点(circle)代表一个block。The Block is signed by creator to prevent tampering. The block contains a payload of anytransactions that creator chooses to create（在block创建时，block创建者能够随意加入她想加入的任何交易。）
- hashgraph - 一种数据结构，记录了以什么顺序，谁gossip了谁
- Gossip about gossip - hashgraph在gossip协议的基础上进行传播扩散。Alice会在gossip传播的时候告诉Bob**所有她知道的信息**，包括txs，2 hash， created time， set of ansestor 's hash等等，这样循环往复，最终每个node上都有全节点的传播路径即hashgraph了
- 可见 seeing - 区块x可见区块y意味着y是x的祖先区块
- 强可见 strongly seeing - 区块x强可见区块y意味着y是x的祖先区块且x到y的所有路径跨过了2/3以上的节点
- 轮次 round - 分为创建轮(created round)和接受轮(received round)
- 创建轮(created round) - 假设一个区块的创建轮次是R或者R+1，其中R是该区块父节点的最大轮次。当且仅当区块能**强可见****绝对多数**的R轮见证人，则该区块的创建轮次为R+1
- 接受轮(received round) - 第r轮（创建轮次）中所有的**知名见证者可见**的任何**普通区块**都会被赋予第r轮的接受轮次
- 见证人 witness - 每个节点在每个轮次中创建的第一个事件就是见证人事件，即该轮次的祖先事件，节点可能在某个轮次中没有见证人事件
- 知名见证人 famous witness - 如果第r轮的见证人区块能被绝对多数的第r+1轮的见证人可见，那么他就是知名见证人

## 理论
 - cornerstone of Byzantine Fault Tolerance 1 - 如果x和y是一个不合法fork的两个不同分支，则w最多只能强可见x和y中的一个。
 - cornerstone of Byzantine Fault Tolerance 2 - 如果hashgraph A和B是一致的(consistent)，则以下是不可能的：在A中x能被一个block强可见，在B中y也能被另外的block强可见

## 共识算法

 1. step 1 - Alice 随机选择一个成员Bob，gossip给他自己所知道的所有block的信息。Bob随后创建一个新的event去记录这些信息。
 2. step 2 - 
 3. step 3 - 
 4. 

## 问题
- 虚拟投票是指在单节点中依托全节点event数据（hash graph）完成全网的投票，而无需进行大量的网络通信。单节点中的全节点hash graph是怎么形成的呢？
- 同步网络与异步网络中的共识协议，其区别是什么？
- 达到什么条件时，网络中的各节点才能有一样的hashgraph呢？
- 

## 材料
- [Hashgraph video by Leemon Baird](https://www.youtube.com/watch?v=wgwYU1Zr9Tg)