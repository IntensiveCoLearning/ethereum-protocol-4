---
timezone: UTC+8
---


# kuove

1. 以太坊初学者
2. 你认为你会完成本次残酷学习吗？可以
3. Telegram：@stephkuove

## Notes

<!-- Content_START -->

### 2025.06.16

#### 单时隙最终确定性（Single Slot Finalty）及质押民主化
目前，需要 2-3 个 epoch（15 分钟左右）才能最终确定（finalize）一个区块，且需要 32 ETH 才能成为质押者。
最初这是为了在以下三个目标之间取得平衡而做出的妥协：

    1、最大化参与到质押中的验证者数量

    2、最小化最终确定时间

    3、最小化运行节点的开销，包括下载、验证和广播其他验证者签名的成本
这三个目标是相互冲突的

希望在理想情况下，保持经济终结性，同时在两个方面改善现状：

    1、在单时隙内最终确定区块（理想情况下，保持甚至减少当前 12 秒的时长），而不是 15 分钟

    2、允许验证者以 1 ETH 进行质押（原为 32 ETH）

存在的挑战
    更快的区块最终确定和更民主化质押的目标，但这都与最小化开销的目标相冲突。

如何做到
单时隙最终性涉及使用一种共识算法来最终确定一个时隙中的区块。
以太坊的独特特性需求是 inactivity leaks：即使超过 1/3 的验证器离线，该特性也允许链继续运行并最终恢复，
已经有提案修改了 Tendermint 共识以适应 inactivity leaks。

问题最难的部分是在验证者数量非常多时，单时隙的最终确定性如何正常工作，而不会导致节点的运营商产生极高的开销。
为此，有几种领先的解决方案
    暴力破解
    Orbit 委员会 —— 一种新机制，允许随机选取的中型委员会来负责链的最终确定性，但这是以一种我们想要的保留攻击成本特性的方式。
    双层质押 —— 一种有两类质押者的机制，一类质押者有较高的存款要求，另一类质押者有较低的存款要求。

还有什么要做，需要权衡什么？
有四种主要的可能路径可供选择（我们也可以采取混合路径）：

    1、维持现状
    2、暴力破解 SSF
    3、Orbit SSF
    4、有双层质押机制的 SSF

可以组合多种策略，例如：

（1+2）：添加 Orbit，但不进行单时隙最终确定。
（1+3）：使用强力技术减少最小存款规模，而无需进行单时隙最终确定。所需的聚合量比单纯使用（3）情况少 64 倍，这样一来问题变得更简单了。
（2+3）：使用保守参数（例如 128k 验证者委员会而不是 8k 或 32k）执行 Orbit SSF，并使用暴力破解技术使其极其高效。
（1+4）：增加彩虹质押而不执行单时隙最终确定

### 2025.06.17
#### 单一秘密领袖选举

哪个验证者将提出下一个区块是可以预知。会产生一个安全漏洞：攻击者可以进行 DoS 攻击。

如何做到

隐藏哪个验证者将生成下一个区块的信息，至少在区块实际生成之前要隐藏这些信息。
如果删除“单独”这一要求，一种解决方案是让任何人都可以创建下一个区块，但要求 randao 揭示小于 2^256 / N。平均而言，只有一个验证者能够满足此要求 —— 但有时会有两个或更多，有时会没有。

单一秘密领袖选举协议通过使用一些加密技术为每个验证者创建一个“盲”验证者 ID 来解决这个问题，然后让许多提议者有机会对盲 ID 池进行改组和重新盲化（这类似于混合网络的工作方式）。在每个时段，都会选择一个随机盲 ID。只有该盲 ID 的所有者才能生成有效的证明来提议区块，且没有人知道该盲 ID 对应的是哪个验证者。


#### 与现有研究有哪些联系？

Dan Boneh 的论文（Paper by Dan Boneh）（2020）：https://eprint.iacr.org/2020/025.pdf

Whisk（以太坊具体提案，2022）：https://ethresear.ch/t/whisk-a-practical-shuffle-based-ssle-protocol-for-ethereum/11763

在 ethresear.ch 的单一秘密领袖选举标签（Single secret leader election tag on ethresear.ch）：https://ethresear.ch/tag/single-secret-leader-election

使用环签名的简化 SSLE（Simplified SSLE using ring signatures）：https://ethresear.ch/t/simplified-ssle/12315


#### 还有什么要做，需要权衡什么？
需要找到并实现一个足够简单的协议，以便轻松地在主网上实现。我们高度重视保持以太坊协议的简易性，不希望进一步增加复杂性。
SSLE 仅用了数百行规范代码，并在复杂的加密中引入了新的假设。如何实现足够有效的抗量子 SSLE 也是一个问题。

最终可能会出现这样的情况：
只有当我们出于其他原因（例如状态树、ZK-EVM）大胆尝试并在 L1 的以太坊协议中引入执行通用零知识证明的机制时，SSLE 的“边际额外复杂性”才会下降到足够低的水平。

另一种选择是根本不理会 SSLE，而是使用协议外缓解措施（例如在 p2p 层）来解决 DoS 问题。

