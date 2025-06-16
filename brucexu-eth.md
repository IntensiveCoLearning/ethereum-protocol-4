---
timezone: UTC+12
---

# Bruce Xu

1. 自我介绍：E 卫兵
2. 你认为你会完成本次残酷学习吗？：会
3. 你的联系方式（推荐 Telegram）：brucexu_eth

# Notes

<!-- Content_START -->

# 2025.07.16

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

# 2025.07.17

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

<!-- Content_END -->
