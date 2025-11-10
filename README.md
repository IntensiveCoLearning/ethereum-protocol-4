# 以太坊底层协议残酷共学 第四期

## 介绍

以太坊协议层是区块链技术的核心组成部分，它定义了网络如何达成共识、如何验证交易、如何维护状态等关键机制。本次残酷共学将深入探讨以太坊协议层的各个方面，包括但不限于：

- 共识机制：从 PoW 到 PoS 的演进
- 网络协议：P2P 网络、节点发现、区块同步
- 状态管理：账户模型、状态树、存储结构
- 交易处理：交易池、Gas 机制、执行引擎
- 分片技术：数据可用性、跨分片通信
- 密码学基础：签名算法、哈希函数、零知识证明

通过本次残酷共学，你将收获对于以太坊协议层（包括但不限于共识算法、网络运行等）非常深入的理解，也将获得跟以太坊协议层核心开发者、华语区 EPF Fellow 与资深开发的直接交流机会！

本次共学也作为 EPF 中文小组的一个活动。[EPF（Ethereum Protocol Fellowship）](https://github.com/eth-protocol-fellows/cohort-five/blob/main/program-guide/program-details.md) 是由 EF 发起并资助的奖学金计划，每年一期，旨在从社区中遴选出 20 至 30 名优秀开发者参与。在奖学金计划期间，EF 提供导师指导和资金支持，帮助开发者迅速掌握以太坊协议层及客户端相关知识。通常，EPF 的报名在每年 4-5 月左右开启，经过大约半年的实践开发，最终在 11 月底的 Devcon/Devconnect 大会上展示成果。我们的学习素材和节奏，将会跟随 EPF 的进展一起。

## 关键词

Ethereum, Protocol, 以太坊底层协议, EPF

## 面向人群

- 对以太坊底层协议概念及概览有一定了解的学生、研究员或开发者
- 有意申请并参加 EPFsg 和 EPF，希望为以太坊协议层做出贡献的朋友
- 最好具备一定的编程基础，因为本课程涉及一定技术深度
- 对 Dapp 和以太坊开发感兴趣的朋友，包括一些基础知识

## 报名时间

- 报名开始时间：2025-06-07
- 报名结束时间：2025-06-15

## 共学时间

- 共学开始时间：2025-06-16
- 共学结束时间：2025-07-11

## 发起人

- 姓名：Bruce Xu
- GitHub ID：brucexu-eth
- Telegram：brucexu_eth
- Email：brucex2710@gmail.com
- 助教：Echo

## 发起组织

- [ETHPanda](https://ethpanda.org/) <img alt="organization-logo" height="60px" src="https://cdn.lxdao.io/10aed40b-4786-4c2b-aaaa-b7d8a119c00e.png" />

## 请假规则

每周请假 2 次

## 社群

Telegram：https://t.me/ETHPandaOrg/5427

## 学习资料/课程安排

学习方式主要以自学每天打卡，每周我们会设置一个周会，方便大家交流、答疑，也会邀请相关专家来做分享。

### 学习资料

#### 核心资料

- EPFsg Wiki 及官网：<https://epf.wiki/#/> 中文版本（由 ETHPanda 社区翻译）：<https://zh.epf.wiki/#/>
- [Ethereum.org](https://ethereum.org/) 官方文档

Vitalik 的协议系列文章：

- [Possible futures of the Ethereum protocol, part 1: The Merge](https://vitalik.eth.limo/general/2024/10/14/futures1.html) | [中文翻译](https://mp.weixin.qq.com/s?__biz=MzI2NzExNTczMw==&mid=2653293318&idx=1&sn=de61234513ee58e091a4fd04e4f5927f&scene=21#wechat_redirect)
- [Possible futures of the Ethereum protocol, part 2: The Surge](https://vitalik.eth.limo/general/2024/10/17/futures2.html) | [中文翻译](https://mp.weixin.qq.com/s?__biz=MzI2NzExNTczMw==&mid=2653293546&idx=1&sn=4170dca0fac69556c3e46539867bfeb7&scene=21#wechat_redirect)
- [Possible futures of the Ethereum protocol, part 3: The Scourge](https://vitalik.eth.limo/general/2024/10/20/futures3.html) | [中文翻译](https://mp.weixin.qq.com/s?__biz=MzI2NzExNTczMw==&mid=2653293588&idx=1&sn=c9ef3a890934cbbc9a4a6f78407aef03&scene=21#wechat_redirect)
- [Possible futures of the Ethereum protocol, part 4: The Verge](https://vitalik.eth.limo/general/2024/10/23/futures4.html) | [中文翻译](https://mp.weixin.qq.com/s?__biz=MzI2NzExNTczMw==&mid=2653293705&idx=1&sn=7cc3650946fd73453170b2a62d8630c3&scene=21#wechat_redirect)
- [Possible futures of the Ethereum protocol, part 5: The Purge](https://vitalik.eth.limo/general/2024/10/26/futures5.html) | [中文翻译](https://mp.weixin.qq.com/s?__biz=MzI2NzExNTczMw==&mid=2653293868&idx=1&sn=8aaea12f3c659de616b995ebed4f85f8&scene=21&poc_token=HMxbS2ijcoqoJhWgC0lmFtA2oj2BVQr_XNg2B9kF)
- [Possible futures of the Ethereum protocol, part 6: The Splurge](https://vitalik.eth.limo/general/2024/10/29/futures6.html) | [中文翻译](https://mp.weixin.qq.com/s/aeWLSl2No8A3oSdZp2no3g)

EPF 的官方学习资料和进展：

我们会将 EPF 官方的学习资料和计划及时同步在这里。

#### 扩展技术文档

- [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf) - 以太坊黄皮书
- [Ethereum 2.0 Specifications](https://github.com/ethereum/consensus-specs) - 以太坊 2.0 规范
- [Ethereum Execution Layer Specs](https://github.com/ethereum/execution-specs) - 执行层规范

#### 客户端实现

- [Geth](https://github.com/ethereum/go-ethereum) - Go 语言的客户端实现
  - [Geth 源码系列 I：Geth 整体架构](https://forum.lxdao.io/t/geth-i-geth/2856)
  - [Geth 源码系列 II：存储设计及实现](https://forum.lxdao.io/t/geth-ii/2857)
  - [Geth 源码系列 III：p2p 网络设计及实现](https://forum.lxdao.io/t/geth-iii-p2p/2891)
  - Geth 源码系列正在连载中，将会及时更新

### 每周协议相关技术分享会

每周四晚上北京时间 7 点到 8 点我们会开设一个在线周会：

Topic: 以太坊底层协议残酷共学周会 | ETHPanda

Download ICS: https://us06web.zoom.us/meeting/tZ0qcu6gqDMsGtGDtG_T40zT1dtfgraB5YW0/ics?icsToken=DJAYOl_c3IewPXRIYwAALAAAAHp06FAnvcicucaTsbqik0eE_3DeQegXjSq9J1a0TeyWHg9Daarl5p_ehum7ho1EimC0mEO8khe0sjiprzAwMDAwMQ&meetingMasterEventId=4NPP49XuR9KU68ngQD9dYA

Join Zoom Meeting
https://us06web.zoom.us/j/89757868605?pwd=dIbMquOGLedKul6E7CtsML8HmpzPiM.1

Meeting ID: 897 5786 8605
Passcode: 955074

内容包括：

- 学员笔记分享，最近问题交流
- 华语区 Fellow 及资深开发技术分享
- 结识更多朋友

## 共学激励

我们认为学习到知识就是最大的财富，希望报名的同学以深入学习以太坊知识为首要目标。

## 报名和打卡规则

因为残酷共学的报名和打卡是基于 GitHub 进行开展的，如果你是非开发者或者对 git 操作不熟悉，请先阅读此文档：[残酷共学 GitHub 新手教程](https://www.notion.so/lxdao/GitHub-bd65b981146947fea1fb675942567a45)

- 报名:

  - Step01：Fork 本仓库。
  - Step02：复制 Template.md 创建你的个人笔记文件，并根据文档指引填写你的信息，并将文件重命名为你的 GitHub ID：YourGitHubID.md。
  - Step03：创建一个 PR 到当前仓库，本残酷共学助教会对你的 PR 进行 review，review 通过后，你的 PR 会被 merge 到 main 分支，这个时候你会收到邀请加入这个仓库 contribution 的邮件，接受邀请后，你会自动获得 main 分支的 push 权限。
  - Step04：完成以上三个步骤，恭喜你报名成功，后续就可以将你的学习记录直接 push 到 main 分支进行更新。
  - 请加入 xxx 群组保持交流：（请添加你创建的群组链接)。加入群组后请在群里报到一下方便助教记录。

- 打卡：
  - 报名成功后，你将拥有 main 分支的 push 权限，你需要将每天学习笔记按日期更新到你的 YourName.md 文档中，提交更新后，我们会自动更新你的打卡状态到下面的打卡记录表。
  - 如果你不在 UTC+8 时区，需要添加时区 code 到你的 YourName.md 文件的开始，错误的时区设置可能会使自动化打卡脚本错误计算打卡时间，具体请参考：https://github.com/IntensiveCoLearning/template/blob/main/Template.md?plain=1#L1
  - 当你提交笔记时，请确保以下几点，否则打卡可能会失败：
    - 在 YourName.md 文档，请将笔记内容放到以下代码块中，且 `<!-- Content_START -->` 和 `<!-- Content_END -->` 不能删除:
    ```
    <!-- Content_START -->
    ### 日期
    笔记内容
    <!-- Content_END -->
    ```
    - 日期格式为 `### 2024.07.11`，请不要随意更改

## 残酷共学打卡记录表

✅ = Done ⭕️ = Missed ❌ = Failed

<!-- START_COMMIT_TABLE -->
| Name | 6.16 | 6.17 | 6.18 | 6.19 | 6.20 | 6.21 | 6.22 | 6.23 | 6.24 | 6.25 | 6.26 | 6.27 | 6.28 | 6.29 | 6.30 | 7.01 | 7.02 | 7.03 | 7.04 | 7.05 | 7.06 | 7.07 | 7.08 | 7.09 | 7.10 | 7.11 |
| ------------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| [brucexu-eth](https://github.com/IntensiveCoLearning/ethereum-protocol-4/blob/main/brucexu-eth.md) | ✅ | ✅ | ✅ | ✅ | ✅ | ⭕️ | ⭕️ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ⭕️ | ✅ | ✅ | ⭕️ | ✅ | ✅ | ✅ | ✅ | ✅ | ⭕️ | ⭕️ |   | ✅ |
| [aliceyzhsu](https://github.com/IntensiveCoLearning/ethereum-protocol-4/blob/main/aliceyzhsu.md) | ✅ | ✅ | ✅ | ✅ | ✅ | ⭕️ | ⭕️ | ✅ | ✅ | ✅ | ✅ | ⭕️ | ✅ | ⭕️ | ⭕️ | ⭕️ | ❌ | | | | | | | | | |
| [Juktong](https://github.com/IntensiveCoLearning/ethereum-protocol-4/blob/main/Juktong.md) | ⭕️ | ✅ | ✅ | ✅ | ⭕️ | ✅ | ✅ | ⭕️ | ⭕️ | ✅ | ✅ | ✅ | ❌ | | | | | | | | | | | | | |
| [rockyfang](https://github.com/IntensiveCoLearning/ethereum-protocol-4/blob/main/rockyfang.md) | ⭕️ | ✅ | ✅ | ⭕️ | ❌ | | | | | | | | | | | | | | | | | | | | | |
| [EchoZheng2333](https://github.com/IntensiveCoLearning/ethereum-protocol-4/blob/main/EchoZheng2333.md) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ⭕️ | ⭕️ | ✅ | ⭕️ | ✅ | ✅ | ⭕️ | ❌ | | | | | | | |
| [jitash](https://github.com/IntensiveCoLearning/ethereum-protocol-4/blob/main/jitash.md) | ⭕️ | ✅ | ✅ | ✅ | ✅ | ✅ | ⭕️ | ✅ | ✅ | ⭕️ | ✅ | ⭕️ | ✅ | ❌ | | | | | | | | | | | | |
| [MrWynn](https://github.com/IntensiveCoLearning/ethereum-protocol-4/blob/main/MrWynn.md) | ✅ | ✅ | ✅ | ✅ | ✅ | ⭕️ | ⭕️ | ⭕️ | ⭕️ | ❌ | | | | | | | | | | | | | | | | |
| [OrionWu](https://github.com/IntensiveCoLearning/ethereum-protocol-4/blob/main/OrionWu.md) | ✅ | ✅ | ⭕️ | ⭕️ | ❌ | | | | | | | | | | | | | | | | | | | | | |
| [chesley666](https://github.com/IntensiveCoLearning/ethereum-protocol-4/blob/main/chesley666.md) | ✅ | ⭕️ | ✅ | ⭕️ | ❌ | | | | | | | | | | | | | | | | | | | | | |
| [ohko4711](https://github.com/IntensiveCoLearning/ethereum-protocol-4/blob/main/ohko4711.md) | ✅ | ✅ | ✅ | ✅ | ✅ | ⭕️ | ⭕️ | ⭕️ | ⭕️ | ❌ | | | | | | | | | | | | | | | | |
| [fffuuuming](https://github.com/IntensiveCoLearning/ethereum-protocol-4/blob/main/fffuuuming.md) | ✅ | ✅ | ✅ | ✅ | ✅ | ⭕️ | ⭕️ | ✅ | ⭕️ | ✅ | ✅ | ✅ | ⭕️ | ✅ | ⭕️ | ⭕️ | ❌ | | | | | | | | | |
| [qingoba](https://github.com/IntensiveCoLearning/ethereum-protocol-4/blob/main/qingoba.md) | ⭕️ | ⭕️ | ❌ | | | | | | | | | | | | | | | | | | | | | | | |
| [kuove](https://github.com/IntensiveCoLearning/ethereum-protocol-4/blob/main/kuove.md) | ✅ | ✅ | ✅ | ✅ | ✅ | ⭕️ | ⭕️ | ✅ | ✅ | ✅ | ✅ | ✅ | ⭕️ | ⭕️ | ✅ | ✅ | ✅ | ✅ | ✅ | ⭕️ | ⭕️ | ✅ | ✅ | ✅ |   | ⭕️ |
| [big-dudu-mosty](https://github.com/IntensiveCoLearning/ethereum-protocol-4/blob/main/big-dudu-mosty.md) | ✅ | ⭕️ | ✅ | ✅ | ⭕️ | ✅ | ✅ | ⭕️ | ⭕️ | ✅ | ❌ | | | | | | | | | | | | | | | |
| [Bo](https://github.com/IntensiveCoLearning/ethereum-protocol-4/blob/main/Bo.md) | ✅ | ✅ | ✅ | ✅ | ⭕️ | ⭕️ | ✅ | ⭕️ | ⭕️ | ✅ | ✅ | ✅ | ✅ | ✅ | ⭕️ | ⭕️ | ❌ | | | | | | | | | |
| [0xAlexWu](https://github.com/IntensiveCoLearning/ethereum-protocol-4/blob/main/0xAlexWu.md) | ⭕️ | ⭕️ | ❌ | | | | | | | | | | | | | | | | | | | | | | | |
<!-- END_COMMIT_TABLE -->











































































































































































































































































































<!-- STATISTICALDATA_START -->
## 统计数据

- 总参与人数: 16
- 完成人数: 2
- 完成用户: brucexu-eth, kuove
- 全勤用户: 
- 淘汰人数: 14
- 淘汰率: 87.50%
- Fork人数: 21
<!-- STATISTICALDATA_END -->