它如何与路线图的其他部分互动？

如果我们添加证明者-提议者分离（APS）机制，比如执行票证，因为我们可以依赖专门的区块构建器，那么执行区块（例如包含以太坊交易的区块）将不再需要 SSLE。不过，对于共识区块（包含协议消息的区块：比如证明、可能还有纳入列表的一些片段等），我们仍然可以从 SSLE 中受益。

#### 更快的交易确认

进一步缩短以太坊的交易确认时间（从 12 秒缩短到 4 秒）。
这样做将显著改善 L1 和基于 rollups 的用户体验，同时使 De-Fi 协议更加高效。
它还将使 L2 更易去中心化，因为它将允许大量 L2 应用程序在 based rollups 上运作，从而减少 L2 构建自己的基于委员会的去中心化排序的需求。

这里大致有两种技术：

减少时隙时间，降至如 8 秒或 4 秒。这并不一定意味着 4 秒的最终确定性：最终确定性本质上需要三轮通信，因此我们可以将每轮通信设为一个单独的区块，这将在 4 秒后至少得到初步确认。

允许提议者在单时隙期间发布预确认。在极端情况下，提议者可以实时将他们看到的交易纳入其区块，并立即为每笔交易发布预确认消息。
提议者发布两个相互冲突的确认的情况可以通过两种方式处理：
    （i）通过罚没（slashing）提议者
    （ii）通过使用证明人（attesters）投票选出哪一个更早。

与现有研究有？


### 2025.06.18

#### 执行层
Geth：由以太坊基金会直接资助的团队维护，使用 Go 语言开发，是公认的最稳定、久经考验的客户端
Nethermind：由 Nethermind 团队开发和维护，使用 C# 语言开发，早期获以太坊基金会和 Gitcoin 社区资助
Besu：最初由 ConsenSys 的 PegaSys 团队开发，现为 Hyperledger 社区项目，使用 Java 语言开发
Erigon：由 Erigon 团队开发和维护，获以太坊基金会、BNB Chain 资助。2017 年从 Geth 分叉而来，目标是提升同步速度和磁盘效率
Reth：由 Paradigm 主导开发，开发语言是 Rust，强调模块化和高性能，目前已经趋近成熟，可以在生产环境使用

#### 执行层简介
可以将以太坊执行层看作是一个由交易驱动的状态机，执行层最基础的职能就是通过 EVM 执行交易来更新状态数据。
除了交易执行之外，还有保存并验证区块和状态数据，运行 p2p 网络并维护交易池等功能。

交易由用户（或者程序）按照以太坊执行层规范定义的格式生成，用户需要对交易进行签名，如果交易是合法的（Nonce 连续、签名正确、gas fee 足够、业务逻辑正确），那么交易最终就会被 EVM 执行，从而更新以太坊网络的状态。这里的状态是指数据结构、数据和数据库的集合，包括外部账户地址、合约地址、地址余额以及代码和数据。

执行层负责执行交易以及维护交易执行之后的状态，共识层负责选择哪些交易来执行。EVM 则是这个状态机中的状态转换函数，函数的输入会来源于多个地方，有可能来源于共识层提供的最新区块信息，也有可能来源于 p2p 网络下载的区块。

共识层和执行层通过 Engine API 来进行通信，这是执行层和共识层之间唯一的通信方式。如果共识层拿到了出块权，就会通过 Engine API 让执行层产出新的区块，如果没有拿到出块权，就会同步最新的区块让执行层验证和执行，从而与整个以太坊网络保持共识。

执行层从逻辑上可以分为 6 个部分：

EVM：负责执行交易，交易执行也是修改状态数的唯一方式
存储：负责 state 以及区块等数据的存储
交易池：用于用户提交的交易，暂时存储，并且会通过 p2p 网络在不同节点之间传播
p2p 网络：用于发现节点、同步交易、下载区块等等功能
RPC 服务：提供访问节点的能力，比如用户向节点发送交易，共识层和执行层之间的交互
BlockChain：负责管理以太坊的区块链数据


