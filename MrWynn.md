---
timezone: UTC+8
---

> 请在上边的 timezone 添加你的当地时区(UTC)，这会有助于你的打卡状态的自动化更新，如果没有添加，默认为北京时间 UTC+8 时区


# wynn

1. 本人是一名接触web3一年多的后端程序员，目前正在一家小的dex团队负责后端相关的工作，主要技术栈是go，想深入了解并学习下以太坊协议相关知识，期望在未来能够为以太坊开发做出点贡献。
2. Absolutely!
3. TG: @wynn520

## Notes

<!-- Content_START -->

### 2025.06.16

#### 主要学习区块构建和交易执行流程

区块如何产生的？  

当共识层通过 engine API's fork choice updated 端点指示执行层客户端时，区块就被创建了，然后通过 payload building routine 启动区块构建过程。

每个时隙都有一个指定的区块提议者，通过共识层的伪随机过程选出。当一个验证者被选为某个时隙的区块提议者时，它的共识客户端通过执行引擎的fork choice updated方法启动区块构建过程，该方法提供了构建区块所需的上下文。

什么是时隙？
```text
以太坊中的“时隙”（slot）是其共识机制（自2022年“合并”后采用权益证明 PoS）中最基本的时间单位，固定为12秒。每个时隙对应一次区块生成机会，由随机选出的验证者负责提议新区块。
```

时隙的核心定义与作用？  
1.时间单位
 - 时隙是信标链（Beacon Chain）的最小时间划分单元，固定为12秒
 - 在理想网络条件下，每个时隙应生成一个新区块。若验证者离线或未及时提交，该时隙可能为空（“跳过时隙”）

2.区块产生机制
 - 区块提议者：每个时隙开始时，系统通过伪随机算法从验证者池中选出一名作为提议者，负责打包交易并创建新区块
 - 验证职责：其他验证者被分配为“证明者”（attesters），对提议的区块进行验证和投票，确保其有效性

下面是一个简化并模拟的区块构建过程：
```go
func build(env environment, pool txpool.Pool, state state.StateDB) (types.Block, state.StateDB) {
    var (
        gasUsed = 0
        txs []types.Transactions
    )

    for ; gasUsed < 30_000_000 || !pool.Empty(); {
        transaction := pool.Pop()
        res, gas, err := vm.Run(env, transaction, state)
        if err != nil {
            // transaction invalid
            continue
        }
        gasUsed += gas
        transactions = append(transactions, transaction)
    }
    return core.Finalize(env, transactions, state)
}
```
1.接收 3 个传参变量
 - env environment 包含环境所有必要信息，包括时间戳、区块编号、前置区块、基础费用以及需要在区块中发生的所有提取操作, 这些信息本质上来源于共识层。
 - 交易池 txpool.Pool 变量，即交易的集合，为简单起见，我们假设这些交易按其价值升序排列，价值越高的交易更容易被确认打包
 - state.StateDB 状态数据库表示执行这些交易的状态存储

最后生成并返回以下内容:

 - 一个完整的区块
 - 一个包含所有交易的状态数据库（state DB），它记录了所有交易的执行结果和更新后的状态
 - 如果在处理过程中出现了错误，还可能会返回一个错误信息

2. 在 build 函数中，我们跟踪 gas 消耗 gasUsed，因为我们可以使用的 gas 是有限的。我们还存储所有将被包含在区块中的交易。

3. 我们继续添加交易，直到交易池为空，或者消耗的 gas 超过 gas 限制。为了简单起见，在这个例子中，gas 限制设定为 3000 万（大约是主网当前的 gas 限制）。

4. 为了获取一笔交易，我们必须查询交易池，假设交易池会维护一个有序的交易列表，确保我们始终获取到下一个最有价值的交易。

5. 交易在 EVM 中执行，在执行交易时，系统将交易、环境和当前状态作为输入，并在这些输入条件下执行交易。交易的执行会根据当前环境（如区块链状态）进行，并在执行过程中更新状态数据库，包括所有已成功执行的交易

6. 如果交易执行失败，并且在执行过程中发生错误，我们将继续处理下一笔交易，而不立即中断。这表明该交易无效，并且由于区块中仍有未使用的 gas，我们不希望立即生成错误。因为在区块中尚未发生错误，因此我们可以继续进行处理。然而，很可能该交易是无效的，因为它在执行过程中发生了错误，或者交易池中的数据略有过时。在这种情况下，我们允许继续并尝试从交易池中获取下一笔交易，继续将其加入到当前区块中。

7. 一旦我们验证运行交易没有错误，我们就会将该交易添加到交易列表中，并将运行返回的气体添加到所使用的气体中。例如，如果第一笔交易是简单的转账，需要花费 21,000 Gas，那么我们使用的 Gas 将从 0 到 21,000，我们将继续执行此过程步骤 3-7，直到满足步骤 3 的条件

8. 我们通过获取一组交易和相关区块信息， 以最终确定生成一个完整的区块, 这样做的目的是为了最后进行一定的计算。由于 header 包含交易根、收据根和提款根，因此必须通过默克尔化列表来计算这些值并将其添加到块的 header 中。

### 2025.06.17

## geth 是什么？

`geth` 是以太坊基金会基于 Go 语言开发以太坊的官方执行层客户端，它实现了 Ethereum 协议(黄皮书)中所有需要的实现的功能模块。我们可以通过启动 `geth` 来运行一个 Ethereum 的节点。在以太坊 Merge 之后，`geth` 作为节点的执行层继续在以太坊生态中发挥重要的作用。 `go-ethereum`是包含了 `geth` 客户端代码和以及编译 `geth` 所需要的其他代码在内的一个完整的代码库。

总结的来说:

1. 基于 `go-ethereum` 代码库中的代码，可以编译出 `geth` 客户端程序。
2. 通过运行 `geth` 客户端程序可以启动一个 Ethereum 的节点。

### go-ethereum 的代码库结构

根据主要的业务功能，我们可以把 `go-ethereum` 划分成如下几个模块。

- Geth Client 模块
- Core 数据结构模块
- State Management 模块
  - StateDB 模块
  - Trie 数据结构模块
  - State Optimization (Pruning)
- Mining 模块
- EVM 模块
- P2P 网络模块
  - 节点数据同步
    - 交易数据
    - 区块数据
    - 区块链数据
