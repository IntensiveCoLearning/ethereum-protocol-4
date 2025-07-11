---
timezone: UTC+8
---

> 请在上边的 timezone 添加你的当地时区(UTC)，这会有助于你的打卡状态的自动化更新，如果没有添加，默认为北京时间 UTC+8 时区


# big-dudu-mosty

1. 自我介绍
    菜鸟级 dapp 开发选手，希望通过这次 EPF 的学习能够加深对以太坊的了解
2. 你认为你会完成本次残酷学习吗？
    会
3. 你的联系方式（推荐 Telegram）
    @dudu_1021
## Notes

<!-- Content_START -->

### 2025.06.16

学习了https://epf.wiki/#/eps/week1中的以太坊基础知识，并完成相关测试 https://ethereum.org/quizzes 简单读了一下https://vitalik.eth.limo/general/2024/10/14/futures1.html Vitalik 的协议系列文章1 笔记：

The Merge 的意义与现状： The Merge 指以太坊从工作量证明（PoW）转向权益证明（PoS），是以太坊历史上最重要的升级之一。这一转变极大地降低了能耗，并为以太坊网络的可持续发展奠定了基础。目前，以太坊 PoS 运行稳定，但在去中心化、性能和抗中心化风险等方面仍有进一步提升的空间。
未来改进目标： 单槽终局（Single Slot Finality, SSF）：将区块终局时间从15分钟缩短到1个slot（约12秒），大幅提升用户体验和安全性。 降低质押门槛：将单个验证者的最低质押从32 ETH降至1 ETH，鼓励更多个人参与质押，增强网络去中心化。 提升鲁棒性：增强以太坊对51%攻击（如终局回滚、阻止终局、审查等）的抵抗和恢复能力，提升整体安全性。
单槽终局的挑战与方案： 实现单槽终局的目标是既要加快终局速度、降低质押门槛，又不能让节点负担过重。主要技术方案包括： 暴力聚合签名：采用更高效的签名聚合协议（如ZK-SNARKs），以便在极短时间内处理百万级验证者的签名。 Orbit委员会机制：通过随机选中中等规模的委员会参与终局，兼顾效率和经济终局性，降低节点压力。 双层质押机制：将质押者分为高额和低额两类，高额质押者负责终局，低额质押者可委托或参与部分职责，既降低门槛又需防范中心化风险。
单一秘密领导者选举（SSLE）： 目前，以太坊下一个出块者的身份是提前公开的，容易受到DDoS攻击。SSLE的目标是通过加密技术隐藏出块者身份，直到出块时才公开，从而提升网络安全性。难点在于如何实现既简单又可扩展、同时具备抗量子攻击能力的协议。
更快的交易确认： 以太坊希望将slot时间从12秒进一步缩短到4秒，以提升用户体验和DeFi协议的效率。实现方法包括缩短slot时间或允许出块者实时发布预确认信息。但过短的slot时间可能导致节点集中在低延迟地区，影响网络的去中心化。
其他研究方向： 51%攻击恢复：减少对“社会层”软分叉的依赖，提升自动化恢复能力，增强网络自愈性。 提高终局门槛：将区块终局所需的支持率从67%提升至80%，进一步增强安全性。 抗量子攻击：为未来量子计算威胁做准备，开发抗量子签名方案，保障以太坊长期安全。

### 2025.06.17

### 2025.06.18

https://vitalik.eth.limo/general/2024/10/17/futures2.html

以太坊协议的未来（二）：The Surge 阶段学习笔记
一、背景与核心目标
以太坊早期有两条扩容路线：分片（sharding）和Layer2协议。
目前主流扩容方案是rollup-centric（以rollup为中心），即L1专注安全和去中心化，L2负责扩展性。
2023年后，EIP-4844等升级大幅提升了L1数据带宽，多个EVM rollup已进入“stage 1”阶段。
目标：L1+L2合计10万+TPS，保持L1去中心化和安全，L2间高度互操作。

二、扩容三难问题（Scalability Trilemma）
三难：去中心化（节点运行门槛低）、扩展性（高TPS）、安全性（攻击者难以作恶）。
传统观点认为三者不可兼得，但数据可用性采样（DAS）+ SNARKs有望突破三难。
Plasma等架构通过激励机制让用户自己监控数据可用性，也是一种思路。

