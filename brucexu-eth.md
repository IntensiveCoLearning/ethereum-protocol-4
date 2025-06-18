---
timezone: UTC+12
---

# Bruce Xu

1. 自我介绍：E 卫兵
2. 你认为你会完成本次残酷学习吗？：会
3. 你的联系方式（推荐 Telegram）：brucexu_eth

# Notes

<!-- Content_START -->

# 2025.06.16

我的学习计划大概是：

1. 扫一遍 VB 文章，整理一些问题和思路
2. 完成 geth 源码阅读文章，本地配置好环境，适应阅读代码
3. 围绕一个 https://forum.ethpanda.org/ 协议问题，从 EIP 标准，到 geth 实现都 debug 一遍，熟悉整个流程是怎么运转的

## https://vitalik.eth.limo/general/2024/10/14/futures1.html

The Merge: key goals

- Single slot finality
- Transaction confirmation and finalization as fast as possible, while preserving decentralization
- Improve staking viability for solo stakers
- Improve robustness
- Improve Ethereum's ability to resist and recover from 51% attacks (including finality reversion, finality blocking, and censorship)

### Single slot finality

Two params:

- 2-3 epochs (~15min) to finalize a block
  - 1 epochs = 32 slots, 1 slot = 12s
  - a committee of 128 validators are selected to participate in the finality process
- 32 ETH for staking

Balance of three goals:

- Maximizing the number of validators
- Minimizing the time to finality
- Minimizing the overhead of running a node

Status quo:

- every single validator has to sign two messages each time finality happens
  - TODO what is the process of finality?
  - long time to process all signatures or beefy nodes to process all at the same time

Solutions:

- randomly selecting a committee to finalize each slot
  - easier to perform an attack, and not working well