- Storage 模块
  - 抽象数据库层
  - LevelDB 调用
- ...

目前，go-ethereum 代码库中的主要目录结构如下所示:

```
cmd/ 以太坊基金会官方开发的一些 Command-line 程序。该目录下的每个子目录都是一个单独运行的 CLI 程序。
   |── clef/ 以太坊官方推出的账户管理程序.
   |── geth/ 以太坊官方的节点客户端。
core/   以太坊核心模块，包括核心数据结构，statedb，EVM 等核心数据结构以及算法实现
   |── rawdb/ db 相关函数的高层封装(在 ethdb 和更底层的 leveldb 之上的封装)
      ├──accessors_state.go 从 Disk Level 读取/写入与 State 相关的数据结构。
   |── state/
      ├── statedb.go  StateDB 是管理以太坊 World State 最核心的代码，用于管理链上所有的 State 相关操作。
      ├── state_object.go state_object 是以太坊账户(包括 EOA & Contract)在 StateDB 具体的实现。
   |── txpool        Transaction Pool 相关的代码。
      |── txpool.go  Transaction Pool 的具体实现。
   |── types/  以太坊中最核心的数据结构
      |── block.go   以太坊 Block 的的数据结构定义与相关函数实现
      |── bloom9.go  以太坊使用的一个 Bloom Filter 的实现
      |── transaction.go 以太坊 Transaction 的数据结构定义与相关函数实现。
      |── transaction_signing.go 用于对 Transaction 进行签名的函数的实现。
      |── receipt.go  以太坊交易收据的实现，用于记录以太坊 Transaction 执行的结果
   |── vm/            以太坊的核心中核心 EVM 相关的一些的数据结构的定义。
      |── evm.go            EVM 数据结构和方法的定义
      |── instructions.go   EVM 指令的具体的定义，核心中的核心中的核心文件。
      |── logger.go   用于追踪 EVM 执行交易过程的日志接口的定义。具体的实现在eth/tracers/logger/logger.go 文件中。
      |── opcode.go   EVM 指令和数值的对应关系。
   |── genesis.go     创世区块相关的函数。每个 geth 客户端/以太坊节点初始化的都需要调用这个模块。
   |── state_processor.go EVM 执行交易的核心代码模块。 
console/
   |── bridge.go
   |── console.go  Geth Web3 控制台的入口
eth/      Ethereum 节点/后端/客户端具体功能定义和实现。例如节点的启动关闭，P2P 网络中交易和区块的同步。
ethdb/    Ethereum 本地存储的相关实现, 包括 leveldb 的调用
   |── leveldb/   Go-Ethereum使用的与 Bitcoin Core version一样的Leveldb作为本机存储用的数据库
internal/ 一些内部使用的工具库的集合，比如在测试用例中模拟 cmd 的工具。在构建 Ethereum 生态相关的工具时值得注意这个文件夹。
miner/
   |── miner.go   矿工模块的实现。
   |── worker.go  Block generation 的实现，包括打包 transaction，计算合法的 Block
p2p/     Ethereum 的P2P模块
   |── params    Ethereum 的一些参数的配置，例如: bootnode 的 enode 地址
   |── bootnodes.go  bootnode 的 enode 地址 like: aws 的一些节点，azure 的一些节点，Ethereum Foundation 的节点和 Rinkeby 测试网的节点
rlp/     RLP的 Encode与 Decode的相关
rpc/     Ethereum RPC客户端的实现
les/     Ethereum light client 轻节点的实现
trie/    Ethereum 中至关重要的数据结构 Merkle Patrica Trie(MPT) 的实现
   |── committer.go    Trie 向 Memory Database 提交数据的工具函数。
   |── database.go     Memory Database，是 Trie 数据和 Disk Database 提交的中间层。同时还实现了 Trie 剪枝的功能。**非常重要**
   |── node.go         MPT中的节点的定义以及相关的函数。
   |── secure_trie.go  基于 Trie 的封装的结构。与 trie 中的函数功能相同，不过secure_trie中的 key 是经过hashKey()函数hash过的，无法通过路径获得原始的 key值 
   |── stack_trie.go   Block 中使用的 Transaction/Receipt Trie 的实现
   |── trie.go         MPT 具体功能的函数实现。
 ```

## 如何启动Geth节点

### 前奏: Geth Console

`geth` 内置了一个 Javascript 的解释器: *Goja* (interpreter)，来作为用户与 `geth` 交互的 CLI Console。我们可以在`console/console.go` 中找到它的定义。

<!-- /*Goja is an implementation of ECMAScript 5.1 in Pure GO*/ -->

```go
// Console is a JavaScript interpreted runtime environment. It is a fully fledged
// JavaScript console attached to a running node via an external or in-process RPC
// client.
type Console struct {
 client   *rpc.Client         // RPC client to execute Ethereum requests through
 jsre     *jsre.JSRE          // JavaScript runtime environment running the interpreter
 prompt   string              // Input prompt prefix string
 prompter prompt.UserPrompter // Input prompter to allow interactive user feedback
 histPath string              // Absolute path to the console scrollback history
 history  []string            // Scroll history maintained by the console
 printer  io.Writer           // Output writer to serialize any display strings to
}
```

### geth 节点的启动流程

了解 Ethereum，我们首先要了解 Ethereum 客户端 Geth 是怎么运行的。 geth 程序的启动点位于 `cmd/geth/main.go/main()` 函数处，如下所示。

```go
func main() {
 if err := app.Run(os.Args); err != nil {
  fmt.Fprintln(os.Stderr, err)
  os.Exit(1)
 }
}
```

我们可以看到 `main()` 函数非常的简短，其主要功能就是启动一个解析 command line命令的工具: `gopkg.in/urfave/cli.v1`。继续深入，我们会发现在 cli app 初始化的时候会调用 `app.Action = geth` ，来调用 `geth()` 函数。而 `geth()` 函数就是用于启动 Ethereum 节点的顶层函数，其代码如下所示。

```go
func geth(ctx *cli.Context) error {
 if args := ctx.Args(); len(args) > 0 {
  return fmt.Errorf("invalid command: %q", args[0])
 }

 prepare(ctx)
 stack, backend := makeFullNode(ctx)
 defer stack.Close()

 startNode(ctx, stack, backend, false)
 stack.Wait()
 return nil
}
```