三、数据可用性采样（DAS）进展
Dencun升级后，以太坊每12秒可处理3个“blob”，总带宽约375KB/slot，极限TPS约173（仅blob数据）。
未来目标：通过PeerDAS、2D采样等技术，将带宽提升到16MB/slot，理论支持5.8万TPS。
PeerDAS：每个节点只需采样一小部分数据，极大降低全节点压力。
2D采样：进一步提升效率，便于分布式区块构建。

四、数据压缩
Rollup每笔交易上链数据约180字节，数据压缩可进一步提升TPS。
技术手段包括：零字节压缩、签名聚合（BLS）、地址指针替换、自定义序列化等。
目标：让每笔交易占用更少字节，提升整体吞吐量。

五、通用Plasma方案
Plasma通过链下处理、链上提交Merkle根，极大提升扩展性。
结合SNARKs后，Plasma可支持更广泛资产类型，挑战机制更简单。
Hybrid plasma/rollup方案（如Intmax）可实现极高TPS和隐私性。

六、L2证明系统成熟化
目前大多数rollup还未完全去信任化，安全理事会可干预。
目标是达到Stage 2：完全信任的证明系统，理事会仅能在代码有bug时介入。
路径：形式化验证（Formal Verification）、多证明人（Multi-provers）等。

七、跨L2互操作性与用户体验
现状：L2之间转账、交互体验差，容易出错。
目标：让L2间转账像L1内一样简单，提升钱包、协议标准化。
技术方向：链特定地址、链特定支付请求、跨链swap与gas支付、轻客户端、keystore钱包、共享token桥等。

八、L1执行扩容
L2扩容后，L1也需提升处理能力，否则会影响ETH经济安全和L2生态。
方案包括：提升gas上限、优化EVM字节码（EOF）、多维gas定价、降低部分opcode gas费、EVM-MAX/SIMD等。
终极方案：原生rollup（enshrined rollups），即L1内置多个并行EVM副本。

九、未来展望与权衡
L1和L2的分工需有明确原则，不能盲目提升L1负载，避免去中心化和安全性受损。
许多技术路线（如DAS、Plasma、原生rollup）都在权衡扩展性、复杂性和安全性。
社区协作、标准化和技术创新是实现最终目标的关键。

### 2025.06.19
以太坊协议未来展望（三）：The Scourge 笔记

一、以太坊L1的中心化风险
以太坊权益证明（PoS）面临的最大风险之一是由于经济压力导致的中心化。大质押者更容易通过规模经济获得更高收益，小质押者则倾向于加入大型池，进而导致：
51%攻击风险上升
交易审查等危机
价值被少数人攫取
风险主要体现在两个方面：
区块构建（Block Construction）
质押资本提供（Staking Capital Provision）
大质押者能运行更复杂的MEV（最大可提取价值）算法，获得更高收益；还能更好地应对资金锁定的不便（如发行流动性质押代币LST）。

二、区块构建管道的改进
1. 当前问题
目前区块构建主要通过协议外的“提议者-构建者分离”（PBS）实现，MEVBoost是主流方案。
区块内容选择权被高度专业化的构建者掌控，中心化风险大。
两大构建者控制了约88%的区块内容选择权，存在审查和市场操纵风险。
2. 主要解决方案
Inclusion Lists（纳入列表）
由提议者（质押者）生成一份有效交易列表，构建者必须全部包含，但可自行排序并添加自有交易。
FOCIL（Fork-choice-enforced inclusion lists）方案：每个区块由多个提议者组成委员会，需全部成员一致才能审查交易，极大提升抗审查性。
APS（Attester-Proposer Separation）
APS进一步将区块提议权分离，减少提议者与构建者的利益绑定，降低中心化风险。
BRAID（多提议者并行方案）
多个提议者并行生成交易列表，最终按确定性算法（如按手续费高低）排序。
但存在“最后行动套利”和“专属订单流”问题，仍需依赖加密内存池（Encrypted Mempool）保障公平。
加密内存池
用户广播加密交易，区块构建时内容不可见，后续再解密。
主要难点在于确保所有交易最终都能被解密和包含，常用门限解密或延迟加密技术。

