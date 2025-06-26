---
timezone: UTC+8
---

> 请在上边的 timezone 添加你的当地时区(UTC)，这会有助于你的打卡状态的自动化更新，如果没有添加，默认为北京时间 UTC+8 时区


# Rhyne Hsu

1. 自我介绍
   - [Rhyne Hsu](https://www.linkedin.com/in/yuanzhen-hsu) / [Sasaki](https://x.com/AntiSasaki)，一个兴趣使然的web3爱好者，希望把自己的技术锻炼得更solid！
3. 你认为你会完成本次残酷学习吗？
   - 会滴！🐛🐛🐛
4. 你的联系方式（推荐 Telegram）
   - TG：@AntiSasaki

## Notes

<!-- Content_START -->

### 2025.06.16
---
#### 📗Basic
- 2014年测试网上线，2015年主网上线
- $1\ eth = 10^{18}\ Wei = 10^9\ Gwei$
- 由比特币的UTXO模型转变为账户模型，可以减少空间消耗，账户以 *modified Merkle Patricia Trie (MPT)* 存储（一种能提供根hash证明的压缩前缀树）
#### 📗Architecture
> The main challenge a consensus protocol aims to solve is building a reliable distributed system on top of unreliable infrastructure.

由共识层和执行层两侧组成
- 共识层负责PoS共识
- 执行层负责实际的链上状态更新
- 共识层和执行层执行通过Engine APIs沟通

**共识层CL**，即信标链 (Beacon Chain)
- 09/15/2022 The Paris hard fork (**The Merge**)
	- 由一个规定的 *TTD (Total Terminal Difficulty)* 触发，一旦有区块的TD (Total Difficulty) 达到TTD，就会终止PoW区块的生成
	- 在PoW阶段，共识是基于最重链，即有最多的 TD 的链
	- 一般的协议更新的trigger都是block height，而这里采用TTD trigger，原因是：如果采用block height，恶意节点可以用小部分算力拓展非最重链来抢先达到规定的block height从而终止所有PoW区块的生成，实现攻击；而如果用TTD trigger则不会受到这样的攻击
	- **Bellatrix Update** 是为The Merge做准备的共识层更新，它规定了CL和EL客户端必须遵循下面的结构。CL客户端必须包含Bellatrix更新；EL客户端必须包含Paris更新并暴露 Engine API 接口
		![client-architecture](https://epf.wiki/wiki/protocol/img/clients-overview.png)
	- **Paris Update** 则是相应的执行层更新
	- [reference](https://etherworld.co/2022/07/20/what-do-bellatrix-paris-ttd-mean-in-ethereum-merge-upgrade/)
- 以太坊的共识机制 (**Gasper**) 实际上由两种共识机制耦合而成：**LMD GHOST** & **Casper FFG**
- Validators
	- 每个验证者最多质押32 ETH，更多质押并不会增加其权重。超过32 ETH的，需要运行更多验证者来实现（一个验证者客户端可以管理多个验证者）
	- 同时，要成为验证者，要求的最少质押也是 32 ETH
- Slots and Epochs
	- **12秒**一个slot，**32 slots**一个epoch，即约 6.4 分钟一个epoch
	- 这要求验证者节点在时间上要基本同步
	- 一个slot是产生区块的一次**机会**，也就是说一个slot不一定会产生一个区块
	- 网络中所有validator在每个epoch中会被均分到各个slots中，每个slot内部再划分committee
	- 每个slot会根据 RANDAO 发射的随机数，伪随机地选出该slot的committees（每个committee至少128个验证者）
	- 每个slot只有对应的committees可以进行投票见证
	- 一个slot中可以有多个commitees（一般都有多个）
	- 同时选出一个validator，由他提出新区块提案，其他validator则对其进行见证 (attest)
	- attestation会在validator之间广播，validator彼此监督是否有人
	- slot中投票后，该区块得到的只是slot级别的通过，并没有被最终确认
	- 因为在每个epoch的最后还需要全网 2/3 的总stake进行投票
- Beacon：由一个名叫 **RANDAO** 的伪随机过程发射随机数，来决定每个epoch中的每个slot的committees，这个RANDAO就像整条链的信标，这就是 *Beacon Chain* 名字的由来

**执行层EL**

#### 📋TODOs
- [ ] 整理所有历史更新的{名称、时间、主要内容}
- [ ] LMD GHOST
- [ ] Casper FFG

> END_OF_TODAY [BOOKMARK](https://epf.wiki/#/wiki/CL/overview?id=blobs)

### 2025.06.17
---
#### 📗Consensus
**Blobs**：[EIP-4844](https://eips.ethereum.org/EIPS/eip-4844) 引入，是Cancun硬分叉的一部分。允许每个区块携带3-6个 Blob sidecars，相当于为网络添加了一个数据层 (*data accessibility layer*)
- 每个Blob的保留期限是 4096 epochs，过期即会drop掉
- KZG commitment

**Finalization**：
- validator有两种投票：LMD GHOST 投票给普通区块，Casper FFG投票给checkpoints
- validator只能给自己对应的slot投LMD GHOST票，而投票checkpoints时，所有validator都可以投Casper FFG票，这个是一个epoch中随时可以投的
- 并且validator也可以延迟给slot投票，不一定非要在自己的slot立即投票
- 当然，立即投票的奖励是最高的，延迟投票会减少奖励
- checkpoint是每个epoch的第一个区块
- 每个 Casper FFG 投票，需要指定 
	1. 其源checkpoint：以往epoch的checkpoint
	2. 目标checkpoint：当前epoch的checkpoint
- checkpoint获得超 2/3 的绝对多数投票Casper FFG后，将会被 *justified*
- checkpoint被 *justified* 之后，其前一个checkpoint才会被 *finalized*
- 也就是说一个checkpoint被finalized大概需要2个epoch的时间（12.8分钟）

**Slashable Offenses**
- Double Proposal
- LMD GHOST Double Vote
- FFG Surround vote
- FFG Double Vote

**validator** 初始需要32个ETH才能激活，当其质押金额少于16ETH时会失活
- 需要服务2048个epoch之后才能主动退出
- 提出退出后，需要4个epoch的冷却期才能退出，用于留给slash的时间窗口
- 同样的，提起激活前也需要有4个epoch的窗口期
- 退出后，未被slash的在 $2^8$ epoch（27h）之后可以提款；而被slash的则需要 $2^13$ epoch （36天）才能退款
- 每个epoch能激活和失活的validator数量有一个规定的阈值
	![validator-lifecycle](https://epf.wiki/images/cl/validator-lifecycle.png)

> <END_OF_TODAY> [BOOKMARK](https://epf.wiki/#/wiki/CL/cl-architecture)

### 2025.06.18
---
#### 📗Consensus Layer
- LMD GHOST 提供 liveness，Casper FFG 提供 finality
> In essence, LMD GHOST keeps the chain moving forward, while Casper FFG ensures stability by finalizing blocks.

- **CAP Theorem**：一个分布式系统不可能同时满足 Consistency, Availability, Partition tolerance
- 一个新区块是如何产生的
	1. 当前slot的block proposer的EL client执行`create block`指令
	2. EL获取mempool中的交易
	3. EL打包交易并执行，生成新区块的hash
	4. CL客户端将交易集合和区块hash放入beacon block（CL的区块），并在网络中广播
	5. 其他validator收到广播区块
	6. 将交易作为 *execution payload* 发送到EL，执行交易并生成hash
	7. 验证hash后，将该区块添加到CL中，完成见证（attest）并广播 *attestation*
- 一个新区块的心路历程：proposed -> attested -> justified -> finalized
- Consensus Client的技术栈
	- libp2p：p2p协议
	- discv5：peer discovery
	- libp2p-noise：加密
	- [SSZ](https://ethereum.org/en/developers/docs/data-structures-and-encoding/ssz/) (Simple SerialiZe)：取代了 RLP的新的编码/序列化算法
	> [RLP](https://ethereum.org/en/developers/docs/data-structures-and-encoding/rlp/) (Recursive-Length Prefix)：曾是以太坊EL的序列化算法

	- (snappy)[https://en.wikipedia.org/wiki/Snappy_(compression)]：压缩算法，保证合理的压缩比的前提下优先压缩/解压速度
- 共识层规范 Consensus Layer *Specification*：[Pyspec](https://github.com/ethereum/consensus-specs) 是为共识层开发者提供的执行规范
- Clients
	- Consensus Client: 运行以太坊PoS共识算法
	- Execution Client: 参与交易的验证和广播，执行状态转移
	- Validator Client: 进行 attest 和 propose 新区块（validator client时 consensus client的一种可选的附加组件）

> <END_OF_TODAY> [BOOKMARK](https://epf.wiki/#/wiki/CL/cl-clients)

### 2025.06.19
---
目前的CL Client种类

|Client|Language|Developer|Status|
|---|---|---|---|
|[Lighthouse](https://github.com/sigp/lighthouse)|Rust|Sigma Prime|Production|
|[Lodestar](https://github.com/ChainSafe/lodestar)|TypeScript|ChainSafe|Production|
|[Nimbus](https://github.com/status-im/nimbus-eth2)|Nim|Status|Production|
|[Prysm](https://github.com/prysmaticlabs/prysm)|Go|Prysmatic Labs|Production|
|[Teku](https://github.com/ConsenSys/teku)|Java|ConsenSys|Production|
|[Grandine](https://github.com/grandinetech/grandine)|Rust|Grandine Developers|Production|
|[Caplin](https://github.com/ledgerwatch/erigon)|Go|Erigon|Development|
|[LambdaClass](https://github.com/lambdaclass/lambda_ethereum_consensus)|Elixir|LambdaClass|Development|

客户端种类的**多样性**对一个网络的安全和鲁棒至关重要，因为单一客户端有出现问题的可能，如果网络中2/3的节点使用单一客户端，一旦该客户端软件出问题，网络就会崩溃。目前最主流的是 Lighthouse 和 Prysm

#### 📗SSZ
SSZ (Simple SerialiZation): 是专为以太坊设计的 1. 序列化 和 2. Merkleization 方案

- 为什么要序列化：将数据结构或对象转换为**可存储或可传输格式**，以便 
	1. 数据持久化 
	2. 网络传输
	3. 跨语言/平台兼容：为跨实现的节点（不同语言）提供统一通信格式
	4. 存储与效率优化：原始对象可能包含很多冗余信息，序列化可以压缩为精简格式，节省空间
	5. 哈希和签名：数据序列化为字节后，便于进行哈希、加密、签名等操作
- 无符号整数编码：Little-Endian Formt，低位byte排在地址前面（最低位），如`0x12345678` 编码为 `[0x78, 0x56, 0x34, 0x12]`
```python
class Example
    id: uint64,
    bytes: List[uint8, 64]
    next: uint64
```
```
# serialize(my_example)
#
# Note: this is a single byte-array split over four lines for readability.
[
  42, 0, 0, 0,  # The little-endian encoding of `id`, 42.
  12, 0, 0, 0,  # The "offset" that indicates where the variable-length value of `bytes` starts (little-endian 12).
  43, 0, 0, 0,  # The little-endian encoding of `next`, 43.
  0, 1, 2       # The value of the `bytes` field.
]
```

> <END_OF_TODAY> [BOOKMARK](https://epf.wiki/#/wiki/CL/SSZ)

### 2025.06.20
---
- SSZ具体各种类型的数据的序列化方法： https://epf.wiki/#/wiki/CL/SSZ
- Merkleization: 指将序列化后的数据构造为Merkle树并得到其根hash的过程
- 弱主观性 *weak subjectivity*
	- 客观：信息的正确性完全可验证
	- 主观：信息的正确性并不能完全可验证
- *Full Syncing*：指PoW时期，所有合法区块都一定可以完美追溯到Genesis块
	- 然后转入PoS之后，由于finality来自于CL而非EL
	- 要断言一个finalized的checkpoint的正确性，必须要额外的验证
- validator退出CL之后，将不能再为未来的区块投票，但仍可以对过去的区块重新投票
- Beacon block syncing 和 Execution block Syncing 的区别
	1. Beacon chain的syncing是反方向的
	2. Beacon chain的信任锚点是主观的：Beacon chain的同步目标是一个“最终确定”区块，但它不能被完全客观验证，只能部分验证其正确性。因此，这个检查点需要通过**链外途径（out-of-band）**获取
	3. 弱主观性周期与乐观导入：在 Execution 层尚未完成同步时，Beacon 链会先 **乐观导入（optimistically import）** 新块，但暂不验证其 Execution Payload
- **Weak Subjectivity Sync** 步骤
	1. Obtain a weak subjectivity checkpoint out-of-band （因为p2p层中的peer是不可信的）
	2. Backfill blocks all the way back to genesis from the weak subjectivity checkpoint
	3. Update the target header for the execution chain
	4. Optimistically follow the head of the chain and continuously update the target header for the execution chain
	5. Once the EL is synced, then mark the CL slots as verified post-verification. The node may now be considered fully synced

> <END_OF_TODAY>

### 2025.06.23
---
#### 📗Geth
> 最popular的 EL 客户端</br>
[source code](https://github.com/ethereum/go-ethereum.git)</br>
[Geth 源码系列 I：Geth 整体架构](https://forum.lxdao.io/t/geth-i-geth/2856)

执行层的六个部分
1. EVM：状态机的状态转移函数。负责执行交易，交易执行也是修改状态数的唯一方式
2. 存储
3. mempool
4. p2p网络
5. RPC服务：提供访问节点的能力，如用户向节点发送请求、CL和EL之间的交互
6. Blockchain

EL node 的三种可能状态及需要做的事
1. 新加入网络：需要通过p2p网络获取区块和状态数据
	- Full Sync：从 genesis block 逐个下载、验证、重建状态数据库
	- Snap Sync：直接下载最新 checkpoint 的状态数据和以后的区块数据
2. 已同步到最新状态，非 proposer：
	- CL 持续通过 Engine API 从共识层获取到当前最新产出的区块
	- CL client 将交易作为 *execution payload* 发送到EL
	- EL 执行交易完成状态转移，并生成hash验证区块
	- 更新状态数据库、本地区块链
	- 将该区块添加到CL中，完成见证（attest）并广播 *attestation*
3. 已同步到最新状态，选为 proposer：
	- CL驱动EL client执行`create block`指令
	- EL获取mempool中的交易
	- EL打包交易并执行，生成新区块的hash
	- CL客户端将交易集合和区块hash放入beacon block（CL的区块），并在网络中广播
- `eth/backend.go` Ethereum核心抽象
``` go
type Ethereum struct {
	config         *ethconfig.Config  // 以太坊配置，包括链配置
	txPool         *txpool.TxPool  // 交易池，用户的交易提交之后先到交易池
	localTxTracker *locals.TxTracker  // 用于跟踪和管理本地交易（local transactions）
	blockchain     *core.BlockChain  // 区块链结构
	handler *handler  // 是以太坊节点的网络层核心组件，负责处理所有与其他节点的通信，包括区块同步、交易广播和接收，以及管理对等节点连接
	discmix *enode.FairMix  // 负责节点发现和节点源管理
	chainDb ethdb.Database  // 负责区块链数据的持久化存储
	eventMux       *event.TypeMux  // 负责处理各种内部事件的发布和订阅
	engine         consensus.Engine  // 共识引擎
	accountManager *accounts.Manager  // 管理用户账户和密钥
	filterMaps      *filtermaps.FilterMaps  // 管理日志过滤器和区块过滤器
	closeFilterMaps chan chan struct{}  // 用于安全关闭 filterMaps 的通道，确保在节点关闭时正确清理资源
	APIBackend *EthAPIBackend  // 为 RPC API 提供后端支持
	miner    *miner.Miner  // 在 PoS 下，与共识引擎协作验证区块
	gasPrice *big.Int  // 节点接受的最低gas价格
	networkID     uint64  // 网络 ID
	netRPCService *ethapi.NetAPI  // 提供网络相关的 RPC 服务，允许通过 RPC 查询网络状态
	p2pServer *p2p.Server  // 管理P2P网络连接，处理节点发现和连接建立并提供底层网络传输功能
	lock sync.RWMutex  // 保护可变字段的并发访问
	shutdownTracker *shutdowncheck.ShutdownTracker  // 跟踪节点是否正常关闭，在异常关闭后帮助恢复
}
```
- geth 节点启动入口：`cmd/geth/main.go`

- 实战教程: [goethereumbook](https://goethereumbook.org/en/)

### 2025.06.24
---
#### 📗Effective Go
今天学习 [Effective Go](https://go.dev/doc/effective_go#commentary)，为 Geth 实战打好语言基础

#### 📗Basic
- go有一个default的官方指定的程序化的格式调整工具 `gofmt`，每次 `CTRL + s` 保存的时候会自动调整格式为理想格式，主要优化缩进和空格
- 命名惯例
	- 变量、函数都用驼峰命名法，首字母小写
	- 要export的函数，首字母大写
- 非常适用于高性能分布式系统，提供了很多并行的支持，适合开发游戏服务端等
- 第一行为`package <pkg_name>`，指定该文件属于哪个包
	- 每个Go程序都必须有一个main包
	- 按照约定，包名与导入路径的最后一个元素一致。例如，"math/rand" 包中的源码均以 package rand 语句开始
- import 来引入包
	- Go中，如果被import的包没有被调用，编译器是会报错的
	- 也可以用分组的形式导入
	``` go
	import (
		"fmt"
		"math"
	)
	```
- 大写字母开头的函数、变量等，是可以导出包外的，相当于public；小写开头的则只在包内可见，相当于protected
	- 所以说调用包的方法和变量时，其名字一定是大写打头的
- **不需要分号结尾**，同python
- 变量都由关键字`var`声明
	- 可以在变量名后显式地声明变量类型 `var a string = "Go"`
	- 也可以用 `:=` 来直接声明变量
	- go中的自动类型推断是在编译时进行的，而python是在运行时进行的
		- 函数外的每个语句都必须以关键字开始（var、func 等），因此`:=`结构不能在函数外使用。
	- 可以同时在一行为多个不同类型的变量赋值：`var c, python, java = true, false, "no!"`
- `const` 用来声明常量，有了const就不用再var了
	``` go
	const (
		a = iota  // 0
		b         // 1
		c         // 2
		d = 5     // 5
		e         // 5
	)
	```
	- `iota` 从0开始，每过一行自增1，到下一个硬编码的变量时中止
	- 中止之后，未被赋值的变量默认等于上一个变量的值
- 可以一行声明和初始化多个变量
- `func`来声明函数：`func function_name( [parameter list] ) [return_types]`
	- 函数可以返回任意数量的返回值
	- 可以在函数定义中声明返回值的名字 `func split(sum int) (x, y int) {}`
- go中也有类似c的指针，但是没有指针运算
	- 结构体的指针也可以直接用点号 `.` 来取成员
- go没有类，但是可以为类型定义方法
	- `func (s *Stack) Push(item int)` 函数名之前指定类名和类指针变量名，来为类型实现方法
- 局部变量在声明之后必须被用到，否则会报编译错误
- 可以同时赋值 `a, b = b, a`
- `make()`只用于创建三种对象：channel, slice, map
	- make返回对象的引用，new返回指针
	- `make([]int, len)` 来创建一个长度为len的数组（自动初始化为0）
- go主要就是玩slice
- go没有tuple

#### 📗Syntax
- go中没有前置自增运算符 (`++i`)，但是有后置的 (`i++`)
	- 不过go中的自增是没有返回值的，也就是不能用自增表达式给别人赋值
- if语句的条件不需要用括号括起来，类似于rust
- 和 for 一样，if 语句可以在条件表达式前执行一个简短语句。 `if v := math.Pow(x, n); v < lim {}`
	- 在这个简短语句中声明的变量在整个if和else语句块中都可见
- for语句条件与c类似，只是不用括号
	- `for idx, char := range str {}` 用`:=`
	- range返回的是一个**enumerate**，如果第一个值默认被赋值为idx
	- 所以想要实际的值，而不是index的话，需要用`_`接收掉index：`for _, val := range arr {}`
- for后面可以只包括条件语句，这时for与c中的while相等 `for i < 100 {}`
- 也可以什么都不带，这时等于rust中的loop `for {}`
- `defer` 推迟，go特有的关键字，表示把后面的语句放在当前函数返回之后再执行
	- 推迟调用的函数被压入栈中，调用时先进后出
	- 用来关闭文件、释放锁很方便 
	``` go
	file, _ = os.Open(path)
	defer func(){file.Close()}
	```
- 定义函数时，在`func`和函数名之间可以有一个接收者Receiver：`func (h IntHeap) Len() int`
	- 接收者可以是值接收者(value receiver)或指针接收者(pointer receiver)
	- 值接收者时，函数中h是原来的h的一个clone，在函数中，如果有人在其他地方改了h，不会影响这个函数，适合不改变该对象的值的函数
	- 指针接收者时，函数中h就是原来的h的指针，对`*h`的修改会更改原本的值

> <END_OF_TODAY>

<!-- Content_END -->
