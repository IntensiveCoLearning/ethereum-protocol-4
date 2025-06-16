---
timezone: UTC+8
---

> 请在上边的 timezone 添加你的当地时区(UTC)，这会有助于你的打卡状态的自动化更新，如果没有添加，默认为北京时间 UTC+8 时区


# Echo

1. 自我介绍
    菜鸟级 dapp 开发选手，希望通过这次 EPF 的学习能够加深对以太坊的了解
2. 你认为你会完成本次残酷学习吗？
    会
3. 你的联系方式（推荐 Telegram）
    @Echo_2333


<!-- Content_START -->

## 2025.06.16

### 一、前置信息介绍
1. 以太坊 The Merge（合并）是以太坊历史上最重要的升级之一，于 2022 年 9 月 15 日成功完成。它的核心内容是将以太坊主网（执行层，原本基于工作量证明 PoW）与信标链（共识层，基于权益证明 PoS）合并，正式将以太坊的共识机制从 PoW 切换为 PoS。
    - **简单流程：**
        - 信标链（Beacon Chain）于 2020 年 12 月上线，作为 PoS 共识层并行运行。
        - 2022 年 9 月 15 日，主网与信标链合并，PoW 被永久关闭，PoS 成为唯一共识机制。
2. Geth 属于执行层客户端，负责交易的执行、状态和数据的维护，使用 Go 语言开发，是公认的最稳定、久经考验的客户端

### 二、Geth 框架的整理

Geth 从逻辑上可以分为 6 个部分：

- EVM：负责执行交易，交易执行也是修改状态数的唯一方式
- 存储：负责 state 以及区块等数据的存储
- 交易池：用于用户提交的交易，暂时存储，并且会通过 p2p 网络在不同节点之间传播
- p2p 网络：用于发现节点、同步交易、下载区块等等功能
- RPC 服务：提供访问节点的能力，比如用户向节点发送交易，共识层和执行层之间的交互
- BlockChain：负责管理以太坊的区块链数据

[![2025-06-16-18-31-01.jpg](https://i.postimg.cc/hvWk5md7/2025-06-16-18-31-01.jpg)](https://postimg.cc/G8KgBHzc)

- 如果以一个抽象的维度来看以太坊的执行层，以太坊作为一台世界计算机，需要包括三个部分，网络、计算和存储，那么以太坊执行层中与这三个部分相对应的组件是：
    - 网络：devp2p
        - 以太坊本质还是一个分布式系统，每个节点通过 p2p 网络与其他节点相连。以太坊中的 p2p 网络协议的实现就是 devp2p。
        - devp2p 有两个核心功能，一个是节点发现，让节点在接入网络时能够与其他节点建立联系；另一个是数据传输服务，在与其他节点建立联系之后，就可以想换交换数据。
    - 计算：EVM
    - 存储：ethdb

#### Geth 完整模块：
1. **账户与加密相关**
    - **accounts**：管理以太坊账户，包括公私钥对的生成、签名验证、地址派生等
    - **crypto**：加密算法（椭圆曲线、哈希、签名验证）
    - **signer**：交易签名管理（含硬件钱包）

2. **共识与信标链**
    - **beacon**：处理与以太坊信标链（Beacon Chain）的交互逻辑，PoS共识相关
    - **consensus**：共识引擎（PoW、PoS、Clique 等）
    - **miner**：挖矿逻辑（PoW 场景）

3. **区块链核心与数据结构**
    - **core**：区块链核心逻辑，处理区块/交易生命周期、状态机、Gas 计算
    - **trie & triedb**：默克尔帕特里夏树，实现账户/合约状态存储
    - **ethdb**：数据库抽象层（LevelDB、内存等）
    - **rlp**：RLP 序列化协议实现

4. **网络与节点服务**
    - **eth**：以太坊协议实现，节点服务、区块同步、交易广播
    - **p2p**：点对点网络协议，节点发现、数据传输
    - **node**：节点服务管理，整合各模块
    - **ethstats**：节点状态上报与监控
    - **event**：事件订阅与发布机制

5. **接口与交互**
    - **cmd**：命令行工具入口
    - **console**：交互式 JavaScript 控制台
    - **rpc**：JSON-RPC 和 IPC 接口
    - **graphql**：GraphQL 接口
    - **ethclient**：Go 客户端库，封装 RPC 接口
    - **RPC服务**（见 rpc/ethclient/console 等）

6. **工具与通用组件**
    - **common**：通用工具类（字节处理、地址转换、数学函数等）
    - **params**：网络参数（主网、测试网、创世块等）
    - **log**：日志系统
    - **metrics**：性能指标收集（Prometheus 支持）
    - **internal**：内部工具/限制外部访问代码

7. **构建、文档与测试**
    - **build**：构建脚本、编译配置
    - **docs**：文档
    - **tests**：集成测试、状态测试

所以本次共学主要围绕 `core`、`eth`、`ethdb`、`node`、`p2p`、`rlp`、`trie` & `triedb` 部分进行源码学习


### 三、Geth 启动

Geth 教程：[https://geth.ethereum.org/docs/getting-started](https://geth.ethereum.org/docs/getting-started) （未维护？很多页面都无法访问）

- 在 Geth 中创建账户的方法有很多种。在本次学习中尝试使用了 Clef 来创建账户，它将用户的密钥管理与 Geth 解耦，使其更加模块化和灵活。
    ``` bash
    # 老版本：
    geth --datadir . account new

    # 新版本：
    clef newaccount --keystore geth-tutorial/keystore
    ```
- 在 geth 启动的过程中 Clef 终端应保持运行。
    ``` bash
    clef --keystore geth-tutorial/keystore --configdir geth-tutorial/clef --chainid 11155111

    # --keystore 私钥存储的位置： geth-tutorial/keystore 
    # --chainid 链 ID： 11155111 为 Sepolia 测试网的链 ID
    ```
- 在 geth 终端做账户交互时需要 Clef 终端进行操作。
    ``` bash
    # geth 终端
    > eth.accounts;

    # clef 终端
    -------- List Account request--------------
    A request has been made to list all accounts. 
    You can select which accounts the caller can see
        [x] 0x12B275ea188d8B2E368880c15A3d7d60563aF811
            URL: keystore:///Users/apple/Desktop/ethereum-protocol/geth/geth-tutorial/keystore/UTC--2025-06-16T07-42-55.120127000Z--12b275ea188d8b2e368880c15a3d7d60563af811
    -------------------------------------------
    Request context:
            NA -> ipc -> NA

    Additional HTTP header data, provided by the external caller:
            User-Agent: ""
            Origin: ""
    Approve? [y/N]:
    > y

    # geth 终端
    > ["0x12b275ea188d8b2e368880c15a3d7d60563af811"]
    ```
- 如果 Clef 审批时间过长，此请求也可能会超时
## 2025.06.17

<!-- Content_END -->