在 `geth()` 函数中，有三个比较重要的函数调用，分别是：`prepare()`，`makeFullNode()`，以及 `startNode()`。

`prepare()` 函数的实现就在当前的 `main.go` 文件中。它主要用于设置一些节点初始化需要的配置。比如，我们在节点启动时看到的这句话: *Starting Geth on Ethereum mainnet...* 就是在 `prepare()` 函数中被打印出来的。

`makeFullNode()` 函数的实现位于 `cmd/geth/config.go` 文件中。它会将 Geth 启动时的命令的上下文加载到配置中，并生成 `stack` 和`backend` 这两个实例。其中 `stack` 是一个 Node 类型的实例，它是通过 `makeFullNode()` 函数调用 `makeConfigNode()` 函数来初始化的。`Node` 是 geth 生命周期中最顶级的实例，它负责管理节点中的 P2P Server, Http Server, Database 等业务非直接相关的高级抽象。关于 Node 类型的定义位于`node/node.go`文件中。

这里的 `backend` 是一个 `ethapi.Backend` 类型的接口，提供了获取以太坊执行层运行时，所需要的基本函数功能。它的定义位于 `internal/ethapi/backend.go` 中。 由于这个接口中函数较多，我们选取了其中的部分关键函数方便大家理解这个接口所提供的基本功能，如下所示。

```go
type Backend interface {
	SyncProgress() ethereum.SyncProgress
	SuggestGasTipCap(ctx context.Context) (*big.Int, error)
	ChainDb() ethdb.Database
	AccountManager() *accounts.Manager
	ExtRPCEnabled() bool
	RPCGasCap() uint64            // global gas cap for eth_call over rpc: DoS protection
	RPCEVMTimeout() time.Duration // global timeout for eth_call over rpc: DoS protection
	RPCTxFeeCap() float64         // global tx fee cap for all transaction related APIs
	UnprotectedAllowed() bool     // allows only for EIP155 transactions.
	SetHead(number uint64)
	HeaderByNumber(ctx context.Context, number rpc.BlockNumber) (*types.Header, error)
	HeaderByHash(ctx context.Context, hash common.Hash) (*types.Header, error)
	HeaderByNumberOrHash(ctx context.Context, blockNrOrHash rpc.BlockNumberOrHash) (*types.Header, error)
	CurrentHeader() *types.Header
	CurrentBlock() *types.Header
	BlockByNumber(ctx context.Context, number rpc.BlockNumber) (*types.Block, error)
	BlockByHash(ctx context.Context, hash common.Hash) (*types.Block, error)
	BlockByNumberOrHash(ctx context.Context, blockNrOrHash rpc.BlockNumberOrHash) (*types.Block, error)
	StateAndHeaderByNumber(ctx context.Context, number rpc.BlockNumber) (*state.StateDB, *types.Header, error)
	StateAndHeaderByNumberOrHash(ctx context.Context, blockNrOrHash rpc.BlockNumberOrHash) (*state.StateDB, *types.Header, error)
	PendingBlockAndReceipts() (*types.Block, types.Receipts)
	GetReceipts(ctx context.Context, hash common.Hash) (types.Receipts, error)
	GetTd(ctx context.Context, hash common.Hash) *big.Int
	GetEVM(ctx context.Context, msg *core.Message, state *state.StateDB, header *types.Header, vmConfig *vm.Config) (*vm.EVM, func() error, error)
	SubscribeChainEvent(ch chan<- core.ChainEvent) event.Subscription
	SubscribeChainHeadEvent(ch chan<- core.ChainHeadEvent) event.Subscription
	SubscribeChainSideEvent(ch chan<- core.ChainSideEvent) event.Subscription
	SendTx(ctx context.Context, signedTx *types.Transaction) error
	GetTransaction(ctx context.Context, txHash common.Hash) (*types.Transaction, common.Hash, uint64, uint64, error)
	GetPoolTransactions() (types.Transactions, error)
	GetPoolTransaction(txHash common.Hash) *types.Transaction
	GetPoolNonce(ctx context.Context, addr common.Address) (uint64, error)
	Stats() (pending int, queued int)
	TxPoolContent() (map[common.Address]types.Transactions, map[common.Address]types.Transactions)
	TxPoolContentFrom(addr common.Address) (types.Transactions, types.Transactions)
	SubscribeNewTxsEvent(chan<- core.NewTxsEvent) event.Subscription
	ChainConfig() *params.ChainConfig
	Engine() consensus.Engine
	GetBody(ctx context.Context, hash common.Hash, number rpc.BlockNumber) (*types.Body, error)
	GetLogs(ctx context.Context, blockHash common.Hash, number uint64) ([][]*types.Log, error)
	SubscribeRemovedLogsEvent(ch chan<- core.RemovedLogsEvent) event.Subscription
	SubscribeLogsEvent(ch chan<- []*types.Log) event.Subscription
	SubscribePendingLogsEvent(ch chan<- []*types.Log) event.Subscription
	BloomStatus() (uint64, uint64)
	ServiceFilter(ctx context.Context, session *bloombits.MatcherSession)
}
```

我们可以发现 `ethapi.Backend` 接口主要对外提供了: 1. General Ethereum APIs, 这些 General APIs 对外提供了查询区块链节点管理对象的接口，例如 `ChainDb()` 返回当前节点的 DB 实例, `AccountManager()`; 2.  Blockchain 相关的 APIs, 例如链上数据的查询(Block & Transaction), `CurrentHeader(), BlockByNumber(), GetTransaction()`; 3. Transaction Pool 相关的APIs, 例如发送交易到本节点的 Transaction Pool, 以及查询交易池中的 Transactions, `GetPoolTransaction`。

目前 Geth 代码库中，有两个 `ethapi.Backend` 接口的实现，分别是: 1. 位于 `eth\api_backend` 中的 `EthAPIBackend`; 2. 位于 `les\api_backend` 的 `LesApiBackend`; 顾名思义，`EthAPIBackend` 提供了针对全节点的 Backend API 服务, 而 `LesApiBackend` 提供了轻节点的 Backend API 服务。总结的来说，如果读者想定制一些新的 RPC API，可以在 `ethapi.Backend` 接口中定义函数，并给 `EthAPIBackend` 添加具体的实现。

