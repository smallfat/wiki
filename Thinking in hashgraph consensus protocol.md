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
- 一致(consistent) - consistent means if there is event x in both hashgraph A and B, then the same parents of x both exists in A and B

## 理论
 - cornerstone of Byzantine Fault Tolerance 1 - 如果x和y是一个不合法fork的两个不同分支，则w最多只能强可见x和y中的一个。
 - cornerstone of Byzantine Fault Tolerance 2 - 如果hashgraph A和B是一致的(consistent)，则以下是不可能的：在A中x能被一个block强可见，在B中y也能被另外的block强可见

## 算法

 - 总体
 
![](./images/Hashgraph-figure4.png)
    
     - 每个成员随机选择目标成员，并向目标成员发送同步数据。
     - 在发送同步数据的同时，每个成员也接受同步数据。
     - Alice发送给Bob的数据，是Alice知道而Bob不知道的。
     - Bob只接受拥有合法签名blocks，然后将这些数据（主要是blocks），加入到hashgraph中
     - 将所有已知的blocks划分轮次
     
 - 划分轮次

![](./images/Figure5.png)

 - 确定witness和famous witness

![](./images/Figure6.png)

 - 确定交易次序

![](./images/Figure7.png)
- 普通block的条件：

## 定理与证明
- 所有的成员节点有consistent hashgraph
- 强可见定理 - if the pair of events(x,y) is a fork, and x is strongly seen by event z in hashgraph A, then y will not be strongly seen by any event in any hashgraph B that is consistent with A.
-  If hashgraphs A and B are consistent and both contain event x, then both will assign the same round created number to x.
-  If hashgraphs A and B are consistent, and the algorithm running on A shows for a given election that a round r witness by member m0 sends a vote vA to a witness created by member m1 in round r + 1, and the algorithm running on B shows that a round r witness by member m0 sends a vote vB to a witness by member m1 in round r + 1, then vA = vB.
-  If hashgraphs A and B are consistent, and A decides a Byzantine agreement election with result v in round r and B has not decided prior to r, then B will decide v in round r + 2 or before
-  If hashgraphs A and B are consistent, and A decides a Byzantine agreement election with result v in round r and B has not decided prior to r, then B will decide v in round r + 2 or before.
-  For any single YES/NO question, consensus is achieved eventually with probability 1.
-  For any round number r, for any hashgraph that has at least one event in round r+3, there will be at least one witness in round r that will be decided to be famous by the consensus algorithm, and this decision will be made by every witness in round r + 3, or earlier
-  If hashgraph A does not contain event x, but does contain all the  parents of x, and hashgraph B is the result of adding x to A, and x is a witness created in round r, and A has at least one witness in round r whose fame has been decided (as either famous or as not famous), then x will be decided as “not famous” in B.
-   (Byzantine Fault Tolerance Theorem). Each event x created by an honest member will eventually be assigned a consensus position in the total order of events, with probability 1.

## 问题
- 虚拟投票是指在单节点中依托全节点event数据（hash graph）完成全网的投票，而无需进行大量的网络通信。单节点中的全节点hash graph是怎么形成的呢？
- 同步网络与异步网络中的共识协议，其区别是什么？
- 达到什么条件时，网络中的各节点才能有一样的hashgraph呢？
- 

## 材料
- [Hashgraph video by Leemon Baird](https://www.youtube.com/watch?v=wgwYU1Zr9Tg)