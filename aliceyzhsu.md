---
timezone: UTC+8
---

> 请在上边的 timezone 添加你的当地时区(UTC)，这会有助于你的打卡状态的自动化更新，如果没有添加，默认为北京时间 UTC+8 时区


# Rhyne Hsu

1. 自我介绍
   - [Rhyne Hsu](https://www.linkedin.com/in/yuanzhen-hsu) / [Sasaki](https://x.com/AntiSasaki)，一个兴趣使然的web3爱好者，希望把自己的技术锻炼得更solid！
3. 你认为你会完成本次残酷学习吗？
   - 会滴！🐛🐛🐛
4. 你的联系方式（推荐 Telegram）
   - TG：@AntiSasaki

## Notes

<!-- Content_START -->

### 2025.06.16
---
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
---
#### 📗Consensus
**Blobs**：[EIP-4844](https://eips.ethereum.org/EIPS/eip-4844) 引入，是Cancun硬分叉的一部分。允许每个区块携带3-6个 Blob sidecars，相当于为网络添加了一个数据层 (*data accessibility layer*)
- 每个Blob的保留期限是 4096 epochs，过期即会drop掉
- KZG commitment

**Finalization**：
- validator有两种投票：LMD GHOST 投票给普通区块，Casper FFG投票给checkpoints
- validator只能给自己对应的slot投LMD GHOST票，而投票checkpoints时，所有validator都可以投Casper FFG票，这个是一个epoch中随时可以投的
- 并且validator也可以延迟给slot投票，不一定非要在自己的slot立即投票
- 当然，立即投票的奖励是最高的，延迟投票会减少奖励
- checkpoint是每个epoch的第一个区块
- 每个 Casper FFG 投票，需要指定 
	1. 其源checkpoint：以往epoch的checkpoint
	2. 目标checkpoint：当前epoch的checkpoint
- checkpoint获得超 2/3 的绝对多数投票Casper FFG后，将会被 *justified*
- checkpoint被 *justified* 之后，其前一个checkpoint才会被 *finalized*
- 也就是说一个checkpoint被finalized大概需要2个epoch的时间（12.8分钟）

**Slashable Offenses**
- Double Proposal
- LMD GHOST Double Vote
- FFG Surround vote
- FFG Double Vote

**validator** 初始需要32个ETH才能激活，当其质押金额少于16ETH时会失活
- 需要服务2048个epoch之后才能主动退出
- 提出退出后，需要4个epoch的冷却期才能退出，用于留给slash的时间窗口
- 同样的，提起激活前也需要有4个epoch的窗口期
- 退出后，未被slash的在 $2^8$ epoch（27h）之后可以提款；而被slash的则需要 $2^13$ epoch （36天）才能退款
- 每个epoch能激活和失活的validator数量有一个规定的阈值
	![validator-lifecycle](https://epf.wiki/images/cl/validator-lifecycle.png)

> <END_OF_TODAY> [BOOKMARK](https://epf.wiki/#/wiki/CL/cl-architecture)

### 2025.06.18
---
#### 📗Consensus Layer
- LMD GHOST 提供 liveness，Casper FFG 提供 finality
> In essence, LMD GHOST keeps the chain moving forward, while Casper FFG ensures stability by finalizing blocks.

- **CAP Theorem**：一个分布式系统不可能同时满足 Consistency, Availability, Partition tolerance
- 一个新区块是如何产生的
	1. 当前slot的block proposer的EL client执行`create block`指令
	2. EL获取mempool中的交易
	3. EL打包交易并执行，生成新区块的hash
	4. CL客户端将交易集合和区块hash放入beacon block（CL的区块），并在网络中广播
	5. 其他validator收到广播区块
	6. 将交易作为 *execution payload* 发送到EL，执行交易并生成hash
	7. 验证hash后，将该区块添加到CL中，完成见证（attest）并广播 *attestation*
- 一个新区块的心路历程：proposed -> attested -> justified -> finalized
- Consensus Client的技术栈
	- libp2p：p2p协议
	- discv5：peer discovery
	- libp2p-noise：加密
	- [SSZ](https://ethereum.org/en/developers/docs/data-structures-and-encoding/ssz/) (Simple SerialiZe)：取代了 RLP的新的编码/序列化算法
	> [RLP](https://ethereum.org/en/developers/docs/data-structures-and-encoding/rlp/) (Recursive-Length Prefix)：曾是以太坊EL的序列化算法

	- (snappy)[https://en.wikipedia.org/wiki/Snappy_(compression)]：压缩算法，保证合理的压缩比的前提下优先压缩/解压速度
- 共识层规范 Consensus Layer *Specification*：[Pyspec](https://github.com/ethereum/consensus-specs) 是为共识层开发者提供的执行规范
- Clients
	- Consensus Client: 运行以太坊PoS共识算法
	- Execution Client: 参与交易的验证和广播，执行状态转移
	- Validator Client: 进行 attest 和 propose 新区块（validator client时 consensus client的一种可选的附加组件）

> <END_OF_TODAY> [BOOKMARK](https://epf.wiki/#/wiki/CL/cl-clients)

### 2025.06.19
---
目前的CL Client种类

|Client|Language|Developer|Status|
|---|---|---|---|
|[Lighthouse](https://github.com/sigp/lighthouse)|Rust|Sigma Prime|Production|
|[Lodestar](https://github.com/ChainSafe/lodestar)|TypeScript|ChainSafe|Production|
|[Nimbus](https://github.com/status-im/nimbus-eth2)|Nim|Status|Production|
|[Prysm](https://github.com/prysmaticlabs/prysm)|Go|Prysmatic Labs|Production|
|[Teku](https://github.com/ConsenSys/teku)|Java|ConsenSys|Production|
|[Grandine](https://github.com/grandinetech/grandine)|Rust|Grandine Developers|Production|
|[Caplin](https://github.com/ledgerwatch/erigon)|Go|Erigon|Development|
|[LambdaClass](https://github.com/lambdaclass/lambda_ethereum_consensus)|Elixir|LambdaClass|Development|

客户端种类的**多样性**对一个网络的安全和鲁棒至关重要，因为单一客户端有出现问题的可能，如果网络中2/3的节点使用单一客户端，一旦该客户端软件出问题，网络就会崩溃。目前最主流的是 Lighthouse 和 Prysm

#### 📗SSZ
SSZ (Simple SerialiZation): 是专为以太坊设计的 1. 序列化 和 2. Merkleization 方案

- 为什么要序列化：将数据结构或对象转换为**可存储或可传输格式**，以便 
	1. 数据持久化 
	2. 网络传输
	3. 跨语言/平台兼容：为跨实现的节点（不同语言）提供统一通信格式
	4. 存储与效率优化：原始对象可能包含很多冗余信息，序列化可以压缩为精简格式，节省空间
	5. 哈希和签名：数据序列化为字节后，便于进行哈希、加密、签名等操作
- 无符号整数编码：Little-Endian Formt，低位byte排在地址前面（最低位），如`0x12345678` 编码为 `[0x78, 0x56, 0x34, 0x12]`
```python
class Example
    id: uint64,
    bytes: List[uint8, 64]
    next: uint64
```
```
# serialize(my_example)
#
# Note: this is a single byte-array split over four lines for readability.
[
  42, 0, 0, 0,  # The little-endian encoding of `id`, 42.
  12, 0, 0, 0,  # The "offset" that indicates where the variable-length value of `bytes` starts (little-endian 12).
  43, 0, 0, 0,  # The little-endian encoding of `next`, 43.
  0, 1, 2       # The value of the `bytes` field.
]
```

> <END_OF_TODAY> [BOOKMARK](https://epf.wiki/#/wiki/CL/SSZ)

### 2025.06.20
---
- SSZ具体各种类型的数据的序列化方法： https://epf.wiki/#/wiki/CL/SSZ
- Merkleization: 指将序列化后的数据构造为Merkle树并得到其根hash的过程
- 弱主观性 *weak subjectivity*
	- 客观：信息的正确性完全可验证
	- 主观：信息的正确性并不能完全可验证
- *Full Syncing*：指PoW时期，所有合法区块都一定可以完美追溯到Genesis块
	- 然后转入PoS之后，由于finality来自于CL而非EL
	- 要断言一个finalized的checkpoint的正确性，必须要额外的验证
- validator退出CL之后，将不能再为未来的区块投票，但仍可以对过去的区块重新投票
- Beacon block syncing 和 Execution block Syncing 的区别
	1. Beacon chain的syncing是反方向的
	2. Beacon chain的信任锚点是主观的：Beacon chain的同步目标是一个“最终确定”区块，但它不能被完全客观验证，只能部分验证其正确性。因此，这个检查点需要通过**链外途径（out-of-band）**获取
	3. 弱主观性周期与乐观导入：在 Execution 层尚未完成同步时，Beacon 链会先 **乐观导入（optimistically import）** 新块，但暂不验证其 Execution Payload
- **Weak Subjectivity Sync** 步骤
	1. Obtain a weak subjectivity checkpoint out-of-band （因为p2p层中的peer是不可信的）
	2. Backfill blocks all the way back to genesis from the weak subjectivity checkpoint
	3. Update the target header for the execution chain
	4. Optimistically follow the head of the chain and continuously update the target header for the execution chain
	5. Once the EL is synced, then mark the CL slots as verified post-verification. The node may now be considered fully synced

> <END_OF_TODAY>

<!-- Content_END -->