`ethapi.Backend` 接口所提供的函数功能，主要读写本地的维护的数据结构(i.e. Transaction Pool, Blockchain)的为主。那么作为一个有网络连接的 Backend, 以太坊的 Backend 或者说 Node 是怎么管理以太坊执行层节点的网络连接，共识等功能模块的呢？

我们深入 `makeFullNode()` 函数可以发现，生成`ethapi.Backend` 接口的语句 `backend, eth := utils.RegisterEthService(stack, &cfg.Eth)`, 还返回了另一个 `Ethereum` 类型的实例 `eth`。 这个 `Ethereum` 类型才是以太坊节点数结构中核心中的核心，它实现了以太坊全节点所需要的所有的 Service。它负责提供更为具体的以太坊的功能性 Service, 负责与以太坊业务直接相关的抽象，比如维护 Blockchain 的更新，共识算法，从 P2P 网络中同步区块，同步P2P节点远端的交易并放到交易池中，等业务功能。我们会在后续详细讲解 `Ethereum` 类型具体提供的服务。

`Ethereum` 实例根据上下文的配置信息在调用 `utils.RegisterEthService()` 函数生成。在`utils.RegisterEthService()`函数中，首先会根据当前的config来判断需要生成的Ethereum backend 的类型，是 light node backend 还是 full node backend。我们可以在 `eth/backend/new()` 函数和 `les/client.go/new()` 中找到这两种 Ethereum backend 的实例是如何初始化的。Ethereum backend 的实例定义了一些更底层的配置，比如chainid，链使用的共识算法的类型等。这两种后端服务的一个典型的区别是 light node backend 不能启动 Mining 服务。在 `utils.RegisterEthService()` 函数的最后，调用了 `Nodes.RegisterAPIs()` 函数，将刚刚生成的 backend 实例注册到 `stack` 实例中。

总结的说，`api_backend` 主要是用于对外提供查询，或者与后端功能性生命周期无关的函数，`Ethereum` 这类的节点层的后端，主要用于管理/控制节点后端的生命周期。

最后一个关键函数，`startNode()` 的作用是正式的启动一个以太坊执行层的节点。它通过调用 `utils.StartNode()` 函数来触发 `Node.Start()` 函数来启动`Stack`实例(Node)。在 `Node.Start()` 函数中，会遍历 `Node.lifecycles` 中注册的后端实例，并启动它们。此外，在 `startNode()` 函数中，还是调用了`unlockAccounts()` 函数，并将解锁的钱包注册到 `stack` 中，以及通过 `stack.Attach()` 函数创建了与 local Geth 交互的 RPClient 模块。

在 `geth()` 函数的最后，函数通过执行 `stack.Wait()`，使得主线程进入了阻塞状态，其他的功能模块的服务被分散到其他的子协程中进行维护。

### Node 节点

Node 类型在 geth 的生命周期性中属于顶级实例，它负责作为与外部通信的高级抽象模块的管理员，比如管理 rpc server，http server，Web Socket，以及 P2P Server外 部接口。同时，Node 中维护了节点运行所需要的后端的实例和服务 (`lifecycles  []Lifecycle`)，例如我们上面提到的负责具体 Service 的`Ethereum` 类型。

```go
// Node is a container on which services can be registered.
type Node struct {
 eventmux      *event.TypeMux
 config        *Config
 accman        *accounts.Manager
 log           log.Logger
 keyDir        string            // key store directory
 keyDirTemp    bool              // If true, key directory will be removed by Stop
 dirLock       fileutil.Releaser // prevents concurrent use of instance directory
 stop          chan struct{}     // Channel to wait for termination notifications
 server        *p2p.Server       // Currently running P2P networking layer
 startStopLock sync.Mutex        // Start/Stop are protected by an additional lock
 state         int               // Tracks state of node lifecycle
 lock          sync.Mutex
 lifecycles    []Lifecycle // All registered backends, services, and auxiliary services that have a lifecycle
 rpcAPIs       []rpc.API   // List of APIs currently provided by the node
 http          *httpServer //
 ws            *httpServer //
 httpAuth      *httpServer //
 wsAuth        *httpServer //
 ipc           *ipcServer  // Stores information about the ipc http server
 inprocHandler *rpc.Server // In-process RPC request handler to process the API requests

 databases map[*closeTrackingDB]struct{} // All open databases
}
```

#### 关闭节点

在前面我们提到，整个程序的主线程因为调用了 `stack.Wait()` 而进入了阻塞状态。我们可以看到 Node 结构中声明了一个叫做 `stop` 的 channel。由于这个 Channel 一直没有被赋值，所以整个 geth 的主进程才进入了阻塞状态，持续并发的执行其他的业务协程。

```go
// Wait blocks until the node is closed.
func (n *Node) Wait() {
 <-n.stop
}
```

当 `n.stop` 这个 Channel 被赋予值的时候，`geth` 主函数就会停止当前的阻塞状态，并开始执行相应的一系列的资源释放的操作。

值得注意的是，在目前的 go-ethereum 的 codebase 中，并没有直接通过给 `stop` 这个 channel 赋值方式来结束主进程的阻塞状态，而是使用一种更简洁粗暴的方式: 调用 `close()` 函数直接关闭 Channel。我们可以在 `node.doClose()` 找到相关的实现。`close()` 是 go 语言的原生函数，用于关闭 Channel 时使用。

```go
// doClose releases resources acquired by New(), collecting errors.
func (n *Node) doClose(errs []error) error {
 // Close databases. This needs the lock because it needs to
 // synchronize with OpenDatabase*.
 n.lock.Lock()
 n.state = closedState
 errs = append(errs, n.closeDatabases()...)
 n.lock.Unlock()

 if err := n.accman.Close(); err != nil {
  errs = append(errs, err)
 }
 if n.keyDirTemp {
  if err := os.RemoveAll(n.keyDir); err != nil {
   errs = append(errs, err)
  }
 }

 // Release instance directory lock.
 n.closeDataDir()

 // Unblock n.Wait.
 close(n.stop)

 // Report any errors that might have occurred.
 switch len(errs) {
 case 0:
  return nil
 case 1:
  return errs[0]
 default:
  return fmt.Errorf("%v", errs)
 }
}
```

### Ethereum 后端设计

我们可以在 `eth/backend.go` 中找到 `Ethereum` 这个结构体的定义。这个结构体包含的成员变量以及接收的方法实现了一个 Ethereum full node 所需要的全部功能和数据结构。我们可以在下面的代码定义中看到，Ethereum结构体中包含 `TxPool`，`Blockchain`，`consensus.Engine`，`miner`等最核心的几个数据结构作为成员变量。