- consensus algorithm to finalize blocks in one slot
  - e.g. [Tendermint](https://docs.tendermint.com/v0.34/introduction/what-is-tendermint.html) TODO how?
  - But does not support TODO [inactivity leak](https://eth2book.info/capella/part2/incentives/inactivity/)

Goals:

- Finalize blocks in one slot 12s
- Stake with 1 ETH, support solo stakers

Paths:

- how to make ssf work with very high validator count?
  - Brute force: better signatures aggregation (ZK-SNARKs?) to process all signatures in one slot
    - high tech, aggregating 1M+ signatures in 5-10s
    - TODO what is the current speed of aggregating signatures?
  - [Orbit committees](https://ethresear.ch/t/orbit-ssf-solo-staking-friendly-validator-set-management-for-ssf/19928/1): new mechanism to randomly-select a committee but in a safe way. TODO how?
    - relax the economic finality requirement
  - two-tiered staking: two classes of stakers, higher deposit and lower deposit.
    - centralization risks, depends on the rights that lower staking tier gets

Multiple strategies can be combined.

TODO [inclusion lists](https://ethereum-magicians.org/t/eip-7547-inclusion-lists/17474)

TODO BLS aggregation

TODO MEV attacks

What are some links to existing research for SSF?

- Paths toward single slot finality (2022): https://notes.ethereum.org/@vbuterin/single_slot_finality
- A concrete proposal for a single slot finality protocol for Ethereum (2023): https://eprint.iacr.org/2023/280
- Orbit SSF: https://ethresear.ch/t/orbit-ssf-solo-staking-friendly-validator-set-management-for-ssf/19928
- Further analysis on Orbit-style mechanisms: https://ethresear.ch/t/vorbit-ssf-with-circular-and-spiral-finality-validator-selection-and-distribution/20464
- Horn, signature aggregation protocol (2022): https://ethresear.ch/t/horn-collecting-signatures-for-faster-finality/14219
- Signature merging for large-scale consensus (2023): https://ethresear.ch/t/signature-merging-for-large-scale-consensus/17386?u=asn
- Signature aggregation protocol proposed by Khovratovich et al: https://hackmd.io/@7dpNYqjKQGeYC7wMlPxHtQ/BykM3ggu0#/
- STARK-based signature aggregation (2022): https://hackmd.io/@vbuterin/stark_aggregation
- Rainbow staking: https://ethresear.ch/t/unbundling-staking-towards-rainbow-staking/18683

### Single secret leader election

which validator is going to propose the next block is known ahead of time, therefore, leading potential DoS attacks.

- 解决 DoS 是主要目的，通过 SSLE 顺便实现了
  - 提高网络去中心化，降低大型质押池对小型验证者的攻击
  - 防止 MEV 攻击等等，主要是保护 proposer，防止一直出不了 slot

# 2025.06.17

two solutions:

- hide the information about the validator
  - let anyone create the next block, use randao to determine
    - randao is for generating random number TODO how?
  - SSLE create a "blinded" validator ID for each validator and then giving many proposers the opportunity to shuffle-and-reblind the pool of blinded IDs (this is similar to how a mixnet works).
    - Only validator know their own blinded ID, no one else knows which validator is going to propose the next block
    - only one proposer in each slot, random selection

What's next:

- TODO how to make Ethereum bing a reasonably simple protocol?
- Need an efficient-enough quantum-resistant SSLE implementation
- use ou-of-protocol to solve the DoS issues (SSLE only used for solve DoS issue?)
- TODO general-purpose zero-knowledge proofs into the Ethereum protocol at L1 for other reasons (eg. state trees, ZK-EVM). 有哪些可以结合的地方？

## Attestation 工作流程详解

### Attesters 基本概念

- **Attesters（证明者）**: 参与区块确认过程的验证者，通过投票证明区块有效性
- 每个 slot 只有 1 个 proposer，但有很多 attesters（通常 128 个组成委员会）
- Attesters 是普通 validators，在特定 slot 被分配证明职责

### 委员会分配机制

- 每个 epoch（32 个 slot）开始时，所有活跃验证者被随机分配到不同委员会
- 每个 slot 都有一个委员会负责对该 slot 的区块进行证明
- 委员会大小通常为 128 个验证者（根据总验证者数量可调整）
- 基于 RANDAO 生成的随机数进行分配

### Attestation 具体流程

#### 1. 区块验证阶段

- 验证区块的有效性（交易、状态转换等）
- 检查区块是否符合协议规则
- 验证区块的父区块关系

#### 2. 发送证明消息

- 如果区块有效，attester 发送 attestation（证明消息）
- 证明消息包含：
  - 对区块头的签名
  - 验证者的身份信息
  - 对链头的投票

#### 3. 签名聚合

- 同一委员会的多个 attestations 被聚合成一个聚合签名
- 使用 BLS 签名聚合技术，将多个签名合并为一个
- 大大减少网络传输和存储开销

### Finality 实现机制

#### 两轮投票系统（类似 PBFT）

**第一轮：Justification（合理化）**

- Attesters 对当前 epoch 的区块进行投票
- 如果一个 epoch 获得超过 2/3 的投票权重，该 epoch 被标记为 "justified"

**第二轮：Finalization（最终化）**

- 当下一个 epoch 也被 justified 时
- 前一个 justified epoch 就被 "finalized"
- 被 finalized 的区块不可逆转

#### 时间线示例

```
Epoch N:   [Slot 0] [Slot 1] ... [Slot 31]
           ↓ Attesters 投票
           Justified (如果 >2/3 投票权重)

Epoch N+1: [Slot 32] [Slot 33] ... [Slot 63]
           ↓ Attesters 投票
           Justified (如果 >2/3 投票权重)
           ↓
           Epoch N 被 Finalized（约12-15分钟后）
```

### 关键安全机制

#### 1. 投票权重计算

- 每个验证者的投票权重 = 其质押的 ETH 数量
- 需要超过 2/3 的**质押权重**（不是验证者数量）同意
- 这确保了经济安全性

#### 2. Casper FFG 协议

- 以太坊使用 Casper FFG (Friendly Finality Gadget)
- 在任何区块链之上都能工作的最终性协议
- 提供数学可证明的最终性保证

#### 3. 惩罚机制

- **Slashing**: 矛盾投票会被大幅惩罚（失去大部分质押）
- **Inactivity Leak**: 长期不参与投票的验证者逐渐失去质押

### 拜占庭容错保证

- 只要诚实验证者控制 >2/3 质押，网络安全
- 最多容忍 1/3 恶意验证者
- 攻击者需要控制 >1/3 总质押才能破坏最终性（需要巨额资金且会自损）

### 实际运行示例

```
时间: Slot 100 (Epoch 3, Slot 4)
1. Proposer 提议区块 100
2. 委员会中128个 attesters 验证区块
3. 假设120个 attesters 认为区块有效
4. 120个签名聚合成一个聚合签名
5. 聚合签名包含在后续区块中
6. 累积足够 attestations 后实现最终性
```

### 与 Proposer 的配合

- **Proposer**: 创建区块（每 slot 1 个，随机选择）
- **Attesters**: 验证并投票确认区块（每 slot 多个，委员会）
- **频率**: 验证者平均每 32 个 slot 当 1 次 attester，几个月才当 1 次 proposer
- **单点风险**: Proposer 离线只影响该 slot，Attesters 分布式投票更可靠

这套机制实现了：

- **安全性**: 数学可证明的最终性
- **活跃性**: 大多数验证者在线就能运行
- **效率**: 签名聚合减少网络开销

# 2025.06.18

Bruce Xu, [18/6/2025 08:38]
我突然想到，未来的以太坊一种节点方式是在客户端，比如 dapp 里面。当用户打开和使用 dapp 的时候，就在运行一个节点，比如浏览器网页。目前有轻节点实现类似的效果 https://eth-light.xyz/ 但是出块什么的不知道行不行。
理论上似乎是可行的，本质上就是一个可以联通网络的可运行代码？不过可能存储层面会有一些问题
如果硬件性能足够强，客户端足够轻量，其实是可以实现的？这样可以实现用户边用以太坊边维持以太坊网络运行，自己的安全也控制在自己手里。完事了越多用户越安全

### Faster transaction confirmations

A great improvement if transaction confirmation from 12s to 4s.

还可以改善 L2 的去中心化，让大家运行在 based rollups 上面。

实现方案：

1. Reduce slot times to 8s or 4s

- finality 采用 3 轮验证，每一轮作为一个独立 block 的话，12s 就可以完成最终性确认
  - 但是网络延迟要求更加严格，增加了共识协议和状态的复杂度
  - 可以按照阶段进行体验优化，比如 4s 的时候是已经提交，4s 是正在处理大概率可以，4s 是已经完成，不可撤销

2. Pre-confirmations

- proposer 在 slot 开始之后，接收到了 tx 就不停地发布 pre-confirmation message 的消息向外界同步我即将打包的 tx 信息，方便快速确认是否被包含
- 如果宣称包含，但是没有实际包含，proposer 需要被惩罚，或者通过某种机制进行确认

问题：

- Proposer 掉线了或者整个 slot 错过了，可能会导致 tx 被延迟？回答：确实是个问题，还是会遇到这个最坏情况
- 在这里面 MEV 是怎么实施的？如果最后来了一个比较高价值的 tx，proposer 可以取消 pre-confirmation 吗？
  - 为 pre-confirmation 的 tx 额外增加保障金？和交易费？这是一个开放式需要解决的问题
- 这个发送的消息在网络中广播，还是会有延迟或者丢失，是否会引起
  网络拥堵？
  - 似乎是 proposer 直接发送，而且是单向广播，不需要交互？似乎还好。那么 proposer 的网络连通性是不是会有问题？如果它的网络不稳定没有发出去？

目前遇到的问题：

- 降低 slot time 主要是节点的网络要求会变高，导致 validator 中心化

我个人感觉 pre-confirmation 的方案是更加可靠的，同时需要配套的 dapp SDK 来实现监听 pre-confirmation 消息并且做出响应和 UX 优化。

### Increasing the quorum threshold

目前是 67% stakers 支持就可以确认 block。增加到 80% 可能会增加安全性，可以在极端情况下暂时停止最终性确认。比如恶意攻击或者客户端出 bug。

这样让 solo stakers 的价值更大，比如需要 80% 的支持才可以的话，有 21% 的反对就可以让这个出块暂停。然后 solo stakers 不像是 staking pool 一样被容易攻击和控制，所以他们就可以发挥力量停掉。21% 的 solo stakers 是一个可以尝试实现的目标。

核心理念就是：停止比出错更好。

### Quantum-resistance

量子计算大概在 2030s 可以击溃目前的密码学。目前 Ethereum 使用的 elliptic curves 等核心加密学算法需要替换成抗量子攻击的算法。

TODO Luban https://docs.luban.wtf/learn/fundamentals/what

TODO Based rollups—superpowers from L1 sequencing https://ethresear.ch/t/based-rollups-superpowers-from-l1-sequencing/15016

TODO Based preconfirmations https://ethresear.ch/t/based-preconfirmations/17353

TODO APS https://www.ephema.io/blog/beyond-the-stars-an-introduction-to-execution-tickets-on-ethereum

<!-- Content_END -->
