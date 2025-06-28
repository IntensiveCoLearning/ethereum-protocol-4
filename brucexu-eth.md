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
  > **AI 补充**: 128 是协议设定的**最小**委员会规模 (`TARGET_COMMITTEE_SIZE`)。实际的委员会规模是根据当前活跃验证者的总数动态计算的 (`活跃验证者总数 / 32`)。例如，在有 960,000 名活跃验证者时，每个 slot 的委员会会有 `960,000 / 32 = 30,000` 名验证者。这个庞大的委员会内部还会再被分成多个子网 (subnet) 来处理证明。所以，目前实际的委员会规模远大于 128。
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

TODO 整理以太坊面临的量子算法危机和解决方法

# 2025.06.19

## https://vitalik.eth.limo/general/2024/10/17/futures2.html

最开始提出了两种扩容方案：

- 分片
  - 每个 node 就负责一小部分，而不是全部的 txs
  - peerDAS 完成 full rollup scaling
- L2
  - 将大部分数据和计算放在别的地方，但是关键信息存储在 L1 上面
  - 从 state channels 到 plasma 再到 rollups 有不同的方案演进
  - 20 年提出 [A rollup-centric ethereum roadmap](https://ethereum-magicians.org/t/a-rollup-centric-ethereum-roadmap/4698)
  - L2 interop
  - blob 扩容
  - 先是 op rollup 然后是 zk rollup

这套战略在 2025 年反思的话：

1. 分片进展太慢，不过正在加速，预计今年上线
2. L2 生态崛起，但是比较分散，没有形成合力，interop 也很难实现，不是一个单纯的技术问题，导致 Solana 这样的竞争对手对 ETH Mainnet 造成沉重打击。然后 L2 的收益也没有很好的回流 ETH，模块化之后，还非常容易更换 DA 到其他项目。L2 的募资也占用了大量的资金和资源，结果被掏空或者创始团队拿走了，然后就是一堆同质化的东西。

The Surge 的目标：

- 10w TPS on L1 L2
- 有一些 L2s 有 ETH 的核心属性
- L2 interoperability，ETH 感觉像是一个 ecosystem

不可能三角：

- 去中心化：要求有很多 node，运行成本低
- 安全：要求有足够高的攻击成本
- 可扩展性：要求有足够多的 txs

解决方案需要跳出对这个三个属性的拉扯和取舍，而是从更高层面或者外部进行创新性的方案设计：

- DAS + SNARK

  - SNARK 轻量的就可以运算验证数据正确，降低 node 运行难度
  - DAS 每个客户端下载和保存一部分数据，降低 node size
    - [few-of-N trust model](https://vitalik.eth.limo/general/2020/08/20/trust.html) 保证了核心安全属性
  - 这样实现 node 轻量化，安全性足够，扩展性高

- Plasma

# 2025.06.20

中期目标是 16MB 一个 slot，结合 rollup data 压缩可以达到 58000 TPS。

PeerDAS is a relatively simple implementation of "1D sampling". Each blob in Ethereum is a degree-4096 polynomial over a 253-bit prime field. We broadcast "shares" of the polynomial, where each share consists of 16 evaluations at an adjacent 16 coordinates taken from a total set of 8192 coordinates. Any 4096 of the 8192 evaluations (with current proposed parameters: any 64 of the 128 possible samples) can recover the blob.

## PeerDAS 数据可用性采样详解

### 1. 基础数学概念

#### 1.1 素数 (Prime Number)

- **定义**: 只能被 1 和自身整除的大于 1 的自然数
- **例子**: 2, 3, 5, 7, 11, 13, 17, 19, 23, 29...
- **非素数**: 4(=2×2), 6(=2×3), 8(=2×4), 9(=3×3)...

#### 1.2 素数域 (Prime Field)

- **定义**: 包含 p 个元素的有限域，其中 p 是素数，但是前面的不完全是
- **表示**: F_p = {0, 1, 2, ..., p-1}
- **运算规则**: 所有运算都在模 p 下进行

**例子 - F₇ 域:**

```
F₇ = {0, 1, 2, 3, 4, 5, 6}
加法: 3 + 5 = 8 ≡ 1 (mod 7)
乘法: 4 × 6 = 24 ≡ 3 (mod 7)
```

#### 1.3 乘法逆元 (Multiplicative Inverse)

- **定义**: 在 F_p 中，a 的逆元 a⁻¹ 满足 a × a⁻¹ ≡ 1 (mod p)
- **用途**: 实现除法运算 a ÷ b = a × b⁻¹

**F₇ 中的逆元表:**

```
数字 | 1 | 2 | 3 | 4 | 5 | 6
逆元 | 1 | 4 | 5 | 2 | 3 | 6
```

**除法例子:**

```
6 ÷ 2 = 6 × 2⁻¹ = 6 × 4 = 24 ≡ 3 (mod 7)
```

#### 1.4 多项式 (Polynomial)

- **一般形式**: f(x) = aₙxⁿ + aₙ₋₁xⁿ⁻¹ + ... + a₁x + a₀
- **系数**: a₀, a₁, ..., aₙ (编码原始数据)
- **次数**: n (最高幂次)

**例子:**

```
f(x) = x² + 3x + 5
系数: [5, 3, 1] (常数项在前)
次数: 2
```

#### 1.5 多项式的关键性质

- **唯一性**: n 次多项式由 n+1 个点唯一确定
- **冗余性**: 给定超过 n+1 个点，可以检测和纠正错误

**例子:**

```
2次多项式需要3个点确定:
已知: (0,5), (1,9), (2,17)
可求出: f(x) = x² + 3x + 5
```

### 2. Reed-Solomon 编码原理

#### 2.1 基本思想

- **目标**: 为数据添加冗余，实现纠错能力
- **方法**: 将数据编码为多项式系数，在多个点评估

#### 2.2 编码过程

```
Step 1: 原始数据 → 多项式系数
原始: [5, 3, 1]
多项式: f(x) = 1x² + 3x + 5

Step 2: 选择评估点 (连续整数)
点: x = 0, 1, 2, 3, 4, 5

Step 3: 计算多项式值
f(0) = 5
f(1) = 1 + 3 + 5 = 9
f(2) = 4 + 6 + 5 = 15
f(3) = 9 + 9 + 5 = 23
f(4) = 16 + 12 + 5 = 33
f(5) = 25 + 15 + 5 = 45

编码结果: [5, 9, 15, 23, 33, 45]
```

#### 2.3 纠错能力

```
原始数据: 3个符号
编码数据: 6个符号 (2倍冗余)
纠错能力: 可恢复任意3个损坏的符号
恢复条件: 至少需要3个正确的点
```

#### 2.4 恢复过程

**例子: 从部分数据恢复原始多项式**

```
已知: f(1)=9, f(2)=15, f(4)=33
目标: 恢复 f(x) = ax² + bx + c

建立方程组:
f(1) = a + b + c = 9
f(2) = 4a + 2b + c = 15
f(4) = 16a + 4b + c = 33

解方程:
方程2-方程1: 3a + b = 6
方程3-方程2: 12a + 2b = 18 → 6a + b = 9
相减: 3a = 3 → a = 1
代入: b = 3, c = 5

恢复结果: f(x) = x² + 3x + 5
原始数据: [5, 3, 1] ✅
```

### 3. PeerDAS 技术实现

#### 3.1 基本参数

```
多项式次数: 4096
素数域大小: 253位素数 (约 2^253)
评估点数: 8192个 (2倍冗余)
评估点: x = 0, 1, 2, ..., 8191
```

> **AI 补充**: 这是一个常见的混淆点。一个 Blob 由 **4096 个**数据块 (field elements) 组成，这些数据块作为系数，定义了一个**最高 4095 次**的多项式 (`P(x) = c₀ + c₁x + ... + c₄₀₉₅x⁴⁰⁹⁵`)。就像 3 个点可以唯一确定一个 2 次多项式（抛物线）一样，`n+1` 个系数可以定义一个 `n` 次多项式。

#### 3.2 Blob 数据处理

```
原始 Blob:
- 大小: 128KB (131,072 字节)
- 内容: Layer 2 交易数据、状态更新等
- 格式: 二进制数据

转换为系数:
Step 1: 二进制 → 数字序列
Step 2: 分组/填充到 4097 个系数
Step 3: 构造 4096次多项式
```

#### 3.3 Share 机制

```
总评估点: 8192个
每个 Share: 16个相邻坐标的评估值
Share 总数: 8192 ÷ 16 = 512个

Share 分布:
Share 1: [f(0), f(1), ..., f(15)]
Share 2: [f(16), f(17), ..., f(31)]
...
Share 512: [f(8176), f(8177), ..., f(8191)]
```

#### 3.4 采样参数

```
当前提议参数:
- 总 shares: 128个 (从512个中选择)
- 恢复所需: 64个 shares (任意64个)
- 冗余度: 2倍 (128/64 = 2)

采样效率:
- 下载比例: 64/512 ≈ 12.5%
- 验证能力: 完整数据可用性确认
```

> **AI 补充**: 这里可能混淆了“数据可用性采样 (sampling)”和“数据恢复 (recovery)”两个概念。
>
> - **采样 (Sampling)**: 轻节点（或验证者）随机下载少量 (例如 64 个) shares 的目的是为了**验证**数据是可用的，它以极高的概率确信，如果这些随机样本存在，那么完整数据也一定存在。单个节点**不会**用这少量样本去恢复完整数据。
> - **恢复 (Recovery)**: 要完整地恢复一个 Blob，需要下载足够多的 shares 以重建那个 4095 次的多项式。具体来说，需要获得 **4096 个**不同的评估点。由于每个 share 包含 16 个评估点，所以理论上至少需要 `4096 / 16 = 256` 个完整的 shares 才能进行数据恢复。

### 4. 数据可用性采样工作流程

#### 4.1 数据发布流程

```
1. Layer 2 提交 Blob 数据 (128KB)
2. 转换为 4097 个多项式系数
3. 构造 4096次多项式 f(x)
4. 在素数域中计算 8192 个评估值
5. 分组为 512 个 shares
6. 广播 shares 到网络节点
```

#### 4.2 数据验证流程

```
1. 验证节点随机选择部分 shares
2. 下载选中的 shares (如64个)
3. 尝试恢复原始多项式
4. 成功恢复 → 数据可用 ✅
5. 失败 → 数据不可用 ❌
```

#### 4.3 优势分析

```
效率提升:
- 传统: 下载完整 128KB
- PeerDAS: 只需 12.5% 的数据

安全保证:
- 数学保证: Reed-Solomon 编码
- 分布式验证: 多节点独立确认
- 拜占庭容错: 即使部分节点恶意也能正确验证

可扩展性:
- 支持大规模数据处理
- 降低节点存储和带宽要求
- 保持网络去中心化特性
```

### 5. 关键技术要点

#### 5.1 存储权衡

```
问题: 编码后存储增加
8192个大数值 vs 原始128KB

解决方案:
- 分布式存储: 每个节点只存储部分shares
- 网络总存储增加，但单节点负担可控
- 功能收益 > 存储成本
```

#### 5.2 计算复杂度

```
多项式评估: O(n²) 朴素算法
优化方法: FFT (快速傅里叶变换)
实际复杂度: O(n log n)
```

#### 5.3 安全性分析

```
攻击场景: 恶意节点提供错误shares
防护机制:
- 多节点验证
- 数学验证 (多项式一致性)
- 经济激励 (诚实节点奖励)
```

### 6. 与以太坊扩容的关系

#### 6.1 在 "The Surge" 中的作用

- **目标**: 提高以太坊数据吞吐量
- **方法**: 通过 PeerDAS 实现高效数据可用性
- **效果**: 支持更多 Layer 2 rollups

#### 6.2 未来发展

```
当前: 1D 采样 (PeerDAS)
未来: 2D 采样 (更高效率)
最终: 完全分片的数据可用性层
```

# 2025.06.23

PeerDAS works by having each client listen on a small number of subnets, where the i'th subnet broadcasts the i'th sample of any blob, and additionally asks for blobs on other subnets that it needs by asking its peers in the global p2p network (who would be listening to different subnets).

so ultimately we want to go further, and do 2D sampling, which works by random sampling not just within blobs, but also between blobs.

TODO https://www.paradigm.xyz/2022/08/das

Data availability 对区块链非常重要。我们需要一种办法可以下载部分数据即可验证数据可用，因为数据量太大。

TODO PeerDAS on ethresear.ch: https://ethresear.ch/t/peerdas-a-simpler-das-approach-using-battle-tested-p2p-components/16541 , and paper: https://eprint.iacr.org/2024/1362

TODO KZG commitments 的工作原理？

we want more academic work on formalizing PeerDAS and other versions of DAS and its interactions with issues such as fork choice rule safety.

Further into the future, we need much more work figuring out the ideal version of 2D DAS and proving its safety properties. We also want to eventually migrate away from KZG to a quantum-resistant, trusted-setup-free alternative.

### BLS 与 ECDSA 的简明对比

- **数学环境**
  - ECDSA: 普通椭圆曲线（secp256k1）
  - BLS: 配对友好曲线（BLS12-381），支持双线性映射
- **随机数需求**
  - ECDSA: 每次签名必须生成随机 nonce `k`，泄露或复用会暴露私钥
  - BLS: 无需随机数，签名公式只有 `sk · H(m)`
- **签名大小**
  - ECDSA: 64 字节 `(r, s)`
  - BLS: 48 字节（压缩曲线点），并且可聚合成 1 个签名
- **聚合能力**
  - ECDSA: 需要交互式多签协议（MuSig2 等），链上数据仍是 64 字节
  - BLS: 天生非交互聚合，多人签名可直接相加，链上只存 48 字节
- **验证成本**
  - ECDSA: 快，一次曲线运算
  - BLS: 一次配对运算，单签慢约 5-10×，但聚合后摊薄
- **典型使用场景**
  - ECDSA: 以太坊执行层账户签名、比特币
  - BLS: 以太坊共识层 attestation 聚合、阈值签名、Filecoin

> 小结：BLS 通过"无随机数 + 配对 + 聚合"把大量签名压缩到 1 条，非常适合成千上万验证者一起上链；而 ECDSA 实现简单、单签更快，在账户签名等场景仍然主流。

## Data compression

几个可以将 blockchain 数据压缩的思路和方法：

- 签名聚合，从 ECDSA 到 BLS，这样可以合并多个签名并且确认其他签名是真实的
- 使用 pointer 指向 addresses，目前 address 是 20-byte，如果只有 history 存储过，可以使用 4-byte 的 pointer 指向这个地址
- 自定义交易 values 的单位，比如 1000000000000000000 可以表示为 1 ETH，这样就可以节省空间

比较大的挑战：

- 切换 BLS 的开发成本很大，而且还要考虑 hardware chips 的安全性
- 动态压缩数据让客户端实现更复杂
- 可能会让一些外部软件比如 explorers 失效或者需要重新开发

# 2025.06.24

## Generalized Plasma

16MB 的 blobs，58000 TPS 也不够 consumer payments、decentralized social 等。由于 privacy 的加入，还会缩小 3-8x。

Plasma 是一种扩容方案，就是链下发布 block 和 tx，然后把交易的 Merkle roots 更新到链上。

Early versions of Plasma were only able to handle the payments use case, and were not able to effectively generalize further.

One way (not the only way) to make an EVM plasma chain: use a ZK-SNARK to construct a parallel UTXO tree that reflects the balance changes made by the EVM, and defines a unique mapping of what is "the same coin" at different points in history. A Plasma construction can then be built on top of that.

### Validium vs Rollup

在 L1 上面存储少量的信息（例如 ZK-SNARK 证明），链下的数据如果以 calldata 或者 blob 的方式上 L1 或者 DA，让所有人可以参与验证，就是 Rollup，如果完全自己处理，就是 Validium。后者的话，如果运营商跑路，那么数据就会丢失无法退出。

隐私性方面，Validium 比较容易实现，如果必须要公开，Aztec 使用加密后的数据发到 L1，然后通过密钥可以解开。

TPS 的计算，单纯从存储的数据量来计算不对吧，还要考虑网络和计算成本？

any validium can have its safety properties improved at least a little bit by adding Plasma features into the exit mechanism.

# 2025.06.25

## L2 proof systems

目前的 L2 还是依赖 security council 等机制，不是非常的安全和去中心化。但是担心 code 有 bug，所以不敢放的泰开。

目前 Layer 2 有几个 stages 的定义；

- Stage 0：用户可以自己 run node 和同步数据，validation 可以是 centralized 和 trusted，比较中心化
- Stage 1：添加 trustless proof system，允许一个 75% threshold vote 的 security council 可以控制，其中 26%+ 投票者必须不在这个团队
- Stage 2：trustless proof system，security council 只能在代码出 bug 的情况下介入。支持升级，但是需要一个 long 时间锁。

The main challenge in reaching stage 2 is getting enough confidence that the proof system actually is trustworthy enough.

- Formal verification: we can use modern mathematical and computational techniques to prove that an (optimistic or validity) proof system only accept blocks that pass the EVM specification.
- Multi-provers: make multiple proof systems, and put funds into a 2-of-3 (or larger) multisig between those proof systems and a security council (and/or other gadget with trust assumptions, eg. TEEs).

For formal verification, we need to create a formally verified version of an entire SNARK prover of an EVM.

形式化验证 (Formal Verification) 是一种基于数学和逻辑的方法，用来证明一个系统（如一个软件程
序、一个硬件芯片或一个密码学协议）的正确性是否符合其预先设定的规约 (Specification)。

换句话说，它不是通过运行测试用例来“寻找 bug”，而是通过严格的数学推导来“证明 bug
不存在”（在给定的规约下）。

与传统测试 (Testing) 的核心区别

为了更好地理解，我们可以用一个比喻：

- 传统测试 (Testing): 就像是造好一座桥后，让几百辆不同重量的卡车开过去，看看桥会不会塌。如果卡车都安全通过了，你会对桥的质量有信心，但你无法保证第一千辆、或者一辆设计奇特的卡车开过去时，桥一定不会塌。测试只能发现 bug，但不能证明没有 bug。
- 形式化验证 (Formal Verification): 就像是在设计阶段，工程师运用物理学和材料力学的所有定律，通过数学计算和逻辑推导，证明出这座桥的设计在理论上可以承受所有低于设计载重的车辆，无论它们是什么形状或重量。它提供的是一种数学上的保证。

形式化验证在航空航天、芯片设计等高风险领域早已是标配，而它对于区块链来说同样至关重要，原因如下：

1.  不可变性 (Immutability): 智能合约一旦部署到链上，就几乎无法修改。如果事后发现一个严重 bug，你不能像传统软件那样发布一个补丁了事，唯一的办法可能是迁移到新合约，这会带来巨大的成本和风险。
2.  高价值 (High Stakes): 智能合约掌管着价值数十亿甚至上百亿美元的数字资产。一个微小的 bug，比如 The DAO 事件中的重入攻击漏洞，就可能导致灾难性的财务损失。
3.  自治性 (Autonomous Nature): “代码即法律 (Code is Law)”。智能合约会根据预设逻辑自动执行，没有管理员可以中途干预。因此，代码逻辑的绝对正确性是信任的基础。

目前主要局限就是：

- 成本高：验证过程耗时、需要专业数学和逻辑知识
- 规约是关键: 它只能保证代码符合你写的规约。如果你在规约阶段就遗漏了某个重要的安全属性，那么即使验证通过，系统依然可能存在漏洞。它回答的是“我们是否正确地构建了系统？”，而不是“我们是否构建了正确的系统？”。

# 2025.06.26

## Cross-L2 interoperability improvements

L2 太多，导致用户很难选择或者切换。比较简单的方式是 re-introduce trust assumptions，就是做一些跨链桥、RPC 等。但是如果我们认为 L2 是 Ethereum 的一部分，我们就需要合并成为一个 unified Ethereum system。

几个解决方案：

- Chain-specific addresses：the chain (L1, Optimism, Arbitrum...) should be part of the address. 帮忙校验，避免打错。
- Chain-specific payment requests：it should be easy and standardized to make a message of the form "send me X tokens of type Y on chain Z". 某个链 dapp 需要当前钱包支付。
  - 建立一个标准化的消息格式（例如，通过一个新的 EIP），允许 dApp 创建一个“支付请求”，其中包含所有必要信息：“请在 Z 链上，向地址 A 发送 X 数量的 Y 代币”。钱包可以解析这个请求，然后自动引导用户完成操作（例如，一键切换网络并批准交易）。
- Cross-chain swaps and gas payment：跨链交易和 gas 支付。
- Light clients: users should be able to actually verify the chains that they are interacting with, and not just trust RPC providers.
- Keystore wallets: today, if you want to update the keys that control your smart contract wallet, you have to do it on all N chains on which that wallet exists. Keystore wallets are a technique that allow the keys to exist in one place (either on L1, or later potentially on an L2), and then be read from any L2 that has a copy of the wallet. 在 L1 设置变量，其他 L2s 都进行读取。
- More radical "shared token bridge" ideas: 构建一个新的 rollup 用于记录所有 L2s 的 token balance，比如 USDC 的数量。然后用户发送请求，可以进行账本层面的数据变更。这样不需要通过 L1 进行“换汇“操作，提高效率。
- Synchronous composability: allow synchronous calls to happen either between a specific L2 and L1, or between multiple L2s. This could be helpful in improving financial efficiency of defi protocols. 同步进行。

我认为 L2 互操作性不只是一个技术问题，还有商业和政治的问题。比如共享代币桥，它要求相互竞争的公司投入巨大成本，去共建一个让用户可以更轻松地“离开自己、投奔对手”的设施。这在商业上是反直觉的。调成本极高: 治理、标准制定、成本分摊等都是巨大的政治难题。

其他的互操作性方案：

- 共享 sequencer：多条 rollup 共用一批 sequencer，同时向多个 L2 出块，天然提供跨链 rollup 同步调用
- “聚合有效性证明”桥（ZK light-client bridge）：每条 Rollup 把自己的有效性证明 (ZK-SNARK) 递交到一个“证明汇总层”或直接递交给对方 Rollup；对方在本链内验证该证明后即可信任对方状态。
- 乐观跨 Rollup 消息层（Optimistic Message Layer）：把 L2→L2 消息先“乐观”relay，再开放挑战窗口；若期间无人提交欺诈证明则确认。
- 意图/订单流撮合网络（Intent-Based / Liquidity Layer）：用户只声明“意图”（intent），去中心化执行器以最佳路径完成操作，可跨多个 L2。
- 共同数据可用层 / 结算层（Shared DA & Settlement）：多条 L2 把区块数据提交到同一 DA 链（Celestia, EigenDA）并在同一结算链（如 L1 或 EigenLayer AVS）互验证。

## Scaling execution on L1

最简单的方式就是 increase the gas limit。还有就是优化 Ethereum client software 的执行效率。几个方向：

- EOF：优化 EVM 执行效率。现在下掉了，打算做 RISC-V 的 EVM。
- Multidimensional gas pricing：一个区块只有一种定价规则，就是 gas limit，这个是有问题的。因为不同资源的消耗对节点的压力是不同的，比如计算消耗 CPU、数据消耗网络、存储消耗硬盘。比如一辆货车可以装 1000 公斤货物，可能是一块铅块，也可能是 5 车体积的棉花。前者浪费空间，后者浪费重量。目前货运公司是多维定价，比如按照体积重或者重量重，取高价收费。这样拆分 computation data and storage 的限制，可以组合起来打满。
- Reduce gas costs of specific opcodes and precompiles：之前为了避免 DoS 攻击，对 opcode 等设置了相应的 gas，现在安全没问题了，可以适当降低 overpriced。
- EVM-MAX and SIMD：EVM-MAX 旨在为 EVM 添加一组原生的、高效的指令，专门用于处理大数模运算。这是所有现代密码学（包括 ZK-SNARKs）的基石。SIMD 是单指令多数据，可以并行处理多个数据。结果: 你获得了一个可以并行执行海量、高速、复杂密码学运算的“协处理器”。这正是 ZK-SNARKs、STARKs 和其他高级密码学协议所需要的。

接下来的三个策略：

- Improve technology (eg. client code, stateless clients, history expiry) to make the L1 easier to verify, and then raise the gas limit. 优化客户端代码，让 L1 更容易验证，然后提高 gas limit。
- Make specific operations cheaper, increasing average capacity without increasing worst-case risks. 降低特定操作的成本，增加平均容量，同时降低最坏情况的风险。
- Native rollups (ie. "create N parallel copies of the EVM", though potentially giving developers a lot of flexibility in the parameters of the copies they deploy). 原生 rollup，就是“创建 N 个并行的 EVM 副本”，但可能给开发者提供很多灵活的参数。

搞清楚 L1 的终极愿景以及什么属于 L2 也很重要。所有东西都在 L1 上面还是很浪费的，也不太容易验证。所以可能把 DeFi、大资金、Sovereign Ownership 等放在 L1 上面，NFT、Game Assets、buying coffee 等放在 L2 上面。

那需要有一种很方便的方式，可以直接在 L2 上面一笔交易 L1 上面的钱。此外，就是对 L1 的使用量和使用场景有限，是否能支撑起来 L1 的价值？

L2 的 rollup 的终极目标是 Native Rollups，就是直接在 L1 上面运行一个 EVM，然后 L2 的 rollup 直接调用 L1 的 EVM。

假设 L1 是 macOS，那么 L2 就是应用。macOS 并不知道应用具体做了什么，应用也只能将优先的信息和数据存储在操作系统。Native Rollup 是 macOS 的虚拟机，L2 相当于创建了一个虚拟机，有更高的权限，可以访问更多的资源。很多基础的底层的逻辑，会被统一，这样 Rollup 就作为协议层的一等公民，被加入到主协议里面。

# 2025.06.27

## https://vitalik.eth.limo/general/2024/10/20/futures3.html

这一个路线是关于 PoS 中心化的问题。主要会产生两个问题：

1. block 出块的问题：侧重于算力、算法优势（MEV 提取）
2. staking 资产的问题：侧重于资金规模优势，锁定最多的 ETH

核心目标：

- 最小化 Ethereum staking layer 的中心化风险（MEV、staking pools）
- Minimize risks of excessive value extraction from users

## Fixing the block construction pipeline

Block construction is largely done through extra-protocol propser-builder separation with MEVBoost.

MEV extraction: use specialized algorithms to determine which transactions to includes, in order to maximize the value.

- PBS: proposer-builder separation, validators still propose blocks, but receive the payload from builders
- APS: attester-proposer separation, the entire slot becomes the builders' responsibility. Preferred.

The separation of powers helps keep validators decentralized.

A sandwich attack could cause users making token swaps to suffer significant losses from slippage.

# 2025.06.28

For fixing the block construction issue, we can break down the block production task: proposer chooses the transactions, and the builder can only choose the ordering and insert some txs of their own.

So we have Inclusion List. In short, at time T, staker creates an inclusion list. At time T+1, a block builder creates a block. The block must include every tx in the inclusion list, they can order and add their own txs.

FOCIL: Fork-choice-enforced inclusion lists proposals involve a committee of multiple inclusion list creators per block.

Single Inclusion List Creator → Easy to Censor
├── 1 proposer creates inclusion list
├── If proposer is malicious/bribed → transaction gets censored
└── Single point of failure

FOCIL solution:

Multiple Inclusion List Creators (Committee) → Censorship Resistant
├── k inclusion list creators (e.g., k=16)
├── Each creator independently decides which txs to include
├── Final inclusion list = UNION of all k lists
└── To censor: ALL k creators must collude (much harder)

FOCIL + APS：

Time T: Committee creates inclusion lists
├── 16 validators each create inclusion list
├── Lists aggregated into master inclusion list
└── Master list committed to fork choice

Time T+1: Auction-based block building
├── Builder auction determines final proposer
├── Winner MUST include all txs from master inclusion list
├── Winner CAN reorder txs and add new ones
└── Winner gets MEV extraction rights for additional txs

<!-- Content_END -->
