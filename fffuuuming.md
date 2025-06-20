---
timezone: UTC+8
---

> 请在上边的 timezone 添加你的当地时区(UTC)，这会有助于你的打卡状态的自动化更新，如果没有添加，默认为北京时间 UTC+8 时区


# fffuuuming

1. 自我介绍：資工碩班，想踏入web3
2. 你认为你会完成本次残酷学习吗？盡力
3. 你的联系方式（推荐 Telegram）@fffuuuming

[HACKMD LINK](https://hackmd.io/@BKuxYlL0T6iT9rROj2QldQ/H1zMMB3Qll)
###### tags:`intensive-colearning`
## Notes

<!-- Content_START -->
### 2025.06.16
[Ethereum State Trie Architecture Explained](https://medium.com/@eiki1212/ethereum-state-trie-architecture-explained-a30237009d4e)
---
- **World State Trie (Global State Trie)**: A global state constantly updated by transaction executions
    - Stores the state of all Ethereum accounts, including `EOA` & contracts
    - Each account is stored as a key-value pair: 
        - **Key** = `Keccak-256(address)`
        - **Value** = RLP-encoded account object below
            ```
            [
              nonce,
              balance,
              storageRoot,  <- pointer to the account's storage trie
              codeHash
            ]
            ```
- **Account Storage Trie**: Store contract’s key-value storage (i.e., its state variables)
    - **Key** = `Keccak-256(storage slot)`
	- **Value** = 32-byte data stored at that slot

Relationship between World State Trie  and Account Storage Trie:
```
World State Trie (global)
│
├── Keccak(addr_1) → [nonce, balance, ⛔, ⛔]            (EOA)
│
├── Keccak(addr_2) → [nonce, balance, root_2, codeHash] (Contract)
│                         │
│                         └── Account Storage Trie
│                             ├── Keccak(slot_0) → value_0
│                             ├── Keccak(slot_1) → value_1
│                             └── ...
│
├── ...
└── Keccak(addr_n) → [nonce, balance, root_n, codeHash]
```
- **Transaction Trie**: records transactions in Ethereum
- **Transaction Receipt Trie(Receipt Trie)**: 
#### More Reference
- [Modified Merkle Patricia Trie — How Ethereum saves a state](https://medium.com/codechain/modified-merkle-patricia-trie-how-ethereum-saves-a-state-e6d7555078dd)
### 2025.06.17
[Consensus Layer Overview](https://epf.wiki/#/wiki/CL/overview?id=consensus-layer-overview)
---
**Goal**: 
Achieve reliable coordination of distributed Ethereum nodes over unreliable infrastructure (internet, misconfigured software, malicious actors...), where each node maintains a copy of Ethereum’s state; while consensus ensures all match exactly and update in the same order

**Byzantine Fault Tolerance (BFT)**:
Ability of a distributed system to function correctly even if some nodes behave arbitrarily or maliciously (Byzantine faults), which is essential for decentralized networks where trust among nodes is not assumed.

**Consensus protocol**
- **PoW / PoS**: not a protocol themselves but a **Sybil resistance mechanisms** (placing a cost on participating in the protocol, prevents attackers from overwhelming the system cheaply) that enable consensus protocols
    - PoW → weight = total computational work
    - PoS → weight = total stake supporting the chain

**Ethereum’s Consensus Protocol: Gasper**
Combination of
- LMD GHOST: fork choice rule (decides the head of the chain)
- Casper FFG: finality gadget (helps finalize blocks)

Block payload:
- Execution Layer: list of transactions.
- Pre-Merge Beacon Chain: attestations only.
- Post-Merge: includes execution payload and attestations.
- Post-EIP-4844 (Deneb): includes data blob commitments.
---
**The Merge (Paris Hard Fork)**: The day Ethereum switched to **PoS**

Ethereum was originally **PoW**, but for preparation for **PoS**, it launched a new parallel chain called **Beacon Chain** for running PoS logic only — **no user transactions**. After the Merge, Beacon Chain became the main engine of Ethereum’s consensus (**Consensus Layer**), while the original Ethereum chain became the **Execution Layer**

**Terminal Total Difficulty (TTD)**:

---

#### Beacon Chain Introduction
- **Validators**: responsible for **block proposal** and **attestation**
    - `32 ETH` per validator to participate
        - Each validator is run by a validator client, which interacts with the beacon node that tracks the Beacon Chain.
    - Can be either proposer or one of committee
    - Attestations help finalize blocks and reach agreement on the chain head
    - Rewarded for honesty and penalized for faults or malicious behavior (e.g., slashing).
- **Committees**: A group of ≥128 validators per slot at least
    - Randomly assigned for security
    - Committees attest to block proposals and help finalize chain state
- **Attestations**: validator’s vote on the chain state (specifically, the head block).
    - Votes are weighted by validators' stake and broadcasted in addition to blocks, recorded in the chain
    - Follow the **LMD-GHOST** fork choice rule to determine the head of the Beacon Chain
- **Slot**: A chance for a block to be added to the Beacon Chain every 30 seconds. Can be empty. Each slot is assigned with
    - 1 block proposer
    - Committees of validators for attestation
![截圖 2025-06-17 晚上8.36.29](https://hackmd.io/_uploads/S1MDZk1Nee.png)
- **Epoch**: 32 slots → 6.4 minutes (384 seconds).
![截圖 2025-06-17 晚上8.35.36](https://hackmd.io/_uploads/rk4EbyJNeg.png)
- **Randomness**: Beacon Chain emits publicly verifiable randomness like a “randomness beacon"
    - **[RANDAO](https://inevitableeth.com/home/ethereum/network/consensus/randao)** (Randomness DAO) + **[VDF](https://inevitableeth.com/home/ethereum/upgrades/consensus-updates/vdf)** (Verifiable Delay Function)
    - At every epoch, a pseudorandom process **RANDAO** selects proposers for each slot, and shuffles validators to committees.
![截圖 2025-06-17 晚上8.34.18](https://hackmd.io/_uploads/B1DZ-yJVgl.png)

Example:
![截圖 2025-06-17 晚上8.38.38](https://hackmd.io/_uploads/BkhR-JkVee.png)
- `Slot 1`: Block proposed, 2 validators attest, 1 is offline.
- `Slot 2`: Block proposed, 1 validator attests to previous block (missed latest).
- `Slot 3`: All committee validators agree on the same head (via LMD-GHOST).

The Beacon Chain maintains:
- A validator registry
- Validator states (active, exited, slashed, etc.)
- Attestations (votes)
### 2025.06.18
[Consensus Layer Overview](https://epf.wiki/#/wiki/CL/overview?id=consensus-layer-overview)
---
#### [EIP-4844](https://eips.ethereum.org/EIPS/eip-4844) (Proto-Danksharding)
Introduces a data availability layer to Ethereum, allowing for the **temporary storage** of arbitrary data on the blockchain: **`Blob`**, which provide cheap data availability for L2s like rollups (reducing gas costs).
- **Retention**: Each `blob` is Retained for 4096 epochs (~18 days), new blocks keep adding `blobs`, while older ones are deleted.
- **Storage Impact**: Nodes need to prepare extra disk usage for 
    ```
    131,928 bytes/blob * 4096 epochs * 32 blocks/epoch * 3~6 blobs/block 
    ≈ 52 ~ 104 GB total temporary storage needed
    ```
- **Cryptographic Integrity**
    - Each blob is committed via **[KZG commitments](https://epf.wiki/#/wiki/Cryptography/kzg)** for **integrity verification** & Compatibility with **proposer-builder separation**
    - Needs a trusted setup, done via the **[KZG Ceremony](https://github.com/ethereum/kzg-ceremony)**
#### Checkpoints (EBB): Used for finalizing the blockchain.
- One per epoch: block at the **first slot of an epoch**, or the **latest prior block** if empty
- A block becomes a checkpoint if it **receives attestations from a majority of validators**
- A block is considered **final** when it **is included in two-thirds of the most recent checkpoint attestations**, ensuring it cannot be reverted
![截圖 2025-06-18 下午2.00.03](https://hackmd.io/_uploads/S1gPLR1Vgl.png)
$\mathrm{EBB(n, k)}$: the j-th epoch boundary block of B, to be the block with the highest slot less than or equal to jC in chain(B)
Epoch Boundary Block at slot number n, serving as the checkpoint of epoch k, so $\mathrm{EBB(n, k)}$ = `slot 180`

Validators cast two types of votes:
- **LMD GHOST votes** for `blocks`
- **Casper FFG votes** for `checkpoints`: includeing a source checkpoint from a previous epoch and a target checkpoint from the current epoch. (e.g. a validator in Epoch 1 might vote for a source checkpoint at the genesis block and a target checkpoint at Slot 64)

Note that only validators assigned to a slot cast LMD GHOST votes, while all validators cast FFG votes for epoch checkpoints.
#### Justification & Finality
- **Supermajority**: ⅔ of the total validator balance
- **justified**: Once a checkpoint receives a supermajority
- **finalized**: the subsequent epoch's checkpoint also achieves justification
### 2025.06.19
[Execution Layer Specification](https://epf.wiki/#/wiki/EL/el-specs?id=execution-layer-specification)
---
EL focus on executing the **state transition function** (STF) with two questions:
- Is it possible to append the block to the end of the blockchain?
- How does the state change as a result?

Simplified Overview:
![截圖 2025-06-15 晚上9.30.42](https://hackmd.io/_uploads/SkaMsH2mee.png)
$$\sigma_{t+1} \equiv \Pi(\sigma_t, B)$$
- $B$ : **current [block](https://github.com/ethereum/execution-specs/blob/0f9e4345b60d36c23fffaa69f70cf9cdb975f4ba/src/ethereum/shanghai/fork_types.py#L217)** that is being sent to the execution layer for processing.
- $\sigma_t$ , $\sigma_{t+1}$: **state of the [blockchain](https://github.com/ethereum/execution-specs/blob/0f9e4345b60d36c23fffaa69f70cf9cdb975f4ba/src/ethereum/shanghai/fork.py#L73)** before / after applying the current block
- $\Pi$ : [block level state transition function](https://github.com/ethereum/execution-specs/blob/0f9e4345b60d36c23fffaa69f70cf9cdb975f4ba/src/ethereum/shanghai/fork.py#L145)

前情提要：**collapse function**: an operation that reduces world state into a single hash, to store in the block header
- Takes a set of account/storage entries.
- Builds a Merkle Patricia Trie (MPT)
- Computes the root hash, e.g., `stateRoot`

預計之後會先轉往 Geth 的架構及實現閱讀
### 2025.06.20
[Geth 源码系列 I：Geth 整体架构](https://forum.lxdao.io/t/geth-i-geth/2856)
---
#### Introduction: 
以太坊其實是一个交易驱动的状态机，並分成兩層，這兩層間以 API (Engine API) 進行通訊
- **Consensus layer**: (CL) 负责**驱动执行层运行**，包括**让执行层产出区块、决定交易的顺序、为区块投票、以及让区块获得最终性**
- **Execution layer** (EL): 不负责决定交易的顺序，只负责
    - **執行交易**
    - **状态变更**（记录交易执行之后的状态变化）
        - 以区块的方式将所有的状态变化都记录下来
        - 在数据库中记录当前的状态
    - 當作交易的入口，通过交易池来存储还没有被打包进区块的交易


由于这个状态机是去中心化的，所以需要通过 p2p 网络与其他的节点通信，(ex: 如果其他的节点需要获取當前節點区块、状态和交易数据，执行层就会通过 p2p 网络将这些信息发送出去) 共同维护状态数据的一致性。

接下來的探討主要聚焦在执行层，有三个核心模块：
- **计算** (EVM implementation)
- **存储** (ethdb implementation)
- **网络** (devp2p implementation)

#### EL Overview
执行层从逻辑上可以分为 6 个部分：
- **EVM**：相當於以太坊的状态转换函数，负责执行交易，交易执行也是修改状态数的唯一方式
    - 交易由用户（或者程序）按照以太坊执行层规范定义的格式生成，用户需要**对交易进行签名**，如果交易是合法的（`Nonce` 连续、签名正确、`gas fee`足够、业务逻辑正确），那么交易最终就会被 EVM 执行，从而更新以太坊网络的状态。这里的状态是指数据结构、数据和数据库的集合，包括外部账户地址、合约地址、地址余额以及代码和数据。
    - 函数的输入会来源于多个地方，有可能来源于共识层提供的最新区块信息，也有可能来源于 p2p 网络下载的区块。
- **存储**：负责 `state` 以及 `block` 等数据的存储
- **交易池**：用于用户提交的交易，暂时存储，并且会通过 p2p 网络在不同节点之间传播
- **p2p 网络**：用于发现节点、同步交易、下载区块等等功能
- **RPC 服务**：提供访问节点的能力，比如用户向节点发送交易，共识层和执行层之间的交互
- **BlockChain**：负责管理以太坊的区块链数据

![f0279126ef14fc319b2d92e35f8de6a7381f1073_2_1380x774](https://hackmd.io/_uploads/Hy_zYLz4ll.png)

EX:
- p2p + rpc: 共识层和执行层通过 Engine API 来进行通信
    - 如果共识层拿到了出块权，就会通过 Engine API 让执行层产出新的区块
    - 如果没有拿到出块权，就会同步最新的区块让执行层验证和执行，从而与整个以太坊网络保持共识。
#### 源码结构
重點關注 core、eth、ethdb、node、p2p、rlp、trie & triedb 等模块：
- core：区块链核心逻辑，处理区块/交易的生命周期管理、状态机、Gas计算等
- eth：以太坊网络协议的完整实现，包括节点服务、区块同步（如快速同步、归档模式）、交易广播等
- ethdb：数据库抽象层，支持 LevelDB、Pebble、内存数据库等，存储区块链数据（区块、状态、交易）
- node：节点服务管理，整合 p2p、RPC、数据库等模块的启动与配置
- p2p：点对点网络协议实现，支持节点发现、数据传输、加密通信
- rlp：实现以太坊专用的数据序列化协议 RLP（Recursive Length Prefix），用于编码/解码区块、交易等数据结构
- trie & triedb：默克尔帕特里夏树（Merkle Patricia Trie）的实现，用于高效存储和管理账户状态、合约存储

下面為個人認為值得一看的：
- console：提供交互式 JavaScript 控制台，允许用户通过命令行直接与以太坊节点交互（如调用 Web3 API、管理账户、查询区块链数据）
- ethclient：实现以太坊客户端库，封装 JSON-RPC 接口，供 Go 开发者与以太坊节点交互（如查询区块、发送交易、部署合约）
- rpc：实现 JSON-RPC 和 IPC 接口，供外部程序与节点交互
- signer：交易签名管理（硬件钱包集成）
#### 执行层模块划分
![ae1f53f220db394aab7d2304bda57a6634b57bb4_2_1380x738](https://hackmd.io/_uploads/S1gd8JKGNgg.png)
1. 外層 API: 外部访问节点的各项能力
    - Engine API: CL <-> EL
    - Eth API: 外部用户或者程序发送交易，获取区块信息
    - Net API: 获取 p2p 网络的状态等等
2. 中層: 核心功能 implementation
    - tx pool
    - 交易打包
    - 產出 block
    - state, block sync
3. 底層：p2p, ethdb, evm
    - 交易池、区块和状态的同步 ->  依赖 **p2p 网络**
    - 区块的产生以及从其他节点同步过来的区块需要被**验证**才能**写入到本地的数据库** -> 依赖 **EVM** 和**数据存储**的能力。

- **Ethereum**
在 `eth/backend.go` 中的 Ethereum struct 是整个以太坊协议的抽象，基本包括了以太坊中的主要组件，但 `EVM` 是一个例外，它会在每次处理交易的时候实例化，不需要随着整个节点初始化，下文中的 Ethereum 都是指这个结构体：
- **Node**
在 `node/node.go` 中的 `Node` 是另一个核心的数据结构，它作为一个容器，负责**管理和协调各种服务的运行**。在下面的结构中，**Lifecycle** 用来管理内部功能的生命周期。比如上面的 Ethereum 抽象就需要依赖 Node 才能启动，并且在 lifecycles 中注册。这样可以将具体的功能与节点的抽象分离，提升整个架构的扩展性

以下為構成以太坊的核心三組件：
1. **devp2p: 網路**
主要有兩個功能：**节点发现**和**数据传输服务**
    - 在 `p2p/enode/node.go` 中的 `Node` struct 代表了 p2p 网络中一个节点，其中 `enr.Record` 中存储了节点详细信息的键值对 [eip-778](https://eips.ethereum.org/EIPS/eip-778)
2. **ethdb: 存儲**
    - 透過提供統一的 interface 完成以太坊数据存储的 abstraction，底层具体的数据库可自行選擇 (e.g. `leveldb` or `pebble`)
    - 有些数据（e.g. `block` data）可以通过 ethdb 接口直接对底层数据库进行读写，其他的数据存储接口都是建立的 ethdb 的基础上
    - ex: 状态数据会被组织成 `MPT` 结构，在 Geth 中对应的实现是 `trie`，在节点运行的过程中，`trie` 数据会产生很多中间状态，这些数据不能直接调用 `ethdb` 进行读写，需要 **`triedb`** 来管理这些数据和中间状态，最后才通过 `ethdb` 来持久化。
3. **EVM**: 計算
    - `core/vm/evm.go` 中的 EVM 结构体定义了 `EVM` 的总体结构及依赖，包括执行上下文，状态数据库依赖等等
    - `core/vm/interpreter.go` 中的 `EVMInterpreter` struct 定义了解释器的实现，负责执行 EVM 字节码
    - `core/vm/contract.go` 中的 `Contract` struct 封装合约调用的具体参数，包括调用者、合约代码、输入等等，并且在 core/vm/opcodes.go 中定义了当前所有的操作码：

其他模塊（在这三个核心组件的基础之上构建起来）：

在 `eth/protocols` 下有当前以太坊的p2p网络子协议的实现。有 `eth/68` 和 `snap` 子协议，这个些子协议都是在 devp2p 上构建的。

- `eth/68`: 以太坊的核心协议，在这个协议的基础之上又实现了交易池（TxPool）、区块同步（Downloader）和交易同步（Fetcher）等功能。
- `snap`: 用于新节点加入网络时快速同步区块和状态数据的，可以大大减少新节点启动的时间。
- `ethdb` 提供了底层数据库的读写能力，由于以太坊协议中有很多复杂的数据结构，直接通过 `ethdb` 无法实现这些数据的管理，所以在 `ethdb` 上又实现了 
    - `rawdb`: 管理区块
    - `statedb`: 状态数据。

EVM 则贯穿所有的主流程，无论是区块构建还是区块验证，都需要用 EVM 执行交易。


#### Geth 节点启动流程
1. **节点初始化**: 初始化节点所需要启动的组件和资源
![7919a72816f209e7ff9eebd8a27de508181134e7_2_1380x728](https://hackmd.io/_uploads/H1U74FfEgx.png)
    - 初始化 `node/node.go` 中的 `Node` struct，所有的功能都需要在这个容器中运行
        - 创建了一个 `Node` instance
        - 初始化 p2p server、账号管理以及 http 等暴露给外部的协议端口。
    - 初始化 `Ethereum` struct，包括以太坊各种核心功能的实现，`Etherereum` 也需要注册到 `Node` 中
        - 初始化化 `ethdb`，并从存储中加载链配置
        - 然后创建共识引擎：不会执行共识操作，而只是会对共识层返回的结果进行验证，如果共识层发生了提款请求，也会在这里完成实际的提款操作。
        - 初始化 `Block Chain` struct 和 `tx pool`。
        - 初始化 handler：所有 p2p 网络请求的处理入口
        - 将一些在 devp2p 基础之上实现的子协议，(e.g. `eth/68`、`snap`) 等注册到 `Node` 容器中
        - 最后 `Ethereum` 会作为一个 `lifecycle` 注册到 `Node` 容器中
    - 注册 Engine API 到 `Node` 中。
2. **节点启动**: 将已经注册的 `RPC` 服务和 `Lifecycle` 全部启动，整个节点即可向外部提供服务。

<!-- Content_END -->