三、质押经济学的改进
1. 当前问题
约30%的ETH已质押，远超安全所需。
若质押比例过高，可能导致：
所有ETH持有者都被动参与质押，进一步中心化
惩罚机制可信度下降
单一LST主导网络，甚至取代ETH本身
以太坊无谓增发
2. 主要方案
两级质押（Two-tier Staking）
设置“风险承担型”（可被惩罚）和“无风险型”（不可被惩罚）两类质押，前者数量有限，后者人人可参与。
或者直接调整发行曲线，质押比例过高时大幅降低收益。
MEV收益捕获
目前MEV收益归提议者所有，导致收益波动大、激励失衡、难以通过调整发行率限制质押规模。
方案：通过提前拍卖区块提议权等方式，将MEV收益纳入协议捕获和分配。

四、应用层解决方案
专用质押硬件（如Dappnode）降低节点运营门槛
Squad Staking（多签质押）让多人协作质押更便捷
针对独立质押者的空投激励
去中心化区块构建市场，结合ZK、MPC、TEE等技术提升隐私与抗审查性
应用层MEV最小化设计，如操作排队、延迟执行、链下撮合等

五、未来展望与挑战
不同方案可组合推进，如先实施FOCIL+APS，再逐步过渡到BRAID。
需持续优化加密内存池、纳入列表设计、APS拍卖机制等细节。
质押经济学需平衡收益、惩罚与去中心化目标，防止LST主导和中心化。
单槽终结性（Single Slot Finality）等其他协议升级与区块构建方案密切相关，需协同设计。

### 2025.06.21
以太坊协议未来展望——第四部分：The Verge 读书笔记

核心概念
The Verge 是以太坊路线图的重要组成部分，最初指的是将以太坊状态存储迁移到 Verkle 树结构，但现在已扩展为更宏大的愿景：使以太坊链验证变得资源高效，使任何移动钱包、浏览器钱包甚至智能手表都能默认进行完全验证。

The Verge 的关键目标
无状态客户端：完全验证客户端和质押节点只需几 GB 存储空间
长期目标：在智能手表上完全验证链（共识和执行）
无状态验证：Verkle 树 vs STARKs

要解决的问题
目前以太坊客户端需要存储数百 GB 的状态数据才能验证区块，且这个数量每年增加约 30 GB。这减少了能运行全验证节点的用户数量，首次设置节点也需要数小时或数天下载状态。

无状态验证技术方案
无状态验证允许节点不需要拥有完整状态就可以验证区块，通过提供见证数据（witness）实现：
包含区块将访问的特定状态位置的值（代码、余额、存储等）
证明这些值正确的加密证明

Verkle 树方案
使用基于椭圆曲线的向量承诺来创建更短的证明
每个父子关系的证明部分只有 32 字节，无论树宽度如何
理论最大证明大小约为 1.3 MB（实际约 2.6 MB）

STARKed 二进制哈希树方案
在二叉树上进行验证，最大 10.4 MB 的证明被 STARK 替代
证明本身只包括被证明的数据，再加上 STARK 的 100-300 kB 固定开销
主要挑战是证明者时间（prover time）

EVM 执行的有效性证明
目标
长期目标是通过下载区块（甚至使用数据可用性采样只下载部分区块）和验证小型证明来验证以太坊区块。

实现方式
EVM 在以太坊规范中定义为状态转换函数：
有预状态 S、区块 B，计算后状态 S' = STF(S, B)
轻客户端持有预状态根 R、后状态根 R' 和区块哈希 H
需证明的内容包括：状态访问的正确性、状态转换的正确性、根重计算的正确性

当前挑战
安全性：确保 SNARK 真正验证 EVM 计算，没有漏洞
解决方案：多证明者和形式化验证
证明者时间：任何以太坊区块能在 4 秒内被证明
需要在三个方向推进：
并行化：每个 GPU 处理一个步骤，然后进行"聚合树"
证明系统优化：使用 Orion、Binius、GKR 等新证明系统
EVM gas 成本和其他更改：优化 EVM 更具证明友好性
共识的有效性证明