```go
type Ethereum struct {
	config *ethconfig.Config
	txPool     *txpool.TxPool
	blockchain *core.BlockChain
   handler            *handler     // handler 是P2P 网络数据同步的核心实例，我们会在后续的网络同步模块仔细的讲解它的功能
	ethDialCandidates  enode.Iterator
	snapDialCandidates enode.Iterator
	merger             *consensus.Merger
	chainDb ethdb.Database // Block chain database
	eventMux       *event.TypeMux
	engine         consensus.Engine
	accountManager *accounts.Manager
	bloomRequests     chan chan *bloombits.Retrieval // Channel receiving bloom data retrieval requests
	bloomIndexer      *core.ChainIndexer             // Bloom indexer operating during block imports
	closeBloomHandler chan struct{}
	APIBackend *EthAPIBackend
	miner     *miner.Miner
	gasPrice  *big.Int
	etherbase common.Address
	networkID     uint64
	netRPCService *ethapi.NetAPI
	p2pServer *p2p.Server
	lock sync.RWMutex // Protects the variadic fields (e.g. gas price and etherbase)
	shutdownTracker *shutdowncheck.ShutdownTracker // Tracks if and when the node has shutdown ungracefully
}
```

节点启动和停止 Mining 的就是通过调用 `Ethereum.StartMining()` 和 `Ethereum.StopMining()` 实现的。设置 Mining 的收益账户是通过调用 `Ethereum.SetEtherbase()` 实现的。

```go
// StartMining starts the miner with the given number of CPU threads. If mining
// is already running, this method adjust the number of threads allowed to use
// and updates the minimum price required by the transaction pool.
func (s *Ethereum) StartMining(threads int) error {
   ...
 // If the miner was not running, initialize it
 if !s.IsMining() {
      ...
      // Start Mining
  go s.miner.Start(eb)
 }
 return nil
}
```


我们从从宏观角度来看，一个节点的主工作流需要: 1.从网络中获取/同步 Transaction 和 Block 的数据 2. 将网络中获取到 Block 添加到 Blockchain 中。而 `handler` 就负责提供中同步区块和交易数据的功能，例如，`downloader.Downloader` 负责从网络中同步 Block ，`fetcher.TxFetcher` 负责从网络中同步交易。

```go
type handler struct {
 networkID  uint64
 forkFilter forkid.Filter // Fork ID filter, constant across the lifetime of the node

 snapSync  uint32 // Flag whether snap sync is enabled (gets disabled if we already have blocks)
 acceptTxs uint32 // Flag whether we're considered synchronised (enables transaction processing)

 checkpointNumber uint64      // Block number for the sync progress validator to cross reference
 checkpointHash   common.Hash // Block hash for the sync progress validator to cross reference

 database ethdb.Database
 txpool   txPool
 chain    *core.BlockChain
 maxPeers int

 downloader   *downloader.Downloader
 blockFetcher *fetcher.BlockFetcher
 txFetcher    *fetcher.TxFetcher
 peers        *peerSet
 merger       *consensus.Merger

 eventMux      *event.TypeMux
 txsCh         chan core.NewTxsEvent
 txsSub        event.Subscription
 minedBlockSub *event.TypeMuxSubscription

 peerRequiredBlocks map[uint64]common.Hash

 // channels for fetcher, syncer, txsyncLoop
 quitSync chan struct{}

 chainSync *chainSyncer
 wg        sync.WaitGroup
 peerWG    sync.WaitGroup
}
```

### 2025.06.18

## EOA账户和合约账户

Ethereum 的运行是一种*基于交易的状态机模型*(Transaction-based State Machine)。整个系统由若干的账户组成 (Account)，类似于银行账户。状态(State)反应了某一账户(Account)在*某一时刻*下的值(value)。在以太坊中，State 对应的基本数据结构，称为 StateObject。当 StateObject 的值发生了变化时，我们称为*状态转移*。在 Ethereum 的运行模型中，StateObject 所包含的数据会因为 Transaction 的执行引发数据更新/删除/创建，引发状态转移，我们说：StateObject 的状态从当前的 State 转移到另一个 State。

在 Ethereum 中，承载 StateObject 的具体实例就是 Ethereum 中的 Account 。通常，我们提到的 State 具体指的就是 Account 在某个时刻下所包含的数据的值。

- Account --> StateObject
- State   --> The value/data of the Account


`外部账户(EOA):`是由用户直接控制的账户，负责签名并发起交易(Transaction)。用户通过控制 Account 的私钥来保证对账户数据的控制权。

`合约账户(Contract):`，简称为合约，是由外部账户通过 Transaction 创建的。合约账户保存了**不可篡改的图灵完备的代码段**，以及一些**持久化的数据变量**。这些代码使用专用的图灵完备的编程语言编写(Solidity)，并通常提供一些对外部访问 API 接口函数。这些 API 接口函数可以通过构造 Transaction，或者通过本地/第三方提供的节点 RPC 服务来调用。这种模式构成了目前的 DApp 生态的基础。


## StateObject

在实际代码中，这两种 Account 都是由 `stateObject` 这一数据结构定义的。

```go
  type stateObject struct {
    address  common.Address
    addrHash common.Hash // hash of ethereum address of the account
    data     types.StateAccount
    db       *StateDB
    dbErr error

    // Write caches.
    trie Trie // storage trie, which becomes non-nil on first access
    code Code // contract bytecode, which gets set when code is loaded

    // 这里的Storage 是一个 map[common.Hash]common.Hash
    originStorage  Storage // Storage cache of original entries to dedup rewrites, reset for every transaction
    pendingStorage Storage // Storage entries that need to be flushed to disk, at the end of an entire block
    dirtyStorage   Storage // Storage entries that have been modified in the current transaction execution
    fakeStorage    Storage // Fake storage which constructed by caller for debugging purpose.

    // Cache flags.
    // When an object is marked suicided it will be delete from the trie
    // during the "update" phase of the state transition.
    dirtyCode bool // true if the code was updated
    suicided  bool
    deleted   bool
  }
```

### EOA Account Generation

创建新账户的依赖的入口函数 `NewAccount` 位于 `accounts/keystore/keystore.go` 文件中。

