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

<!-- Content_END -->