面临的挑战
以太坊共识涉及处理存款、提款、签名、验证者余额更新等，要完全验证区块，还需证明共识部分。
主要挑战在于：
每个区块需要为每个验证者证明 1-16 次 BLS12-381 椭圆曲线加法
当前每个插槽有约 3 万验证者签名，未来可能增至 100 万
需要验证配对运算（每个插槽最多 128 个）
SHA256 哈希验证大量使用

潜在解决方案
哈希函数更改：从 SHA256 改为 Poseidon 可获得约 100 倍提升
直接存储打乱的验证者记录（如果使用 Orbit）
签名聚合：使用递归证明在网络中分担验证负载

### 2025.06.22
以太坊协议的未来展望（五）：清理（The Purge）学习笔记
> 原文作者：Vitalik Buterin
> 原文链接：[Possible futures of the Ethereum protocol, part 5: The Purge](https://vitalik.eth.limo/general/2024/10/26/futures5.html)
一、背景与目标
以太坊面临的两大“膨胀”问题：
历史数据膨胀：所有历史交易、账户都需永久存储，导致节点负担和同步时间不断增加。
协议复杂度膨胀：新功能易加难删，代码复杂度随时间上升。
清理（The Purge）的目标是：
减少客户端存储需求（历史、状态）
简化协议，移除不必要的特性
同时保证区块链的“永久性”与可持续性
二、历史数据清理（History Expiry）
1. 问题
全节点需存储约1.1TB执行层+几百GB共识层数据，绝大部分为历史数据。
节点体积每年增长数百GB。
2. 解决思路
区块链的历史可通过哈希链和默克尔证明验证，无需每个节点都存全历史。
类似BT下载，每个节点只存一部分历史，整体冗余度不变。
现状：共识区块只保留6个月，Blobs只保留18天。EIP-4444提议历史区块/收据只保留1年。
3. 未来方案
节点只需存储最近一段时间的数据，老数据通过P2P网络（如Portal network）分布式存储。
可用纠删码提升数据鲁棒性。
参考资料：
EIP-4444
Portal network
4. 取舍
直接丢弃历史依赖中心化存档节点，简单但削弱“永久性”。
构建分布式存储网络，难度更高但更安全。
三、状态数据清理（State Expiry）
1. 问题
即使历史可清理，状态（账户余额、合约存储等）仍持续增长，每年约50GB。
状态难以过期，因为EVM假设状态永久可读。
2. 方案
2.1 部分状态过期（Partial State Expiry）
将状态分块，只有最近访问的块保留完整数据，其他只存32字节承诺（stub）。
访问过期数据需提供默克尔证明“复活”。
代表提案：EIP-7736
2.2 地址-周期状态过期（Address-period-based）
状态树按周期（如每年）新建，老树冻结。
只需存最近两棵树，访问老数据需带证明。
新对象写入需用新周期地址，兼容性需扩展地址格式（如32字节地址）。
2.3 地址空间扩展/收缩
扩展：地址从20字节变32字节，兼容性挑战大。
收缩：保留部分地址空间用于新格式，存在安全风险（如地址碰撞）。
3. 取舍
只做无状态化，不做状态过期，状态只需少数节点存储。
做部分状态过期，状态增长速率大幅降低。
做完全状态过期，需解决地址格式兼容与安全问题。
四、协议特性清理（Feature Cleanup）
1. 目标
简化协议，减少bug和维护成本，提高安全性和中立性。
2. 具体措施
移除SELFDESTRUCT等复杂/少用特性（已基本完成）。
RLP→SSZ序列化格式统一。
移除老交易类型、日志bloom过滤器、信标链委员会等。
EVM内部简化：气费机制、预编译合约、gas可见性、跳转指令等。
参考提案：EIP-4762、EIP-7668
3. 变更流程
讨论移除特性
分析影响，决定是否推进
正式EIP，主流工具链配合
多年过渡后正式移除
五、EOF与更激进的简化
EOF（EVM Object Format）：引入更严格的EVM格式，为未来升级和简化打基础。
更激进方案：将协议大部分逻辑移到合约层，L1只做最小化VM，EVM变成rollup之一。
六、与其他路线图的关系
历史/状态清理是实现极简节点、快速同步的前提。
状态过期有助于未来状态树格式升级。
协议简化与新特性推进可协同进行。

### 2025.06.25
以太坊协议未来展望学习笔记：第六部分 Splurge 杂项改进
一、EVM 改进核心内容
1. 解决的核心问题
静态分析困难：现有 EVM 难以进行代码优化与形式化验证
效率瓶颈：高级密码学操作需预编译支持，通用性不足
2. 关键改进方向
（1）EVM 对象格式（EOF）
核心特性：代码 / 数据分离、禁止动态跳转、新增子例程机制
影响：旧合约可兼容过渡，新合约通过字节码优化降低 Gas 消耗
路线图：计划纳入下一次硬分叉，是后续升级基础
（2）模块化算术扩展（EVM-MAX）
功能：新增模块化算术操作码，支持 Montgomery 乘法等优化
应用场景：为 STARK 证明、格密码等密码学运算提供底层支持
（3）SIMD 集成
设计思路：结合 EIP-6690 与 EIP-616，支持批量数据并行计算
性能提升：可加速哈希函数、椭圆曲线运算等场景达 10 倍以上
3. 待解决问题
需验证 EVM-MAX+SIMD 组合的实际 Gas 消耗
平衡 L1 协议复杂度与基础设施适配成本
二、账户抽象技术演进
1. 核心目标
突破 ECDSA 签名限制：支持抗量子密码、密钥轮换等场景
提升用户体验：实现 ERC20 支付 Gas、社交恢复钱包等功能
降低隐私协议依赖：减少对中继器的中心化依赖
2. 关键实现路径
（1）ERC-4337 协议外方案
双阶段处理：验证阶段仅访问自身账户，防范多账户失效攻击
扩展功能：Paymasters 代付机制、BLS 签名聚合提升效率
（2）EIP-7702 协议内初步落地
目标：为所有用户（含 EOA）提供账户抽象便捷特性
意义：避免生态分裂，短期快速提升用户体验
（3）EIP-7701 基于 EOF 的原生方案
技术创新：为账户设置独立验证代码段，替代传统签名验证
安全挑战：需平衡验证逻辑复杂度与 DoS 攻击防护
3. 未来挑战
如何安全实现智能合约控制交易验证
L1 与 L2 账户抽象方案的兼容性设计
三、EIP-1559 改进要点
1. 现有机制缺陷
目标区块利用率公式存在数学偏差（实际 50-53%）
极端流量下调整速度不足，资源定价不合理
2. 优化方向
（1）多维 Gas 机制
核心设计：将 Gas 分为执行、Blob、调用数据等独立维度
关键 EIP：
EIP-7706：为调用数据新增 Gas 维度
EIP-7623：限制最大调用数据量，缓解资源浪费
（2）动态调整算法升级
借鉴 EIP-4844：采用更高效的基础费用计算模型
目标：实现区块利用率的快速收敛与精准控制
3. 实施挑战
协议复杂度增加，需重构合约 Gas 限制逻辑
需优先在 EOF 环境中试点多维 Gas 机制
四、可验证延迟函数（VDF）技术
1. 核心价值
替代 RANDAO：提供不可操控的随机性来源
应用场景：提议者选择、链上随机数依赖应用
2. 技术原理
函数特性：只能顺序计算，无法通过并行加速（如 10^9 次哈希迭代）
安全风险：需防范 ASIC 硬件加速与意外并行化攻击
3. 落地现状
尚无完全满足需求的方案，仍处于研究阶段
短期可接受退化为 RANDAO 级安全的折中方案
五、密码学前沿探索
1. 不可区分混淆（Indistinguishability Obfuscation）
核心能力：创建隐藏内部逻辑的 "加密程序"，支持任意密码学原语
应用场景：通用可信设置、隐私保护 EVM、加密内存池
2. 量子一次性签名
技术特性：每个签名仅能用于特定类型消息一次
潜在影响：实现完美抗胁迫投票、量子货币等场景
3. 技术瓶颈
不可区分混淆现有实现效率极低（运行需 1 年以上）
量子计算机尚未普及，相关应用仍处于理论阶段
六、总结：Splurge 板块的战略意义
基础设施优化：EVM 与账户抽象是 Layer2 扩容的底层支撑
经济模型完善：EIP-1559 改进提升资源利用效率
长期技术储备：VDF 与前沿密码学为 5-10 年后的技术突破铺路

### 2025.06.27
以太坊执行层学习笔记：从架构到源码实践
一、执行层核心定位与架构概述
1. 状态机本质
   以太坊执行层是一个交易驱动的状态机，核心职能是通过 EVM 执行交易来更新链上状态（账户余额、合约数据等），并通过 P2P 网络与其他节点同步数据。
2. 合并（The Merge）后的分层架构
   执行层：负责交易执行、状态维护（如 Geth、Erigon）。
   共识层：负责区块共识（如 Prysm、Lighthouse）。
   通信方式：通过Engine API交互，执行层接收共识层的区块生成 / 验证指令。
二、执行层核心模块与源码结构
1. 三大基础组件
   网络（devp2p）
    实现节点发现（Kademlia 算法）和数据传输，源码在p2p/模块
    核心结构：enode.Node（节点信息）、discover.Table（节点发现表）
   计算（EVM）
     状态转换的唯一入口，源码在core/vm/
     关键结构：
        EVM：执行上下文（区块 / 交易信息、状态数据库）
        EVMInterpreter：指令解释器（处理 OpCode 执行）
        Contract：合约调用参数（调用者、输入数据、Gas）
   存储（ethdb）
       抽象底层数据库接口（LevelDB/Pebble），源码在ethdb/
   上层管理：
   rawdb：区块数据读写
   statedb：MPT 状态树管理
2. Geth 源码核心模块表
模块路径	功能描述

core/	区块链核心逻辑（区块管理、状态机、Gas 计算）

eth/	以太坊协议实现（区块同步、交易广播、Engine API）

node/	节点容器，管理组件生命周期（如 Ethereum 实例、RPC 服务）

p2p/	P2P 网络协议（节点发现、数据传输）

trie/	Merkle Patricia Trie 实现，用于存储账户状态和合约数据

三、执行层关键流程解析
1. 节点同步流程
   Full Sync
   从创世区块开始下载并验证所有区块，通过 EVM 重建状态数据库.
   Snap Sync
   直接下载最新检查点状态和后续区块，跳过全量验证，提升同步效率。
2. 区块处理核心流程
   共识层触发：通过Engine API通知执行层生成新区块
   交易池打包：从交易池获取待执行交易，通过 EVM 执行并更新状态
   区块验证：执行层验证共识层提交的区块，确保交易执行结果正确
3. 核心数据结构
   Ethereum（eth/backend.go）
   ![image](https://github.com/user-attachments/assets/8b070fc0-4f6d-4326-aefe-c65b2c6d72ea)
   Node（node/node.go）
   ![image](https://github.com/user-attachments/assets/942bd892-15aa-48af-ab4b-2f50d88c531a)
4.Geth 节点启动流程
  1. 初始化阶段（核心步骤）
     加载配置：创建Node实例，初始化 RPC 服务、账户管理、P2P 网络
     初始化 Ethereum：
        创建ethdb数据库，实例化共识引擎（验证共识层结果），初始化区块链（core.BlockChain）和交易池。
     注册协议与 API：
        子协议（eth/68、snap），Engine API（执行层与共识层通信）
   2. 启动阶段
      
   ![image](https://github.com/user-attachments/assets/5b47c9df-d724-4bef-915a-1b4df2674811)

五、执行层核心功能总结
  1. 交易执行机制
     唯一通过 EVM 修改状态的方式
     最佳实践：遵循CEI 模式（检查 - 效果 - 交互）防止重入攻击
  2. 状态存储方案
     MPT 树：高效存储账户 / 合约数据，支持快速哈希验证
     分层存储：ethdb抽象底层数据库，statedb管理 MPT 树状态
  3. 网络通信架构
     devp2p 协议：
       节点发现：基于 Kademlia 算法的分布式哈希表
       数据传输：支持eth/68（核心协议）和snap（快速同步）
     去中心化同步：新节点通过 P2P 网络从多个节点获取区块 / 状态




<!-- Content_END -->
