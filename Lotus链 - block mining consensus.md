---
title: Lotus链 - block mining consensus : Expected Consensus
tags: 
grammar_cjkRuby: true
---

# Lotus链的两种共识和出块机制
Lotus链是IPFS的激励层，主要作用是有效激励用户投入资源进行分布式存储。从区块链的角度考虑，必须形成共识保证block mining的正确性；从有效存储的角度考虑，亦需要形成共识保证storage mining的正确性。因此，Lotus内存在两种共识，即：block mining场景下的共识（Expected Consensus）和storage mining场景下的共识。
利用EC共识，系统在所有候选节点中，依据各节点当前有效存储量与总有效存储中的比例作为概率，来确定当前轮次的获胜者。由于按照概率来确定获胜者，每轮选举获胜者的数量可能是0，也有可能>1。获胜者有权利创建新的block，成为DAG（tipset）中的其中一个block。

# 期望共识（Expected Consensus）
EC共识是基于概率的拜占庭容错协议。每一轮会运行领导人选举，理想的情况下，有一名或一名以上候选人会赢得选举，从而获得出块的权利；在不理想的情况下，可能没有候选人赢得选举。赢家们会基于随机数计算选举的证明数据。在随后的轮次中，选举证明数据将会被验证。

### leader选举
Leader选举必须是秘密、公正、可验证的，系统使用随机数来确保这几个原则。在Lotus链中，存在一个独立的ticket链，这些tickets正是被用作随机数产生器的输入。
###### 随机数产生
The source of truth is defined below, but the currently defined DSTs are:
```
for drawing randomness from an on-chain ticket:
TicketDrawingDST = 1

for generating a new random ticket:
TicketProductionDST = 2

for generating randomness for running ElectionPoSt:
ElectionPoStChallengeSeedDST = 3

for generating randomness for running SurprisePoSt:
SurprisePoStChallengeSeedDST = 4

for selection of which miners to surprise:
SurprisePoStSelectMinersDST = 5

for selection of which sectors to sample:
SurprisePoStSampleSectors = 6
```
For a given ticket's randomness ticket_randomness:
```
buffer = Bytes{}
buffer.append(IntToBigEndianBytes(AppropriateDST))
buffer.append(-1) // a flag to be used in cases where FIL might need longer randomness outputs. Currently unused
buffer.append(ticket_randomness)
buffer.append(other needed serialized inputs)

randomness = SHA256(buffer)
```

###### 进行选举


### 选举证明
###### PoSt
###### VRF
###### zkSNARK

### 共识实现
