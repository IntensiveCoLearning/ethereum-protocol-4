---
timezone: UTC+8
---

> 请在上边的 timezone 添加你的当地时区(UTC)，这会有助于你的打卡状态的自动化更新，如果没有添加，默认为北京时间 UTC+8 时区


# 你的名字

1. 自我介绍
   - [Rhyne Hsu](https://www.linkedin.com/in/yuanzhen-hsu) / [Sasaki](https://x.com/AntiSasaki)，一个兴趣使然的web3爱好者，希望把自己的技术锻炼得更solid！
3. 你认为你会完成本次残酷学习吗？
   - 会滴！🐛🐛🐛
4. 你的联系方式（推荐 Telegram）
   - TG：@AntiSasaki

## Notes

<!-- Content_START -->

### 2025.06.16
#### 📗Basic
- 2014年测试网上线，2015年主网上线
- $1\ eth = 10^{18}\ Wei = 10^9\ Gwei$
- 由比特币的UTXO模型转变为账户模型，可以减少空间消耗，账户以 *modified Merkle Patricia Trie (MPT)* 存储（一种能提供根hash证明的压缩前缀树）
#### 📗Architecture
> The main challenge a consensus protocol aims to solve is building a reliable distributed system on top of unreliable infrastructure.

由共识层和执行层两侧组成
- 共识层负责PoS共识
- 执行层负责实际的链上状态更新
- 共识层和执行层执行通过Engine APIs沟通

**共识层CL**，即信标链 (Beacon Chain)
- 09/15/2022 The Paris hard fork (**The Merge**)
	- 由一个规定的 *TTD (Total Terminal Difficulty)* 触发，一旦有区块的TD (Total Difficulty) 达到TTD，就会终止PoW区块的生成
	- 在PoW阶段，共识是基于最重链，即有最多的 TD 的链
	- 一般的协议更新的trigger都是block height，而这里采用TTD trigger，原因是：如果采用block height，恶意节点可以用小部分算力拓展非最重链来抢先达到规定的block height从而终止所有PoW区块的生成，实现攻击；而如果用TTD trigger则不会受到这样的攻击
	- **Bellatrix Update** 是为The Merge做准备的共识层更新，它规定了CL和EL客户端必须遵循下面的结构。CL客户端必须包含Bellatrix更新；EL客户端必须包含Paris更新并暴露 Engine API 接口
		![client-architecture](https://epf.wiki/wiki/protocol/img/clients-overview.png)
	- **Paris Update** 则是相应的执行层更新
	- [reference](https://etherworld.co/2022/07/20/what-do-bellatrix-paris-ttd-mean-in-ethereum-merge-upgrade/)
- 以太坊的共识机制 (**Gasper**) 实际上由两种共识机制耦合而成：**LMD GHOST** & **Casper FFG**
- Validators
	- 每个验证者最多质押32 ETH，更多质押并不会增加其权重。超过32 ETH的，需要运行更多验证者来实现（一个验证者客户端可以管理多个验证者）
	- 同时，要成为验证者，要求的最少质押也是 32 ETH
- Slots and Epochs
	- **12秒**一个slot，**32 slots**一个epoch，即约 6.4 分钟一个epoch
	- 这要求验证者节点在时间上要基本同步
	- 一个slot是产生区块的一次**机会**，也就是说一个slot不一定会产生一个区块
	- 网络中所有validator在每个epoch中会被均分到各个slots中，每个slot内部再划分committee
	- 每个slot会根据 RANDAO 发射的随机数，伪随机地选出该slot的committees（每个committee至少128个验证者）
	- 每个slot只有对应的committees可以进行投票见证
	- 一个slot中可以有多个commitees（一般都有多个）
	- 同时选出一个validator，由他提出新区块提案，其他validator则对其进行见证 (attest)
	- attestation会在validator之间广播，validator彼此监督是否有人
	- slot中投票后，该区块得到的只是slot级别的通过，并没有被最终确认
	- 因为在每个epoch的最后还需要全网 2/3 的总stake进行投票
- Beacon：由一个名叫 **RANDAO** 的伪随机过程发射随机数，来决定每个epoch中的每个slot的committees，这个RANDAO就像整条链的信标，这就是 *Beacon Chain* 名字的由来

**执行层EL**

#### 📋TODOs
- [ ] 整理所有历史更新的{名称、时间、主要内容}
- [ ] LMD GHOST
- [ ] Casper FFG

> END_OF_TODAY [BOOKMARK](https://epf.wiki/#/wiki/CL/overview?id=blobs)

### 2025.06.17

<!-- Content_END -->