```go
// passphrase 参数用于本地加密
func (ks *KeyStore) NewAccount(passphrase string) (accounts.Account, error) {
//生成account的函数
 _, account, err := storeNewKey(ks.storage, crand.Reader, passphrase)
 if err != nil {
  return accounts.Account{}, err
 }
 // Add the account to the cache immediately rather
 // than waiting for file system notifications to pick it up.
 ks.cache.add(account)
 ks.refreshWallets()
 return account, nil
}
```

上述代码段中，最核心的调用是 `storeNewKey` 函数。在 `storeNewKey` 函数中，首先就调用了 `newKey` 函数，该函数的主要功能就是生成一个账户需要的私钥和公钥对。而 `newKey` 函数的核心是调用了生成椭圆曲线加密对相关的函数 `ecdsa.GenerateKey`。

```go
func newKey(rand io.Reader) (*Key, error) {
 privateKeyECDSA, err := ecdsa.GenerateKey(crypto.S256(), rand)
 if err != nil {
  return nil, err
 }
 return newKeyFromECDSA(privateKeyECDSA), nil
}

```

在整个流程中，首先生成的是账户的私钥，而账户对应的地址，是基于该私钥在椭圆曲线上对用的公钥值经过哈希计算得到的。  
如何从账户私钥计算出账户地址的。

- 首先，我们在创建一个新的 EOA 账户的时候，首先会通过 `GenerateKey` 函数随机的得到一串私钥，它是一个 32bytes 长的变量，表现为64位16进制数。这个私钥就是平时需要用户激活钱包时，发送交易时必要的门禁卡，一旦这个私钥暴露了，钱包也将不再安全。
  - 64个16进制位，256bit，32字节
    `var AlicePrivateKey = "289c2857d4598e37fb9647507e47a309d6133539bf21a8b9cb6df88fd5232032"`
- 在得到私钥后，我们使用私钥来计算公钥和地址地址。基于上述私钥，我们使用 ECDSA 算法，选择 spec256k1 曲线进行计算。通过将私钥带入到所选择的椭圆曲线中，计算出点的坐标即是公钥。以太坊和比特币使用了同样的 spec256k1 曲线，在实际的代码中，我们也可以看到在 go-Ethereum 直接调用了比特币的 secp256k1 的C语言代码。
    `ecdsaSK, err := crypto.ToECDSA(privateKey)`
- 对私钥进行椭圆加密之后，我们可以得到一个 64bytes 的数，它是由两个 32bytes 的数构成，这两个数代表了 spec256k1 曲线上某个点的 XY 坐标值。
    `ecdsaPK := ecdsaSK.PublicKey`
- 最终账户的地址，是基于上述公钥(ecdsaSK.PublicKey)进行 **Keccak-256算法** 计算之后得到的哈希值的后20个字节，用0x开头表示(Keccak-256 是 SHA-3（Secure Hash Algorithm 3）标准下的一种哈希算法)。
    `addr := crypto.PubkeyToAddress(ecdsaSK.PublicKey)`

#### Signature & Verification：签名与验证

这里我们简述一下，怎么利用 ECDSA 来进行数字签名和校验的。

- 以太坊签名校验的核心思想是:首先基于上面得到的ECDSA 下的私钥 ecdsaSK对数据 msg 进行签名 (sign) 得到 msgSig.
    `sig, err := crypto.Sign(msg[:], ecdsaSK)`
    `msgSig := decodeHex(hex.EncodeToString(sig))`
- 然后基于 msg 和 msgSig 可以反推出来签名的公钥（用于生成账户地址的公钥ecdsaPK）。
    `recoveredPub, err := crypto.Ecrecover(msg[:],msgSig)`
- 通过反推出来的公钥可以得到发送者的地址，并与当前交易的发送者在 ECDSA 下的pk进行对比。
    `crypto.VerifySignature(testPk, msg[:], msgSig[:len(msgSig)-1])`
- 这套体系的安全性保证在于，即使知道了公钥 ecdsaPk/ecdsaSK.PublicKey也难以推测出 ecdsaSK以及生成他的 privateKey。

## Contract：合约和合约存储

### Contract Storage：合约存储

 `Storage` 类型的定义。具体如下所示。

```go
type Storage map[common.Hash]common.Hash
```

我们可以看到，`Storage` 是一个 key 和 value 都是 `common.Hash` 类型的 map 结构。`common.Hash` 类型本质上是一个长度为 32bytes 的 `byte` 类型数组。

从数据层面讲，外部账户(EOA)与合约账户(Contract)不同的点在于: 外部账户并没有维护自己的代码(codeHash)以及额外的 Storage 层。相比与外部账户，合约账户额外保存了一个存储层(Storage)用于存储合约代码中持久化的变量的数据。

在 Ethereum 中，每个合约都维护了自己的*独立*的存储空间，用于保存合约中的持久化变量，我们称为 Storage 层。Storage 层的基本组成单元称为槽 (Slot)。若干个 Slot 按照栈(Stack)的方式顺序集合在一起就构造成了 Storage 层。每个 Slot 的大小是 256 bits，也就是最多保存 `32 bytes` 的数据。作为基本的存储单元，Slot 管理的方式与内存或者HDD中的基本单元的管理方式类似，通过地址索引的方式被上层函数访问。Slot的地址索引的长度同样是32 bytes(256 bits)，寻址空间从 0x0000000000000000000000000000000000000000000000000000000000000000 到 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF。因此，每个Contract的Storage层最多可以保存$2^{256} - 1$个 Slot。也就说在理论状态下，一个 Contract 可以最多保存 $(2^{256} - 1)$ 32 bytes的数据，这是个相当大的数字。

为了更好的管理 Storage 层的 Slot 数据，Contract 同样使用 MPT 作为索引来管理 Storage 层的Slot。这里值得注意的是，合约 Storage 层的数据并不会跟随交易一起被打包进入 Block 中。正如我们之前提到的，管理合约账户中 Storage 层 Storage Trie 的根数据被保存在 StateAccount 结构体中的 Root 变量中(它是一个32bytes长的byte数组)。当某个 Contract 的 Storage 层的数据发生变化时，会向上传导，并更新 World State Root 的值，从而影响到Chain链上数据。目前，Storage 层的数据读取和修改是在执行相关 Transaction 的时候，通过调用 EVM 中的两个专用的指令 *OpSload* 和 *OpSstore* 来执行的。

