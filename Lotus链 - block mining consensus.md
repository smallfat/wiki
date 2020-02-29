---
title: Lotus链 - Expected Consensus(期望共识协议) 一
tags: 
grammar_cjkRuby: true
---

# Lotus链的两种共识和出块机制
Lotus链是IPFS的激励层，主要作用是有效激励用户投入资源进行分布式存储。从区块链的角度考虑，必须形成共识保证block mining的正确性；从有效存储的角度考虑，亦需要形成共识保证storage mining的正确性。因此，Lotus内存在两种共识，即：block mining场景下的共识（Expected Consensus）和storage mining场景下的共识。
利用EC共识，系统在所有候选节点中，依据各节点当前有效存储量与总有效存储中的比例作为概率，来确定当前轮次的获胜者。由于按照概率来确定获胜者，每轮选举获胜者的数量可能是0，也有可能>1。获胜者有权利创建新的block，成为DAG（tipset）中的其中一个block。

# 期望共识（Expected Consensus）
EC共识是基于概率的拜占庭容错协议。每一轮会运行领导人选举，理想的情况下，有一名或一名以上候选人会赢得选举，从而获得出块的权利；在不理想的情况下，可能没有候选人赢得选举。赢家们会基于随机数计算选举的证明数据。在随后的轮次中，选举证明数据将会被验证。

### leader选举
Leader选举必须是秘密、公正、可验证的。

随机数被用在选举过程中，以确保其秘密性。在Lotus链中，存在一个独立的ticket链，这些tickets正是被用作随机数产生器的输入。

选举，首先需要产生候选人，然后从候选人中选出获胜者。选举完成后，在随后的轮次中，还需要持续的验证此次选举的正确性，这正是选举的可验证性。这个过程被称为ElectionPoSt，即选举时空证明。
选举的公正性是指：
- 按照某miner的有效存储数与总网络存储数的比例作为获胜概率来进行选举
  这决定了有效sector中的内容必须被纳入到选举的计算中

- 对单个节点而言，这次选举是针对该节点上的miner而进行的
  因此miner的id应该也要纳入到选举的计算中 

Lotus链进行选举的流程主要包括：
- 产生随机数
- 生成Partial Tickets
- 生成Challenge Tickets
- 进行选举

选举流程保证了秘密、公正和可验证性。下面分别进行说明。

##### 产生随机数
在当前轮次，随机数的生成公式如下：
```
post_randomness = VRF(minerID || currentBlockHeight || ChainRandomness(currentBlockHeight - SPC_LOOKBACK_POST))
```
由公式可以看出
- 随机数种子 - 当前轮次前SPC_LOOKBACK_POST轮的随机数，叠加minerID，再叠加当前轮次高度
- 使用VRF（可验证随机函数）来生成随机数，具体使用的算法由VRF的实现决定

##### 生成Partial Tickets
1. miner需要从他的有效存储sectors中随机选择numSectorsSampled个合格的sector，用作样本sector。
2. 将每一个sample sector的内容划分为n个固定大小的数据块，编号为B_1 ~ B_n。从这n个数据块中，随机选出k个数据块(k<n)，数据块的内容分别记为C_1_Output ~ C_k_Output。
3. 针对每一个sample sector，Miner使用如下函数生成PartialTicket：
```
PartialTicket = Hash(post_randomness || minerID || C_1_Output || … || C_k_Output)
```
由函数可以看出，之前步骤生成的随机数post_randomness，miner id，数据块内容C_x_Output都作为参数连接到一起，然后做hash操作，生成Partial Ticket
由于随机数的存在，可以认为，Partial Ticket的值也是一个随机数

4. 对每一个sample sector做类似操作，最后一共生成numSectorsSampled个Partial Ticket

##### 生成Challenge Tickets

Challenge Ticket的计算方法如下：
```
challengeTicket = finalizeTicket(PartialTicket) 

def finalizeTicket(partialTicket):
    return Hash(partialTicket)
```
由此可知，Challenge Ticket也可以认为是一个随机数，不可预测。Hash算法需要保证其值在整个Hash值域是均匀分布的。

##### 进行选举
- 理论基础
由于Challenge Ticket的均匀分布特性，那么只要将其进行归一化(Normalization)为(0,1)值区间后，与一个阈值Target进行大小比较，若小于Target值，则选举成功。Target就是选举获胜概率。如下公式所示：
```
const MaxChallengeTicketSize = 2^len(Hash)
ChallengeTicket/MaxChallengeTicketSize < Target
```
而选举获胜概率是由miner的有效存储值与整个网络有效存储值的比值决定的。公式演变为：
```
const MaxChallengeTicketSize = 2^len(Hash)
ChallengeTicket/MaxChallengeTicketSize < ActivePowerInSector/NetworkPower

*ActivePowerInSector: 扇区有效存储量
*NetworkPower：       网络总有效存储量
```

- 伪代码
```
winningTickets = []
def checkTicketForWinners(partialTickets):
    for partialTicket in partialTickets:
        challengeTicket = finalizeTicket(PartialTicket) 
        if TicketIsWinner(challengeTicket):
            winningTickets += partialTicket
			
const maxChallengeTicketSize = 2^len(Hash)

def TicketIsWinner(challengeTicket):
    // Check that `ChallengeTicket < Target`
    return ChallengeTicket * NetworkPower < ActivePowerInSector * MaxChallengeTicketSize
```

由于计算效率的关系，EC共识协议的实现做了一些优化，在以后的文章中会谈到。

# 结语
EC共识利用概率计算实现了无交互式的Leader选举，实现了同时产生多个块的DAG链结构，有效地提高了链的出块效率，扩展了链容量。