![执行层的关键流程，以及每个部分的职能](https://forum.lxdao.io/uploads/default/original/2X/f/f0279126ef14fc319b2d92e35f8de6a7381f1073.png)
对于执行层，有三个关键流程：

如果是新加入以太坊的节点，需要通过 p2p 网络从其他的节点同步区块和状态数据，如果是 Full Sync，会从创世区块开始逐个下载区块，验证区块并通过 EVM 重建状态数据库，如果是 Snap Sync，则跳过全部区块验证的过程，直接下载最新 checkpoint 的状态数据和以后的区块数据
如果是已经同步到最新状态的节点，那么就会持续通过 Engine API 从共识层获取到当前最新产出的区块，并验证区块，然后通过 EVM 执行区块中所有的交易来更新状态数据库，并将区块写入本地链
如果是已经同步到最新状态，并且共识层拿到了出块权的节点，就会通过 Engine API 驱动执行层产出最新的区块，执行层从交易池获取交易并执行，然后组装成区块通过 Engine API 传递给共识层，由共识层将区块广播到共识层 p2p 网络


#### go-ethereumdede的代码结构
go-ethereum 的代码结构很庞大，但其中很多代码属于辅助代码和单元测试，在研究 Geth 源码时，只需要关注协议的核心实现，各个模块功能如下。
需要重点关注 core、eth、ethdb、node、p2p、rlp、trie & triedb 等模块：

    accounts：管理以太坊账户，包括公私钥对的生成、签名验证、地址派生等
    beacon：处理与以太坊信标链（Beacon Chain）的交互逻辑，支持权益证明（PoS）共识的合并（The Merge）后功能
    build：构建脚本和编译配置（如 Dockerfile、跨平台编译支持）
    cmd：命令行工具入口，包含多个子命令
    common：通用工具类，如字节处理、地址格式转换、数学函数
    consensus：定义consensus engine ，包括之前的工作量证明（Ethash）和单机权益证明（Clique）以及 Beacon engine 等
    console：提供交互式 JavaScript 控制台，允许用户通过命令行直接与以太坊节点交互（如调用 Web3 API、管理账户、查询区块链数据）
    core：区块链核心逻辑，处理区块/交易的生命周期管理、状态机、Gas计算等
    crypto：加密算法实现，包括椭圆曲线（secp256k1）、哈希（Keccak-256）、签名验证
    docs：文档（如设计规范、API 说明）
    eth：以太坊网络协议的完整实现，包括节点服务、区块同步（如快速同步、归档模式）、交易广播等
    ethclient：实现以太坊客户端库，封装 JSON-RPC 接口，供 Go 开发者与以太坊节点交互（如查询区块、发送交易、部署合约）
    ethdb：数据库抽象层，支持 LevelDB、Pebble、内存数据库等，存储区块链数据（区块、状态、交易）
    ethstats：收集并上报节点运行状态到统计服务，用于监控网络健康状态
    event：实现事件订阅与发布机制，支持节点内部模块间的异步通信（如新区块到达、交易池更新）
    graphql：提供 GraphQL 接口，支持复杂查询（替代部分 JSON-RPC 功能）
    internal：内部工具或限制外部访问的代码
    log：日志系统，支持分级日志输出、上下文日志记录
    mertrics：性能指标收集（Prometheus 支持）
    miner：挖矿相关逻辑，生成新区块并打包交易（PoW 场景下）
    node：节点服务管理，整合 p2p、RPC、数据库等模块的启动与配置
    p2p：点对点网络协议实现，支持节点发现、数据传输、加密通信
    params：定义以太坊网络参数（主网、测试网、创世区块配置）
    rlp：实现以太坊专用的数据序列化协议 RLP（Recursive Length Prefix），用于编码/解码区块、交易等数据结构
    rpc：实现 JSON-RPC 和 IPC 接口，供外部程序与节点交互
    signer：交易签名管理（硬件钱包集成）
    tests：集成测试和状态测试，验证协议兼容性
    trie & triedb：默克尔帕特里夏树（Merkle Patricia Trie）的实现，用于高效存储和管理账户状态、合约存储
### 2025.06.19
#### 执行层模块划分
外部访问 Geth 节点有两种形式，一种是通过 RPC，另外一种是通过 Console。
RPC 适合开放给外部的用户来使用，Console 适合节点的管理者使用。但无论是通过 RPC 还是 Console，都是使用内部已经封装好的能力，这些能力通过分层的方式来构建。

最外层是 API 用于外部访问节点的各项能力，
Engine API 用于执行层和共识层之间的通信，Eth API 用于外部用户或者程序发送交易，获取区块信息，Net API 用于获取 p2p 网络的状态等等。 
比如用户通过 API 发送了一个交易，那么这个交易最终会被提交到交易池中，通过交易池来管理，再比如用户需要获取一个区块数据，那么就需要调用数据库的能力去获取对应的区块。

在 API 的下一层就核心功能的实现，包括交易池、交易打包、产出区块、区块和状态的同步等等。
这些功能再往下就需要依赖更底层的能力，比如交易池、区块和状态的同步需要依赖 p2p 网络的能力，区块的产生以及从其他节点同步过来的区块需要被验证才能写入到本地的数据库，这些就需要依赖 EVM 和数据存储的能力。
![执行层模块划分](https://forum.lxdao.io/uploads/default/original/2X/a/ae1f53f220db394aab7d2304bda57a6634b57bb4.png)

#### 执行层核心数据结构
##### Ethereum
在 eth/backend.go 中的 Ethereum 结构是整个以太坊协议的抽象，基本包括了以太坊中的主要组件，但 EVM 是一个例外，它会在每次处理交易的时候实例化，不需要随着整个节点初始化，下文中的 Ethereum 都是指这个结构体：

```

type Ethereum struct {
// 以太坊配置，包括链配置
    config         *ethconfig.Config
    // 交易池，用户的交易提交之后先到交易池
    txPool         *txpool.TxPool
    // 用于跟踪和管理本地交易（local transactions）
    localTxTracker *locals.TxTracker
    // 区块链结构
    blockchain     *core.BlockChain

// 是以太坊节点的网络层核心组件，负责处理所有与其他节点的通信，包括区块同步、交易广播和接收，以及管理对等节点连接
    handler *handler
    // 负责节点发现和节点源管理
    discmix *enode.FairMix

    // 负责区块链数据的持久化存储
    chainDb ethdb.Database
// 负责处理各种内部事件的发布和订阅
    eventMux       *event.TypeMux
    // 共识引擎
    engine         consensus.Engine
    // 管理用户账户和密钥
    accountManager *accounts.Manager

// 管理日志过滤器和区块过滤器
    filterMaps      *filtermaps.FilterMaps
    // 用于安全关闭 filterMaps 的通道，确保在节点关闭时正确清理资源
    closeFilterMaps chan chan struct{}

// 为 RPC API 提供后端支持
    APIBackend *EthAPIBackend

// 在 PoS 下，与共识引擎协作验证区块
    miner    *miner.Miner
    // 节点接受的最低gas价格
    gasPrice *big.Int
// 网络 ID
    networkID     uint64
    // 提供网络相关的 RPC 服务，允许通过 RPC 查询网络状态
    netRPCService *ethapi.NetAPI

// 管理P2P网络连接，处理节点发现和连接建立并提供底层网络传输功能
    p2pServer *p2p.Server

// 保护可变字段的并发访问
    lock sync.RWMutex 
    // 跟踪节点是否正常关闭，在异常关闭后帮助恢复
    shutdownTracker *shutdowncheck.ShutdownTracker 
}

```

##### Node

在 node/node.go 中的 Node 是另一个核心的数据结构，它作为一个容器，负责管理和协调各种服务的运行。
在下面的结构中，需要关注一下 lifecycles 字段，Lifecycle 用来管理内部功能的生命周期。比如上面的 Ethereum 抽象就需要依赖 Node 才能启动，并且在 lifecycles 中注册。这样可以将具体的功能与节点的抽象分离，提升整个架构的扩展性，这个 Node 需要与 devp2p 中的 Node 区分开。


```
type Node struct {
	eventmux      *event.TypeMux
	config        *Config
	// 账户管理器，负责管理钱包和账户
	accman        *accounts.Manager
	log           log.Logger
	keyDir        string        
	keyDirTemp    bool          
	dirLock       *flock.Flock  
	stop          chan struct{}
	// p2p 网络实例 
	server        *p2p.Server   
	startStopLock sync.Mutex
	// 跟踪节点生命周期状态（初始化、运行中、已关闭）    
	state         int           

	lock          sync.Mutex
	// 所有注册的后端、服务和辅助服务
	lifecycles    []Lifecycle
	// 当前提供的 API 列表 
	rpcAPIs       []rpc.API
	// 为 RPC 提供的不同访问方式   
	http          *httpServer 
	ws            *httpServer 
	httpAuth      *httpServer 
	wsAuth        *httpServer 
	ipc           *ipcServer  
	inprocHandler *rpc.Server 
	databases map[*closeTrackingDB]struct{} 
}
```
### 2025.06.20
如果以一个抽象的维度来看以太坊的执行层，以太坊作为一台世界计算机，需要包括三个部分，网络、计算和存储，那么以太坊执行层中与这三个部分相对应的组件是：

    网络：devp2p
    计算：EVM
    存储：ethdb
#### devp2p
以太坊本质还是一个分布式系统，每个节点通过 p2p 网络与其他节点相连。以太坊中的 p2p 网络协议的实现就是 devp2p。

devp2p 有两个核心功能，
一个是节点发现，让节点在接入网络时能够与其他节点建立联系；
另一个是数据传输服务，在与其他节点建立联系之后，就可以想换交换数据。

在 p2p/enode/node.go 中的 Node 结构代表了 p2p 网络中一个节点，其中 enr.Record 结构中存储了节点详细信息的键值对，包括身份信息（节点身份所使用的签名算法、公钥）、网络信息（IP 地址，端口号）、支持的协议信息（比如支持 eth/68 和 snap 协议）和其他的自定义信息，这些信息通过 RLP 的方式编码，具体的规范在 eip-778 中定义：

```
type Node struct {
    // 节点记录，包含节点的各种属性
    r  enr.Record
    // 节点的唯一标识符，32字节长度  
    id ID          

    // hostname 跟踪节点的DNS名称
    hostname string

    // 节点的IP地址
    ip  netip.Addr
    // UDP端口  
    udp uint16     
    // TCP端口 
    tcp uint16      
}

// enr.Record
type Record struct {
    // 序列号
    seq       uint64       
    // 签名
    signature []byte       
    // RLP 编码后的记录
    raw       []byte       
    // 所有键值对的排序列表
    pairs     []pair       
}
```
在 p2p/discover/table.go 中的 Table 结构是 devp2p 实现节点发现协议的核心数据结构，它实现了类似 Kademlia 的分布式哈希表，用于维护和管理网络中的节点信息。

```
type Table struct {
	mutex        sync.Mutex
	// 按距离索引已知节点        
	buckets      [nBuckets]*bucket
	// 引导节点 
	nursery      []*enode.Node     
	rand         reseedingRandom   
	ips          netutil.DistinctNetSet
	revalidation tableRevalidation
  
  // 已知节点的数据库
	db  *enode.DB 
	net transport
	cfg Config
	log log.Logger

	// 周期性的处理网络中的各种事件
	refreshReq      chan chan struct{}
	revalResponseCh chan revalidationResponse
	addNodeCh       chan addNodeOp
	addNodeHandled  chan bool
	trackRequestCh  chan trackRequestOp
	initDone        chan struct{}
	closeReq        chan struct{}
	closed          chan struct{}

  // 增加和移除节点的接口
	nodeAddedHook   func(*bucket, *tableNode)
	nodeRemovedHook func(*bucket, *tableNode)
}
```

#### ethdb
ethdb 完成以太坊数据存储的抽象，提供统一的存储接口，底层具体的数据库可以是 leveldb，也可以是 pebble 或者其他的数据库。可以有很多的扩展，只要在接口层面保持统一。

有些数据（如区块数据）可以通过 ethdb 接口直接对底层数据库进行读写，其他的数据存储接口都是建立的 ethdb 的基础上，比如数据库有很大部分的数据是状态数据，这些数据会被组织成 MPT 结构，在 Geth 中对应的实现是 trie，在节点运行的过程中，trie 数据会产生很多中间状态，这些数据不能直接调用 ethdb 进行读写，需要 triedb 来管理这些数据和中间状态，最后才通过 ethdb 来持久化。

在 ethdb/database.go 中定义底层数据库的读写能力的接口，但没有包括具体的实现，具体的实现将由不同的数据库自身来实现。比如 leveldb 或者 pebble 数据库。在 Database 中定义了两层数据读写的接口，其中 KeyValueStore 接口用于存储活跃的、可能频繁变化的数据，如最新的区块、状态等。AncientStore 则用于处理历史区块数据，这些数据一旦写入就很少改变。

```
// 数据库的顶层接口
type Database interface {
    KeyValueStore
    AncientStore
}

// KV 数据的读写接口
type KeyValueStore interface {
	KeyValueReader
	KeyValueWriter
	KeyValueStater
	KeyValueRangeDeleter
	Batcher
	Iteratee
	Compacter
	io.Closer
}

// 处理老数据的读写的接口
type AncientStore interface {
	AncientReader
	AncientWriter
	AncientStater
	io.Closer
}
```

#### EVM

EVM 是以太坊这个状态机的状态转换函数，所有状态数据的更新都只能通过 EVM 来进行，p2p 网络可以接受到交易和区块信息，这些信息被 EVM 处理之后会成为状态数据库的一部分。EVM 屏蔽了底层硬件的不同，让程序在不同平台的 EVM 上执行都能得到一致的结果。这是一种很成熟的设计方式，Java 语言中 JVM 也是类似的设计。

EVM 的实现有三个主要的组件，core/vm/evm.go 中的 EVM 结构体定义了 EVM 的总体结构及依赖，包括执行上下文，状态数据库依赖等等； core/vm/interpreter.go 中的 EVMInterpreter 结构体定义了解释器的实现，负责执行 EVM 字节码；core/vm/contract.go 中的 Contract 结构体封装合约调用的具体参数，包括调用者、合约代码、输入等等，并且在 core/vm/opcodes.go 中定义了当前所有的操作码：

```
// EVM
type EVM struct {
    // 区块上下文，包含区块相关信息
    Context BlockContext
    // 交易上下文，包含交易相关信息     
    TxContext
    // 状态数据库，用于访问和修改账户状态               
    StateDB StateDB
    // 当前调用深度         
    depth int
    // 链配置参数               
    chainConfig *params.ChainConfig  
    chainRules params.Rules
    // EVM配置          
    Config Config
    // 字节码解释器                    
    interpreter *EVMInterpreter
    // 中止执行的标志      
    abort atomic.Bool                
    callGasTemp uint64
    // 预编译合约映射               
    precompiles map[common.Address]PrecompiledContract  
    jumpDests map[common.Hash]bitvec  
}

type EVMInterpreter struct {
    // 指向所属的EVM实例
    evm   *EVM
    // 操作码跳转表               
    table *JumpTable         
    // Keccak256哈希器实例，在操作码间共享
    hasher    crypto.KeccakState
    // Keccak256哈希结果缓冲区  
    hasherBuf common.Hash         
    // 是否为只读模式，只读模式下不允许状态修改
    readOnly   bool
    // 上一次CALL的返回数据，用于后续重用          
    returnData []byte        
}

type Contract struct {
    // 调用者地址
    caller  common.Address
    // 合约地址   
    address common.Address   

    jumpdests map[common.Hash]bitvec  
    analysis  bitvec                  
    // 合约字节码
    Code     []byte
    // 代码哈希          
    CodeHash common.Hash
    // 调用输入     
    Input    []byte          
    // 是否为合约部署
    IsDeployment bool
    // 是否为系统调用        
    IsSystemCall bool        
    // 可用gas量
    Gas   uint64
    // 调用附带的 ETH 数量             
    value *uint256.Int       
}
```
### 2025.06.23
#### 其他模块实现
执行层的功能通过分层的方式来实现，其他的模块和功能都是在这三个核心组件的基础之上构建起来的。
以下是几个核心的模块。

在 eth/protocols 下有当前以太坊的p2p网络子协议的实现。有 eth/68 和 snap 子协议，这个些子协议都是在 devp2p 上构建的。

eth/68 是以太坊的核心协议，协议名称就是 eth，68 是它的版本号，然后在这个协议的基础之上又实现了交易池（TxPool）、区块同步（Downloader）和交易同步（Fetcher）等功能。snap 协议用于新节点加入网络时快速同步区块和状态数据的，可以大大减少新节点启动的时间。

ethdb 提供了底层数据库的读写能力，由于以太坊协议中有很多复杂的数据结构，直接通过 ethdb 无法实现这些数据的管理，所以在 ethdb 上又实现了 rawdb 和 statedb 来分别管理区块和状态数据。

EVM 则贯穿所有的主流程，无论是区块构建还是区块验证，都需要用 EVM 执行交易。

#### Geth 节点启动流程
Geth 的启动会分为两个阶段，第一阶段会初始化节点所需要启动的组件和资源，第二节点会正式启动节点，然后对外服务。
![Geth 节点启动流程](https://forum.lxdao.io/uploads/default/original/2X/3/34e938761faadcf5c92e11d9ac4d90d7114c9deb.png)

##### 节点初始化
在启动一个 geth 节点时，会涉及到以下的代码：
![启动geth节点](https://forum.lxdao.io/uploads/default/original/2X/7/7919a72816f209e7ff9eebd8a27de508181134e7.png)

各模块的初始化如下：

    cmd/geth/main.go：geth 节点启动入口
    cmd/geth/config.go（makeFullNode）：加载配置，初始化节点
    node/node.go：初始化以太坊节点的核心容器
    node.rpcstack.go：初始化 RPC 模块
    accounts.manager.go：初始化 accountManager
    eth/backend.go：初始化 Ethereum 实例
    node/node.go OpenDatabaseWithFreezer：初始化 chaindb
    eth/ethconfig/config.go：初始化共识引擎实例（这里的共识引擎并不真正参与共识，只是会验证共识层的结果，以及处理 validator 的提款请求）
    core/blockchain.go：初始化 blockchain
    core/filterMaps.go：初始化 filtermaps
    core/txpool/blobpool/blobpool.go：初始化 blob 交易池
    core/txpool/legacypool/legacypool.go：初始化普通交易池
    cord/txpool/locals/tx_tracker.go：本地交易追踪（需要配置开启本地交易追踪，本地交易会被更高优先级处理）
    eth/handler.go：初始化协议的 Handler 实例
    miner/miner.go：实例化交易打包的模块（原挖矿模块）
    eth/api_backend.go：实例化 RPC 服务
    eth/gasprice/gasprice.go：实例化 gas 价格查询服务
    internal/ethapi/api.go：实例化 P2P 网络 RPC API
    node/node.go(RegisterAPIs)：注册 RPC API
    node/node.go(RegisterProtocols)：注册 p2p 的 Ptotocols
    node/node.go(RegisterLifecycle)：注册各个组件的生命周期
    cmd/utils/flags.go(RegisterFilterAPI)：注册 Filter RPC API
    cmd/utils/flags.go(RegisterGraphQLService)：注册 GraphQL RPC API（如果配置了的话）
    cmd/utils/flags.go(RegisterEthStatsService)：注册 EthStats RPC API（如果配置了的话）
    eth/catalyst/api.go：注册 Engine API
节点的初始化会在 cmd/geth/config.go 中的 makeFullNode 中完成，重点会初始化以下三个模块：
![重点会初始化以下三个模块](https://forum.lxdao.io/uploads/default/original/2X/6/66dabd43a59e5bfe9962a5afb2ef0ecc6e90e002.png)

在第一步会初始化 node/node.go 中的 Node 结构，就是整个节点容器，所有的功能都需要在这个容器中运行，第二步会初始化 Ethereum 结构，其中包括以太坊各种核心功能的实现，Etherereum 也需要注册到 Node 中，第三步就是注册 Engine API 到 Node 中。

其中 Node 初始化就是创建了一个 Node 实例，然后初始化 p2p server、账号管理以及 http 等暴露给外部的协议端口。
![Node初始化](https://forum.lxdao.io/uploads/default/optimized/2X/6/66dabd43a59e5bfe9962a5afb2ef0ecc6e90e002_2_1380x644.png)


Ethereum 的初始化就会复杂很多，大多数的核心功能都是在这里初始化。首先会初始化化 ethdb，并从存储中加载链配置，然后创建共识引擎，这里的共识引擎不会执行共识操作，而只是会对共识层返回的结果进行验证，如果共识层发生了提款请求，也会在这里完成实际的提款操作。然后再初始化 Block Chain 结构和交易池。

这些都完成之后就会初始化 handler，handler 是所有 p2p 网络请求的处理入口，包括交易同步、区块下载等等，是以太坊实现去中心化运行的关键组件。在这些都完成之后，就会将一些在 devp2p 基础之上实现的子协议，比如 eth/68、snap 等注册到 Node 容器中，最后 Ethereum 会作为一个 lifecycle 注册到 Node 容器中，Ethereum 初始化完成。
![Ethereum初始化](https://forum.lxdao.io/uploads/default/optimized/2X/f/fb8584abc9d9f024033d491eecfe2a7fc5862bff_2_1380x780.png)
最后 Engine API 的初始化相对简单，只是将 Engine API 注册到 Node 中。到这里，节点初始化就全部完成了。

节点启动
在完成节点的初始化之后，就需要启动节点了，节点启动的流程相对简单，只需要将已经注册的 RPC 服务和 Lifecycle 全部启动，那么整个节点就可以向外部提供服务了。
![节点启动](https://forum.lxdao.io/uploads/default/original/2X/b/b6721d53f2e6f6afa9faad10f714889b70717b3c.png)


##### 总结
在深入理解以太坊执行层的实现之前，需要对以太坊有一个整体的认识，可以将以太坊整体看作是一个交易驱动的状态机，执行层负责交易的执行和状态的变更，共识层则负责驱动执行层运行，包括让执行层产出区块、决定交易的顺序、为区块投票、以及让区块获得最终性。由于这个状态机是去中心化的，所以需要通过 p2p 网络与其他的节点通信，共同维护状态数据的一致性。

在执行层不负责决定交易的顺序，只负责执行交易并记录交易执行之后的状态变化。这里的记录有两种形式，一种是以区块的方式将所有的状态变化都记录下来，另一种是在数据库中记录当前的状态。同时执行层也是交易的入口，通过交易池来存储还没有被打包进区块的交易。如果其他的节点需要获取区块、状态和交易数据，执行层就会通过 p2p 网络将这些信息发送出去。

对于执行层，有三个核心模块：计算、存储和网络。计算对应 EVM 的实现，存储则对应了 ethdb 的实现，网络对了 devp2p 的实现。有了这样的整体认识之后，就可以深入去理解每一个子模块，而不会迷失在具体的细节中。

### 2025.06.24
#### Geth底层数据库
Geth 将其数据库分为两部分：快速访问存储（KV数据库，用于最近的区块和状态数据）和称为 freezer 的存储（用于较旧的区块和收据数据，即“ancients”）。

    快速访问存储: Geth 用户可能对底层数据库的选项较为熟悉——可以通过 --db.engine 参数进行配置。目前的默认选项是 pebbledb，也可以选择 leveldb。这两个都是 Geth 所依赖的第三方键值数据库，负责存储路径为 datadir/geth/chaindata（所有的区块和状态数据）和 datadir/geth/nodes（数据库元数据文件，体积很小）的文件。通过 --history.state value 设置快速访问保存的最近历史状态区块数，默认为90,000个区块。
    freezer 或 ancients存储（历史数据），其目录路径通常为 datadir/geth/chaindata/ancients。由于历史数据基本是静态的，不需要高性能 I/O，因此可以节省宝贵的 SSD 空间，用于存储更活跃的数据。
本文的重点是状态数据，它存储在 KV 数据库中。因此，文中提到的底层数据库默认是指这个 KV 存储，而非 freezer。

#### Geth存储结构：五个逻辑数据库
Geth 的底层使用 LevelDB/PebbleDB 存储所有以 RLP 编码后的数据，但其逻辑上划分出五种用途不同的数据库：
| 名称    | 描述 |
| ---------------- | ---------------------------- |
| State Trie       | 世界状态，包括账户、合约存储      |
| Contract Codes   | 合约代码                      |
| State snapshot   | 世界状态快照                   |
| Receipts         | 交易收据                      |
| Headers/Blocks   | 区块数据                      |
	
每种数据通过 key 前缀（core/rawdb/schema.go）区分，逻辑上实现职责分离。通过geth db inspect可以查看 Geth 存储的所有以太坊数据（块高22,347,000），可以看到磁盘空间占用最大的是区块、收据和状态数据。

#### 源码视角下的存储分层: 6种DB
Geth 中包含
StateDB、state.Database、trie.Trie、TrieDB、rawdb 和 ethdb 六个数据库模块，
它们如同一棵“状态生命树”的各个层级。最顶层的 StateDB 是 EVM 执行阶段的状态接口，负责处理账户与存储的读写请求，并将这些请求逐层下传，最终由最底层负责物理持久化的 ethdb 读/写物理数据库。
![6种DB](https://forum.lxdao.io/uploads/default/original/2X/9/9c386e7b33c68114a619f8a60b4ad3e86db87e92.png)

### 2025.06.25
##### StateDB

在 Geth 中，StateDB 是 EVM 与底层状态存储之间的唯一桥梁，负责抽象和管理合约账户、余额、nonce、存储槽等信息的读写，对所有其他数据库（TrieDB, EthDB）的状态相关读写都由StateDB中的相关接口触发，可以说 StateDB 是所有状态数据库的大脑。
它并不直接操作底层的 Trie 或底层数据库（ethdb），而是提供一个简化的内存视图，让 EVM 能以熟悉的账户模型进行交互。
因此，多数依赖 Geth 的项目其实也不会关心底层的 EthDB或 TrieDB 是怎么实现的——它们能正常工作就够了，没必要动。大多数基于 Geth 的分叉项目都会修改StateDB 结构，以适应自己的业务逻辑。例如，Arbitrum 修改了 StateDB 以管理它们的 Stylus 程序；EVMOS 修改了 StateDB 来追踪对其有状态预编译合约（stateful precompile）的调用。

源码中，StateDB 的主要定义位于 core/state/statedb.go。它的核心结构维护了一系列内存状态对象（stateObject），每个stateObject对应一个账户（包含合约存储）。它还包含一个 journal（事务日志）用于支持回滚，以及用于追踪状态更改的缓存机制。在交易处理和区块打包过程中，StateDB 提供临时状态变更的记录，只有在最终确认后才会写入底层数据库。

StateDB 的核心读写接口如下，基本都是账户模型相关的API:
```
// 读相关
func (s *StateDB) GetBalance(addr common.Address) *uint256.Int 
func (s *StateDB) GetStorageRoot(addr common.Address) common.Hash 
// 写入dirty状态数据
func (s *StateDB) SetStorage(addr common.Address, storage map[common.Hash]common.Hash) 
// 将EVM执行过程中发生的状态变更(dirty数据) commit到后端数据库中
func (s *StateDB) commitAndFlush(block uint64, deleteEmptyObjects bool, noStorageWiping bool) (*stateUpdate, error)
```

###### 生命周期
StateDB的生命周期只持续一个区块。在一个区块被处理并提交之后，这个 StateDB 就会被废弃，不再具有作用
```
EVM 第一次读取某个地址时，StateDB 会从Trie→TrieDB→EthDB数据库中加载它的值，并将其缓存一个新的状态对象（stateObject.originalStorage）中。这一阶段被视为“干净的对象”（clean object）。
当交易与该账户发生交互并改变其状态时，对象就变成“脏的”（dirty）。stateObject会同时追踪该账户的原始状态和所有修改后的数据，包括其存储槽及其干净/脏状态。
如果整个交易最终成功被打包到区块，StateDB.Finalise() 会被调用。这个函数负责清理已 selfdestruct 的合约，并重置 journal（事务日志）以及 gas refund 计数器。
当所有交易都执行完毕后，StateDB.Commit() 被调用。在这之前，状态树Trie实际上还未被更改。直到这一步，StateDB 才会将内存中的状态变更写入存储 Trie，计算出每个账户的最终 storage root，从而生成账户的最终状态。接下来，所有“脏”的状态对象会被写入 Trie中，更新其结构并计算新的 stateRoot。
最后，这些更新后的节点会被传递给 TrieDB，它会根据不同的后端（PathDB/HashDB）缓存这些节点，并最终将它们持久化到磁盘（LevelDB/PebbleDB）——前提是这些数据没有因为链重组被丢弃。
```
### 2025.06.26
##### State.Database

state.Database 是 Geth 中连接 StateDB 与底层数据库（EthDB 与 TrieDB）的重要中间层，它为状态访问提供了一组简洁的接口和实用方法。虽然它的接口比较薄，但在源码中，它扮演了多个关键角色，尤其是在状态树访问与优化方面。

在 Geth 源码中（core/state/database.go），state.Database 接口由 state.cachingDB 这一具体数据结构实现。它的主要作用包括：

**提供统一的状态访问接口**
state.Database 是构建 StateDB 的必要依赖，它封装了打开账户 Trie 和存储 Trie 的逻辑，例如：

    ```
    func (db *cachingDB) OpenTrie(root common.Hash) (Trie, error)
    func (db *cachingDB) OpenStorageTrie(stateRoot common.Hash, address common.Address, root common.Hash, trie Trie) (Trie, error)
    ```
这些方法隐藏了底层 TrieDB 的复杂性，开发者在构建某个区块的状态时，只需调用这些方法获取正确的 Trie 实例，而不必直接操作 hash 路径、trie 编码或底层数据库。
**暂存和复用合约代码（code cache）**
合约代码的访问代价较高，且往往在多个块中重复使用。为此，state.Database 中实现了代码缓存逻辑，避免重复从磁盘加载合约字节码。这一优化对提高区块执行效率至关重要：

```
func (db *CachingDB) ContractCodeWithPrefix(address common.Address, codeHash common.Hash) []byte
```

这个接口允许按地址和代码哈希快速命中缓存，若未命中才回退到底层数据库加载。

**长生命周期，跨多个区块复用**
与 StateDB 的生命周期仅限于单个区块不同，state.Database 的生命周期和整个链（core.Blockchain）保持一致。它在节点启动时构造，并贯穿整个运行周期，作为 StateDB 的“忠实伙伴”，为其在每个区块处理时提供支持。

**为未来的 Verkle Tree 迁移做准备**
虽然当前 state.Database 看似只是“代码缓存+trie访问封装”，但它在 Geth 架构中的定位非常前瞻性。一旦未来的状态结构切换至 Verkle Trie，它将成为迁移过程的核心组件：处理新旧结构之间的桥接状态。


<!-- Content_END -->
