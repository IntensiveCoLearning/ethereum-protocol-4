---
timezone: UTC+8
---

> 请在上边的 timezone 添加你的当地时区(UTC)，这会有助于你的打卡状态的自动化更新，如果没有添加，默认为北京时间 UTC+8 时区


# OrionWu

1. 自我介绍
我叫 Orion，来自福州，目前在一家初创公司从事合约相关开发，希望能对区块链底层有更深的理解。
2. 你认为你会完成本次残酷学习吗？ 
会的
1. 你的联系方式（推荐 Telegram）
https://t.me/orion_wu_x

## Notes

<!-- Content_START -->

### 2025.06.16

以太坊PoS作为验证者，必须满足以下条件：

1. 需要存入32个ETH作为抵押。
2. 运行执行层和共识层客户端以及验证者客户端。

solts(12s) 每个slot内只有0或1个区块。
epochs(32 slots)
每个slot随机选择一个验证者作为区块proposer，并且随机选择一组验证者集合来验证区块。

一笔交易发出先被执行客户端验证有效性，添加到本地内存池中，再通过执行层广播交易。当slots验证者接收到交易后，将交易从本地内存池中取出打包成payload，并生成状态更改，通知共识客户端。共识客户端将payload和状态更改打包到“信标区块”中，通过共识层广播信标区块。其他节点接收到信标区块后，交易在本地重放，确保状态更改正确。如果是验证者客户端，则对区块签名并加入到节点的本地数据库。
在每个epoch结束时，通过判断交易是否在“绝对多数链接”（获得超过网络总质押以太币数量66%的同意）中，来决定交易是否被最终确认。

例如：区块运行到epoch 5。epoch 4 slot 0是epoch 4的检查点，epoch 5 slot 32是epoch 5的检查点。此时epoch 4 slot 0被称作Source Checkpoint，epoch 5 slot 32被称作 Target Checkpoint。epoch 4 slot 0的状态为Justified，epoch 5 slot 32的状态为Pending，当投票完成后（epoch 5的最后一个slot），epoch 5 slot 0的状态变为Justified，同时epoch 4 slot 0的状态变为Finalized。
代表epoch 4之内及之前的所有区块不可逆。因此区块要达到最终确认状态需要经过两个epoch的时间。

所以并不是只有在两个Finalized Checkpoint之间的区块才是最终确认的，而是所有Finalized Checkpoint所在的Epoch内及之前的区块都是最终确认的。而此时距离交易创建到最终确认已经经过了2个Epoch的时间。


### 2025.06.17

<!-- Content_END -->