在 `go-ethereum` 中，变量对应的存储层的 Slot，是按照其在在合约中的声明顺序，从第一个 Slot（position：0）开始分配的。

对于固定长度的变量，其值的所占用的 Slot 的位置在合约初始化开始的时候就已经分配的。即使变量只是被声明还没有真正的赋值，保存其值所需要的 Slot 也已经被 EVM 分配完毕。而不是在第一次进行变量赋值的时候，进行再对变量所需要的的 Slot 进行分配。

`Geth`会将两个小于 32 bytes 长的变量合并到一个 Slot 中，这样的做法节省了物理空间，但是也同样带来读写放大的问题。因为在 `Geth` 中，读操作最小的读的单位都是按照 `32bytes` 来进行的(由于`OpSload`指令的实际调用)。在本例中，即使我们只需要读取`isTrue`或者`addr`这两个变量的值，在具体的函数调用中，我们仍然需要将对应的 Slot 先读取到内存中。同样的，如果我们想修改这两个变量的值，同样需要对整个的 Slot 进行重写。这无疑增加了额外的开销。所以在 Ethereum 使用 32 bytes 的变量，在某些情况下消耗的 Gas 反而比更小长度类型的变量要小(例如 unit8)。这也是为什么 Ethereum 官方也建议使用长度为 32 bytes 变量的原因。

对于变长数组和 Map 结构的变量存储分配则相对的复杂。虽然 Map 本身就是 `key-value` 的结构，但是在 Storage 层并不直接使用 map 中 key 的值或者 key 的值的 sha3 哈希值来作为 Storage 分配的 Slot 的索引值。目前，EVM 首先会使用map中元素的key的值和当前Map变量声明位置对应的 slot 的值进行拼接，再使用拼接后的值的 `keccak256` 哈希值作为 Slot 的位置索引(Position)。

### 2025.06.19

学习MPT树的结构，但还没完全看懂，只是知道了个大概，明天继续研究。

### 2025.06.20

# 默克尔帕特里夏树（Merkle Patricia Tree）

默克尔帕特里夏树（Merkle Patricia Tree）简称MPT树，MPT树结合了字典树和默克尔树的优点，在压缩字典树中根节点是空的，而MPT树可以在根节点保存整棵树的哈希校验和，而校验和的生成则是采用了和默克尔树生成一致的方式。

以太坊采用MPT树来保存，交易，收据以及世界状态，为了压缩整体的树高，降低操作的复杂度，以太坊又对MPT树进行了一些工程上的优化。将树节点分成了四种；
- 空节点（hashNode）
- 叶子节点（valueNode）
- 分支节点（fullNode）
- 扩展节点（shortNode）

通过以太坊黄皮书中很经典的一张图，来了解不同节点的具体结构和作用

![](./img/wynn_1.png)

可以看到有四个状态要存储在世界状态的MPT树中，需要存入的值是键值对的形式。自顶向下，我们首先看到的`keccak256`生成的根哈希，参考默克尔树的`Top Hash`，其次看到的是绿色的`扩展节点Extension Node`，其中`共同前缀shared nibble`是a7，采用了压缩前缀树的方式进行了合并，接着看到蓝色的`分支节点Branch Node`，其中有表示十六进制的字符和一个value，最后的value是fullnode的数据部分，最后看到紫色的`叶子节点leadfNode`用来存储具体的数据，它同样对路径进行了压缩。

以太坊的智能合约为例，智能合约部署成功后会得到一个地址，比如是`0xa16de3199ca3ee1bc1e0926d53c12ffacdf3f2c4`，除去开头的`0x`表示这是一个十六进制的字符串的标识，把其余字符按照压缩前缀树的规则存入MPT树，键（Key）就是合约的地址，键对应的值（Value）就是合约账户的信息，其中有合约代码和合约状态。合约中的状态变量又会做为一个键值对存入合约每个合约自己的MPT树中。相当于合约自己的MPT树又通过一个MPT树来进行组织。如果账户是外部账户，没有合约代码和合约状态，就直接存储了账户余额。而组织这一切的MPT树就叫世界状态树。世界状态树只需要存储每个合约状态树的根哈希，当合约状态变更，合约状态树的根哈希发生变化，传导到世界状态树，世界状态树的根哈希发生变化，从而让外部感知到世界状态发生了修改。

> 需要持久化的数据都是以键值对的形式存在的，而键是由keccak256这个函数计算得到，用十六进制表示后刚好对应MPT 树中分支节点的0-f的16个分支。

最终的数据还是以键值对的形式存储在LevelDB中的，MPT树相当于提供了一个缓存，帮助我们快速找到需要的数据。

## MPT树持久化
在Go实现的以太坊中，MPT树最终是存储在LevelDB数据库中的，LevelDB是Google开源的持久化KV单机数据库，具有很高的随机写，顺序读/写性能。以太坊通过对叶子节点按照一定的编码规则编码后存入LevelDB。

以太坊的MPT树提供了三种不同的编码方式来满足不同场景的不同需求，三种编码方式为；
- Raw编码（原生字符）
- Hex编码（扩展16进制编码）
- Hex-Prefix编码（16进制前缀编码）

三者的关系如下图所示，分别解决的是MPT对外提供接口的编码，在内存中的编码，和持久化到数据库中的编码。

![](./img/wynn_2.png)

### Raw编码
MPT对外提供的API采用的就是Raw编码方式，这种编码方式不会对key进行修改，如果key是“foo”, value是"bar"，编码后的key就是["f", "o", "o"]。

假设我们要把`a`作为key放入MPT树，key可以直接用`a`的ASCII表示97就可以了。

### Hex编码
可以发现采用Raw编码以后，从a-z一共26个字母，如果采用`分支节点（BranchNode）`存储的话需要26个空间，如果再加上0-9一共10个数字和一个value，总共需要37个空间，以太坊的开发者权衡了一下觉得太多了，于是就改良了编码方式，有了Hex编码。

以太坊先定义了一个新单位`nibble`，一个`nibble`表示4个bit，0.5个byte。然后按照如下规则编码；
- 将Raw输入的每个字符（1byte）拆分成2个nibble，前4位和后4位各一个nibble；
- 将每个nibble扩展为1个byte（8个bit）；
- 然后分别将Raw编码后的十六进制结果的每个`b`进行如下操作
    - b / 16；
   - b % 16；

