---
timezone: UTC+8
---

> 请在上边的 timezone 添加你的当地时区(UTC)，这会有助于你的打卡状态的自动化更新，如果没有添加，默认为北京时间 UTC+8 时区

# 你的名字

1. 自我介绍 chesley666
2. 你认为你会完成本次残酷学习吗？ 会
3. 你的联系方式（推荐 Telegram） rorozn*OvQ*

## Notes

<!-- Content_START -->

### 2025.06.16

- 本次学习目标

  - 学习 V 神关于以太坊升级路线的文章
  - geth 客户端：实践环境搭建，学习源码实现

- 基础概念
  - slot：时隙
    - 以太坊 PoS 中“每 12 秒”的出块时间窗口，是区块生成与最终性判断的基本节拍器。
    - 在每个 slot 里会发生什么？
      1. 选择 proposer（区块提议者）；
      2. proposer 打包交易，广播新区块；
      3. 其他验证者（attesters）在 slot 内对该区块进行投票；
      4. 这些投票用于共识、最终性判断和奖励发放；
    - 如果 slot 错过了：  
       没有 proposer 出块，slot 留空；  
       不影响后续 slot 的正常进行；
      提高 validator 在线率是 PoS 设计的重要部分。
    - 与其他单位的关系：
      - 1 slot 12 秒，区块的时间粒度
      - 1 epoch = 2 slot 约 6.4 分钟
      - 最终确认（2025.6）：2 个 epoch，大约 13 分钟
      - SSF 机制下的目标：1 个 slot 内完成最终确定
- [Possible futures of the Ethereum protocol, part 1: The Merge](https://vitalik.eth.limo/general/2024/10/14/futures1.html) | [中文翻译](https://mp.weixin.qq.com/s?__biz=MzI2NzExNTczMw==&mid=2653293318&idx=1&sn=de61234513ee58e091a4fd04e4f5927f&scene=21#wechat_redirect)  
  需要升级的原因：

  - 加快最终性（Finality）：目前需等待 2–3 个 epoch（约 15 分钟），不够即时。
  - 降低单人质押门槛：当前需至少 32 ETH，门槛高，阻碍个体参与。
  - 增强抗攻击能力：提高系统对 51% 攻击、finality 回滚、阻塞及审查的抵抗与恢复能力。

- 技术方向
  - 单槽最终性 SSF（Single‑slot finality）  
     目标：缩短到单个 slot（约 12 秒）即可 finality，并支持 1 ETH 起质押。  
     三个主要方案：
    1. 强力签名聚合（Brute‑force SSF）：采用 ZK‑SNARKs 等技术在短时间内聚合百万级签名；
       - 在一个 slot 内收集并聚合成千上万个验证者的签名；
       - 需要使用高效的聚合签名方案，如：
         ZK SNARKs/STARKs（证明某个签名集合是有效的）；  
          BLS 聚合优化；  
          Horn、BLS-Trees 等研究协议；
       - 挑战：
         - 对带宽、计算要求高；
         - ZK 技术仍存在延迟；
         - 节点负担重。
    2. Orbit 委员会机制：随机抽取中等规模的 validator committee，实现经济假定的最终性；
       - 每个 slot 只抽取一个小规模委员会（比如 2048 个验证者）进行快速签名投票；
       - 大幅降低签名数量要求；
       - 但安全模型变为“抗少数攻击者”，而非“抗大多数攻击者”（因为整个系统只信任小委员会）；
       - 引入惩罚机制确保委员会成员诚实；
       - 挑战：安全性略弱于全体验证者投票；但可通过惩罚强化。
    3. 双层质押架构：高 ETH 层负责最终性，低 ETH 层具备部分参与权（如权证、打包等）。
       - 将验证者分为两层：
         - 高质押层（high-stake layer）：例如 1000+ ETH 的大户，负责最终性；
         - 低质押层（low-stake layer）：比如 1 ETH 起的小节点，参与共识或打包但不影响最终性；
       - 可让更多人参与但不牺牲最终性速度；
       - 挑战：经济和治理模型设计复杂，可能加剧中心化。
  - 单保密领导者选举（Single secret leader election, SSLE）  
     问题：当前 block proposer 是公开的，容易被 DoS。  
     方案：基于 mix‑net 混洗 blinded IDs（如 Whisk 协议），隐藏 proposer 身份。防止选举结果被追踪。
  - 加速交易确认  
     目标：将 slot 时间缩短至 4–8 秒，或通过 proposer 实时“预确认”交易，实现几秒内确认体验。  
     方向：
    - 缩 slot 时长；
    - 引入 proposer pre‑confirmation，同时需协调 attester-proposer separation 机制，在每笔交易发布时即时生成预确认。
  - 其他研究方向
    - 增强 51% 攻击自动恢复能力，避免社区手动软分叉。
    - 提升 quorum 阈值（如从 67% 提升至 80%），加强安全性。
    - 考虑量子抗性方案，未来需将 BLS 签名替换为抗量子算法。

### 2025.06.17

### 2025.06.18

- [Possible futures of the Ethereum protocol, part 2: The Surge](https://vitalik.eth.limo/general/2024/10/17/futures2.html) | [中文翻译](https://mp.weixin.qq.com/s?__biz=MzI2NzExNTczMw==&mid=2653293546&idx=1&sn=4170dca0fac69556c3e46539867bfeb7&scene=21#wechat_redirect)
- Surge（激增）阶段目标：
  - 通过 L1+L2 达到 100,000+ TPS；
  - 保持 L1 的 去中心化与稳健性；
  - 部分 L2 完全继承以太坊的核心属性（信任最小化、开放、抗审查）；
  - 实现 跨 L2 最大互操作性，让用户感受像在“一个以太坊”里操作
- Surge 核心技术路径

  1. 可扩展性三角悖论

     维持 去中心化、可扩展、与安全性之间的平衡极具挑战；

     Surge 通过将执行高度搬到 Rollup，高效利用 L1 的数据功能来缓解该三角难题。
     chaincatcher.com+4vitalik.eth.limo+4medium.com+4

  2. 数据可用性采样（DAS）

     Rollup 增加数据上链带来了带宽压力；

     利用 EIP‑4844 blobs 已提升带宽；

     下一步引入 DAS（如 PeerDAS/EIP‑7594），只需采样数据片段即可验证可用性，实现真正弹性的数据扩容。
     vitalik.eth.limo+8medium.com+8m.theblockbeats.info+8

  3. 数据压缩

     除了采样，还需引入更强的数据压缩技术，减少链上数据占用（文章提及未来研究方向但未深入具体方案）。

  4. 泛化 Plasma（Generalized Plasma）

     回顾旧方案 Plasma，在 modern Rollup 框架下引入类似机制，实现更轻的状态证明与退出设计，以应对高频小额交易场景。
     m.theblockbeats.info+4vitalik.eth.limo+4linkedin.com+4

  5. L2 证明系统成熟

     鼓励 L2 proof 技术进步（例如 ZK rollup 的证明性能、证明尺寸优化）；

     为 Rollup 承诺高吞吐、低成本准备基础。
     news.ycombinator.com+13vitalik.eth.limo+13linkedin.com+13

  6. 跨 L2 互操作与 UX

     改善用户在不同 Rollup 之间资产与体验迁移；

     打造类似“以太坊全局”统一 UX。

  7. L1 本身执行扩展

     在 L1 直接提升执行能力仍是选项，比如提高 gas limit、优化 EVM（如 EOF 字节码）；

     目标让 L1 可处理更多直接执行而非全部依赖 L2

### 2025.06.19

<!-- Content_END -->
