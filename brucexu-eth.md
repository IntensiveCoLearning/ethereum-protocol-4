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

<!-- Content_END -->