有疑惑的话看一下这个例子就懂了
```
a的ASCII编码为97（十进制），转换十六进制为61
采用Hex编码
[0] = 61 / 16 = 3
[1] = 61 % 16 = 13
编码后的结果 [3, 13]

在举个例子
输入"cat"，Raw编码后 [63, 61, 74]
63 / 16  = 3
63 % 16 = 15
61 / 16 = 3
61 % 16 = 13
74 / 16 = 4
74 % 16 = 10
编码后的结果 [3，15，3，13，4，10]
```
树的最后一位value是一个标识符，如果存储的是真实的数据项，即该节点是叶子节点，则在末尾添加一个ASCII值为16的字符作为终止标识符。添加后的结果是` [3，15，3，13，4，10, 16]`

采用Hex编码以后，可以看到原本需要的37个空间存储的消耗都被压缩到了17个空间，横向压缩，但是增加了纵向空间的消耗，是一种工程的妥协。根据Key的数量多少，压缩与否各有优劣。

### HP编码（Hex-Prefix Encoding 16进制前缀编码）
前面介绍的Hex编码后的数据是在内存中的，如果要对Hex编码后的数据进行持久化，就会发现一个问题，我们对原数据进行了扩展，本来1个byte的数据被我们变成了2个byte，显然这对于存储来说是不可接受的，于是就又有了HP编码。

HP编码的过程如下；
- 输入key（Hex编码的结果）如果有标识符，则去掉这个标识符。
- key的头部填充一个nibble，填充的规则如下
    - 如果key的nibble长度是偶数则最后一位0
    - 如果key的nibble长度是奇数则最后一位1
    - 如果key是`扩展节点`则倒数第二位是0
    - 如果key是`叶子节点`则倒数第二位是1

例子：nibble长度是奇数的扩展节点填充为0001。

举个例子：
```
"cat"经过HP编码后的结果 [3，15，3，13，4，10, 16]
再用HP编码
1. 去掉16，同时表明这个是叶子节点。
2. 叶子节点，nibble数量是奇数个，这两个条件得出需要填充的值为 0010 0000
3. 将HP编码后的结果用二进制表示 [0010, 0000，0011, 1111, 0011, 1101, 0100, 1010]
4. 将HP编码后的结果合并成byte，为[00100000, 00111111, 00111101, 01001010]转换为十进制是[32, 63, 61, 74]
```

相较与cat的Raw编码，经过上面的Hex编码和HP编码，既可以能在内存中构建出MPT树，又可以尽可能减小存储所占用的空间，不得不说以太坊设计的巧妙。

### 安全的MPT

在上面介绍三种编码并没有解决一个问题，如果我们的key非常的长，会导致树非常的深，读写性能急剧的下降，如果有人不怀好意设计了一个独特的key甚至是可以起到DDOS攻击的作用，为了避免上面的问题，以太坊对key进行了一个特别的操作。

将所有的key都进行了一个`keccak256(key)`的操作，这样就保证了所有key的长度都为一致定长。

### 持久化MPT

MPT树节点Key的三种编码形式，但是这三种编码都是对key进行的操作，最终持久化到LevelDB中是k-v的形式，还需要对value进行处理。

在以太坊存储键值对之前会采用RLP编码对键值对进行转码，将键值对编码后作为value，计算编码后数据的哈希（keccak256）作为key，存储在levelDB中。

在具体的实现中，为了避免出现相同的key，以太坊会给key增加一些前缀用作区分，比如合约中的MPT树，会增加合约地址，区块数据会增加表示区块的字符和区块号。
MPT树是以太坊非常非常核心的数据结构，在存储区块，交易，交易回执中都有用到。

## MPT树应用
以太坊区块中有三颗MPT树，分别是状态树，交易树，收据树 ，分别存储了以太坊中的世界状态，本区块的交易，本区块的交易回执，其中交易和交易回执是一一对应的。

### 状态树
当以太坊每次执行一笔交易的时候，以太坊对应的世界状态都会发生改变。以太坊作为一个公链平台，上面有大量合约所对应的海量状态，如果每次发生状态改变都重建状态树无疑会消耗巨大的计算资源。为了解决这个问题，以太坊提出了增量修改状态树的方法。每次以太坊状态发生改变后并不会去修改原来的MPT树，而是会新建一些分支，如下图所示；

![](./img/wynn_3.png)

以太坊每次状态发生改变后，只会影响到世界状态MPT树的少量节点，新生成的世界状态树只需要重新计算受到影响的少量节点和与之相连节点的哈希即可。

以太坊的状态非常适用于用MPT树来存储，MPT树的特点与以太坊状态的特点非常契合，当采用MPT树存储以太坊状态以后可以带来如下的优势；

- 当一个账户的余额发生改变后，对应路径的哈希也发生了变化，然后自底而上的更新对应路径上的哈希值，直至Satet Root，这样可以计算最少的哈希次数。
- 以太坊中的全节点维护的是增量的MPT状态树，因为每次一个区块对世界状态的修改都只是很小的一部分，增量修改既有利于区块回滚，又可以节约开销。
- 在以太坊中区块临时分叉很普遍，但是由于以太坊智能合约的复杂性，如果不记录原始状态，很难根据合约代码回滚状态。

### 收据树
以太坊在智能合约执行时会产生一个交易回执（Receipt）记录了此笔交易的执行结果，交易信息和区块信息。

当查询轻节点查询通过布隆过滤器找到交易后，为了避免误识，还会再次查询回执来避免误识。

### 交易树
交易树的作用是提供了交易的默克尔证明，证明某个交易被打包到某个区块里，轻节点不用存储区块体仅根据提供的默克尔证明就可以快速判断交易是否已经被打包。

### 三颗树的差异
交易树和收据树只依赖当前的区块，而状态树是把链上所有状态都包含进去，交易树和收据树是独立的，状态树会共享树的节点。

为什么状态树要包含所有链上所有的状态呢？

举个例子，当一笔转账操作的发起时，需要判断发起账户是否有足够的ETH来完成这笔转账，这个时候要通过查找状态树查看对应账户的状态，但是如果为了节约空间，只保存了当前区块账户的状态，就需要逐块查找，非常影响性能，甚至这个转账交易的发起者都不存

<!-- Content_END -->
