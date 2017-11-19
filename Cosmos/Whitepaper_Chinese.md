# Cosmos
**A Network of Distributed Ledgers**

**分布式账本网络**

Jae Kwon jae@tendermint.com<br/>
Ethan Buchman ethan@tendermint.com

For discussions, [join our Slack](http://forum.tendermint.com:3000/)!
讨论[请加入Slack](http://forum.tendermint.com:3000/)!

_NOTE: If you can read this on GitHub, then we're still actively developing this
document.  Please check regularly for updates!_

_注意：如果你能在github上阅读，我们仍然会积极地更新这个文档，请定期检查更新！_

## Table of Contents ###########################################################
  * [Introduction](#introduction)
  * [Tendermint](#tendermint)
    * [Validators](#validators)
    * [Consensus](#consensus)
    * [Light Clients](#light-clients)
    * [Preventing Attacks](#preventing-attacks)
    * [ABCI](#abci)
  * [Cosmos Overview](#cosmos-overview)
    * [Tendermint-BFT](#tendermint-bft)
    * [Governance](#governance)
  * [The Hub and Zones](#the-hub-and-zones)
    * [The Hub](#the-hub)
    * [The Zones](#the-zones)
  * [Inter-blockchain Communication (IBC)](#inter-blockchain-communication-ibc)
  * [Use Cases](#use-cases)
    * [Distributed Exchange](#distributed-exchange)
    * [Bridging to Other Cryptocurrencies](#bridging-to-other-cryptocurrencies)
    * [Ethereum Scaling](#ethereum-scaling)
    * [Multi-Application Integration](#multi-application-integration)
    * [Network Partition Mitigation](#network-partition-mitigation)
    * [Federated Name Resolution System](#federated-name-resolution-system)
  * [Issuance and Incentives](#issuance-and-incentives)
    * [The Atom Token](#the-atom-token)
      * [Fundraiser](#fundraiser)
    * [Limitations on the Number of
    Validators](#limitations-on-the-number-of-validators)
    * [Becoming a Validator after Genesis
    Day](#becoming-a-validator-after-genesis-day)
    * [Penalties for Validators](#penalties-for-validators)
    * [Transaction Fees](#transaction-fees)
    * [Incentivizing Hackers](#incentivizing-hackers)
  * [Governance Specification](#governance-specification)
    * [Parameter Change Proposal](#parameter-change-proposal)
    * [Text Proposal](#text-proposal)
  * [Roadmap](#roadmap)
  * [Related Work](#related-work)
    * [Consensus Systems](#consensus-systems)
      * [Classic Byzantine Fault Tolerance](#classic-byzantine-fault-tolerance)
      * [BitShares Delegated Stake](#bitshares-delegated-stake)
      * [Stellar](#stellar)
      * [BitcoinNG](#bitcoinng)
      * [Casper](#casper)
    * [Horizontal Scaling](#horizontal-scaling)
      * [Interledger Protocol](#interledger-protocol)
      * [Sidechains](#sidechains)
      * [Ethereum Scalability Efforts](#ethereum-scalability-efforts)
    * [General Scaling](#general-scaling)
      * [Lightning Network](#lightning-network)
      * [Segregated Witness](#segregated-witness)
  * [Appendix](#appendix)
    * [Fork Accountability](#fork-accountability)
    * [Tendermint Consensus](#tendermint-consensus)
    * [Tendermint Light Clients](#tendermint-light-clients)
    * [Preventing Long Range Attacks](#preventing-long-range-attacks)
    * [Overcoming Forks and Censorship
    Attacks](#overcoming-forks-and-censorship-attacks)
    * [ABCI Specification](#abci-specification)
    * [IBC Packet Delivery
    Acknowledgement](#ibc-packet-delivery-acknowledgement)
    * [Merkle Tree &amp; Proof
    Specification](#merkle-tree--proof-specification)
    * [Transaction Types](#transaction-types)
      * [IBCBlockCommitTx](#ibcblockcommittx)
      * [IBCPacketTx](#ibcpackettx)
  * [Acknowledgements](#acknowledgements)
  * [Citations](#citations)

## <P1-Reviewed>Introduction | 介绍 ################################################################

The combined success of the open-source ecosystem, decentralized
file-sharing, and public cryptocurrencies has inspired an understanding that
decentralized internet protocols can be used to radically improve socio-economic
infrastructure.  We have seen specialized blockchain applications like Bitcoin
[\[1\]][1] (a cryptocurrency), Zerocash [\[2\]][2] (a cryptocurrency for
privacy), and generalized smart contract platforms such as Ethereum [\[3\]][3],
with countless distributed applications for the Ethereum Virtual Machine (EVM) such as Augur (a prediction
market) and TheDAO [\[4\]][4] (an investment club).

开源的生态系统、去中心化的文件共享、以及公共的加密货币，这一系列技术的成功使人们启发和理解，去中心化的互联网协议是可以从根本上改善社会经济基础架构的。我们已经见识过个有专长的区块链应用，诸如比特币[\[1\]][1]（加密货币），ZCASH [\[2\]][2] (隐私加密货币)，也看到了例如以太坊 [\[3\]][3] 的大众智能合约平台，还有无数基于 EVM (以太坊虚拟机)开发的分布式应用，例如 Augur（预测市场）和 TheDAO [\[4\]][4] （投资俱乐部）

To date, however, these blockchains have suffered from a number of drawbacks,
including their gross energy inefficiency, poor or limited performance, and
immature governance mechanisms.  Proposals to scale
Bitcoin's transaction throughput, such as Segregated-Witness [\[5\]][5] and
BitcoinNG [\[6\]][6], are vertical scaling solutions that remain
limited by the capacity of a single physical machine, in order to ensure the
property of complete auditability.  The Lightning Network [\[7\]][7] can help
scale Bitcoin transaction volume by leaving some transactions off the ledger
completely, and is well suited for micropayments and privacy-preserving payment
rails, but may not be suitable for more generalized scaling needs.

然而，迄今为止，这些区块链已经暴露了各种缺陷，包括总体能效低下，性能不佳或受到限制和缺乏成熟的治理机制。为了扩大比特币交易吞吐量，已经研发了许多诸如隔离见证 [\[5\]][5]（Segregated-Witness）和BitcoinNG [\[6\]][6]（一种新的可扩展协议）这样的解决方案，但这些垂直扩展解决方案仍然受到单一物理机容量的限制，以确保完整的可审计性。闪电网络 [\[7\]][7] 可以通过部分交易完全记录在主链账本外来扩展比特币的交易容量，这种方法十分适用于微支付和隐私保护支付通道，但是无法适用于更通用的扩展需求。

An ideal solution is one that allows multiple parallel blockchains to
interoperate while retaining their security properties. This has proven
difficult, if not impossible, with proof-of-work. Merged mining, for instance,
allows the work done to secure a parent chain to be reused on a child chain,
but transactions must still be validated, in order, by each node, and a
merge-mined blockchain is vulnerable to attack if a majority of the hashing
power on the parent is not actively merge-mining the child.  An academic review
of [alternative blockchain network
architectures](http://vukolic.com/iNetSec_2015.pdf) is provided for additional
context, and we provide summaries of other proposals and their drawbacks in
[Related Work](#related-work).

理想的解决方案是允许多个并行的区块链交互操作的同时保持其安全特性。事实证明，采用工作量证明很难做到这一点，但也并非不可能。例如合并挖矿，允许在工作完成的同时，确保母链在子链上被重复使用，但交易必须通过每个节点依次进行验证，而且如果母链上的大多数哈希算力没有积极地对子链进行合并挖矿，那么就容易遭受到攻击。关于[可替代区块链网络架构的学术回顾](http://vukolic.com/iNetSec_2015.pdf) 将在附件中展示，我们也会在[相关工作](#related-work)中对其他（技术）方案和缺陷进行概括。


Here we present Cosmos, a novel blockchain network architecture that addresses all
of these problems.  Cosmos is a network connecting many independent blockchains, called
zones.  The zones are powered by Tendermint Core [\[8\]][8], which provides a
high-performance, consistent, secure
[PBFT-like](http://tendermint.com/blog/tendermint-vs-pbft/) consensus engine,
where strict [fork-accountability](#fork-accountability) guarantees hold over
the behaviour of malicious actors.  Tendermint Core's BFT consensus algorithm is
well suited for scaling public proof-of-stake blockchains.  Blockchains with other consensus models, including proof-of-work blockchains like Ethereum and Bitcoin can be connected to the Cosmos network using adapter zones.

这里我们要介绍的 Cosmos，一个全新的区块链网络架构，能够解决所有这些问题。Cosmos 是由许多被称之为“分区”的独立区块链组成的网络。分区在 Tendermint Core [\[8\]][8]的支持下运行，Tendermint Core 是一个[类似拜占庭容错](http://tendermint.com/blog/tendermint-vs-pbft/)安全共识引擎，具有高性能、一致性的特性，并且在严格的[分叉追责](#fork-accountability) 机制下能够制止恶意破坏者的行为。Tendermint Core 的拜占庭容错共识算法十分适合用于扩展权益证明(PoS)机制下的公共区块链。使用其他共识模型的区块链, 包括类似基于权益证明(PoS)的以太坊，以及比特币也能够通过使用适配分区被 Cosmos 网络连接。

The first zone on Cosmos is called the Cosmos Hub. The Cosmos Hub is a
multi-asset proof-of-stake cryptocurrency with a simple governance mechanism
which enables the network to adapt and upgrade.  In addition, the Cosmos Hub can be
extended by connecting other zones.

Cosmos 的第一个分区称之为 Cosmos 枢纽。Cosmos 枢纽是一种多资产权益证明加密货币网络，它通过简单的治理机制能够对网络进行适配和升级。此外，Cosmos 枢纽可以通过链接其他分区来实现扩展。

The hub and zones of the Cosmos network communicate with each other via an
inter-blockchain communication (IBC) protocol, a kind of virtual UDP or TCP for
blockchains.  Tokens can be transferred from one zone to another securely and
quickly without the need for exchange liquidity between zones.  Instead, all
inter-zone token transfers go through the Cosmos Hub, which keeps track of the
total amount of tokens held by each zone.  The hub isolates each zone from the
failure of other zones.  Because anyone can connect a new zone to the Cosmos Hub,
zones allow for future-compatibility with new blockchain innovations.

Cosmos 网络的枢纽及各个分区可以通过区块链间通信（IBC）协议进行通信，这种协议就是针对区块链的虚拟用户数据报协议（UDP）或者传输控制协议（TCP）。代币可以安全、快速地从一个分区转到其他分区，而无需在两个分区之间拥具有汇兑流动性。相反，所有跨分区的代币转移都会通过 Cosmos 枢纽，以此来追踪记录每个分区持有代币的总量。这个枢纽会将每个分区与其他故障分区隔离开。因为每个人都可以将新的分区连接到 Cosmos 枢纽，所以分区将可以向后兼容新的区块链技术。

With Cosmos interoperability between blockchains can be achieved. The potential of an internet of value, where assets are issued and controlled by different sets of validators, yet can be moved and exchanged seamlessly between blockchains without relying on trusted third parties can be realized.

利用 Cosmos 可以实现区块链间的互操作。这是一个具有潜力的有价值的互联网络，其中的资产由不同的验证人发布和控制，并可以在不依靠需要信任的第三方的情况下实现跨链资产无缝的转移和交易。

## <P1-Reviewed>Tendermint ##################################################################

In this section we describe the Tendermint consensus protocol and the interface
used to build applications with it. For more details, see the [appendix](#appendix).

在这一部分我们将阐述Tendermint共识协议和用于建立其应用程序的接口。 更多信息，请参见[附录](#appendix)

### <P1-Reviewed>Validators | 验证人

In classical Byzantine fault-tolerant (BFT) algorithms, each node has the same
weight.  In Tendermint, nodes have a non-negative amount of _voting power_, and
nodes that have positive voting power are called _validators_.  Validators
participate in the consensus protocol by broadcasting cryptographic signatures,
or _votes_, to agree upon the next block.

在经典的拜占庭容错算法中，每个节点有相同的权重。在 Tendermint，节点有着不同数量（非负）的 _投票权_，而那些拥有相当数量投票权的节点称之为  _验证人_。验证人通过广播加密签名、投票或者对下一个区块表决同意来参与共识协议。

Validators' voting powers are determined at genesis, or are changed
deterministically by the blockchain, depending on the application.  For example,
in a proof-of-stake application such as the Cosmos Hub, the voting power may be
determined by the amount of staking tokens bonded as collateral.

验证者的投票权是一开始就确定好了，或者根据应用程序由区块链来决定修改投票权。例如，在像Cosmos 枢纽的权益证明应用里，投票权可由绑定为押金的代币数量来决定。

_NOTE: Fractions like ⅔ and ⅓ refer to fractions of the total voting power,
never the total number of validators, unless all the validators have equal
weight. >⅔ means "more than ⅔", ≥⅓ means "at least ⅓"._


注意：像⅔和⅓这样的分数指的是占总投票权的分数，而不是总验证人，除非所有验证人拥有相同权重。而>⅔ 的意思是"超过⅔ "，≥⅓则是"⅓或者更多"的意思。

### <P1-Reviewed>Consensus | 共识

Tendermint is a partially synchronous BFT consensus protocol derived from the
DLS consensus algorithm [\[20\]][20]. Tendermint is notable for its simplicity,
performance, and [fork-accountability](#fork-accountability).  The protocol
requires a fixed known set of validators, where each validator is identified by
their public key.  Validators attempt to come to consensus on one block at a time,
where a block is a list of transactions.  Voting for consensus on a block proceeds in
rounds. Each round has a round-leader, or proposer, who proposes a block. The
validators then vote, in stages, on whether to accept the proposed block
or move on to the next round. The proposer for a round is chosen
deterministically from the ordered list of validators, in proportion to their
voting power.

Tendermint 是部分同步运作的拜占庭容错共识协议，这种协议源自DLS共识算法 [\[20\]][20]。Tendermint以简易性、高性能以及[分叉问责制](#fork-accountability)而著称。协议要求这组验证人固定且被熟知，并且每个验证人都有其公钥验证身份。这些验证人试图同时在一个区块上达成共识，这些区块是一系列的交易记录。每个区块的共识轮流进行，每一轮都会有个领头人，或者提议人，由他们来发起区块。之后验证人分阶段对是否接受该区块，或者是否进入下一轮做出投票。每轮的提议人会从验证人顺序列表中按照其投票权比例来选择确定。

The full details of the protocol are described
[here](https://github.com/tendermint/tendermint/wiki/Byzantine-Consensus-Algorithm).

更多协议的全部细节，请点击[这里](https://github.com/tendermint/tendermint/wiki/Byzantine-Consensus-Algorithm).

Tendermint’s security derives from its use of optimal Byzantine fault-tolerance
via super-majority (>⅔) voting and a locking mechanism.  Together, they ensure
that:

* ≥⅓ voting power must be Byzantine to cause a violation of safety, where more
  than two values are committed.
* if any set of validators ever succeeds in violating safety, or even attempts
  to do so, they can be identified by the protocol.  This includes both voting
for conflicting blocks and broadcasting unjustified votes.

Tendermint 采用了使用大多数投票（超过三分之二）和锁定机制的最优拜占庭容错，来确保其安全性。这些能够保证：

* 蓄意破坏者想要造成安全性问题，必须有三分之一以上的投票权，并且要提交超过两份以上的值。
* 如果有一组验证人成功破坏了安全性，或者曾试图这么做，他们会被协议识别。协议包括对有冲突的区块进行投票和广播那些有问题的投票。

Despite its strong guarantees, Tendermint provides exceptional performance.  In
benchmarks of 64 nodes distributed across 7 datacenters on 5 continents, on
commodity cloud instances, Tendermint consensus can process thousands of
transactions per second, with commit latencies on the order of one to two
seconds.  Notably, performance of well over a thousand transactions per second
is maintained even in harsh adversarial conditions, with validators crashing or
broadcasting maliciously crafted votes.  See the figure below for details.

除了其超强的安全性外，Tendermint还具备杰出的性能。以商用型云平台为例，Tendermint共识以分布在五大洲七个数据中心的64位节点为基准，其每秒可以处理成千上万笔交易，订单提交延迟时间为1-2秒。而值得关注的是，即使是在极其恶劣的敌对环境中，比如验证人崩溃了或者是广播恶意破坏的投票，也能维持这种每秒超过千笔交易的较高性能。详见下图。

![Figure of Tendermint throughput performance](https://raw.githubusercontent.com/gnuclear/atom-whitepaper/master/images/tendermint_throughput_blocksize.png)

### <P1-Reviewed> Light Clients | 轻客户端

A major benefit of Tendermint's consensus algorithm is simplified light client
security, making it an ideal candidate for mobile and internet-of-things use
cases.  While a Bitcoin light client must sync chains of block headers and find
the one with the most proof of work, Tendermint light clients need only to keep
up with changes to the validator set, and then verify the >⅔ PreCommits
in the latest block to determine the latest state.

Tendermint 共识算法的主要好处是具有安全简易的客户端，使其成为手机和物联网用例的理想选择。比特币轻客户端必须同步运行区块头组成的链，并且找到工作量证明最多的那一条链，而Tendermint轻客戸端只需和验证组的变化保持一致，然后简单地验证最新区块中预先提交的>⅔，来确定最新情况。

Succinct light client proofs also enable [inter-blockchain
communication](#inter-blockchain-communication-ibc).

这种简单的轻客戸端证明机制也可以实现[区块链之间的通信](#inter-blockchain-communication-ibc)。

### <P1-Reviewed> Preventing Attacks | 防止攻击

Tendermint has protective measures for preventing certain notable
attacks, like [long-range-nothing-at-stake double
spends](#preventing-long-range-attacks) and
[censorship](#overcoming-forks-and-censorship-attacks). These are discussed more
fully in the [appendix](#appendix).

Tendermint 有各种各样的防御措施来防止一些明显的攻击，比如[远程无利害关系双花攻击](#preventing-long-range-attacks) 及[审查制度](#overcoming-forks-and-censorship-attacks)。 这些在[附录](#appendix)中有更详细的讨论。

### <P1-Reviewed>  ABCI | 区块链应用接口

The Tendermint consensus algorithm is implemented in a program called Tendermint
Core.  Tendermint Core is an application-agnostic "consensus engine" that can
turn any deterministic blackbox application into a distributedly replicated
blockchain. Tendermint Core connects to blockchain
applications via the Application Blockchain Interface (ABCI) [\[17\]][17]. Thus, ABCI
allows for blockchain applications to be programmed in any language, not just
the programming language that the consensus engine is written in.  Additionally,
ABCI makes it possible to easily swap out the consensus layer of any existing
blockchain stack.

Tendermint共识算法是在叫做 Tendermint Core 的程序中实现的。这个程序是独立于应用的“共识引擎”，可以将任何已经确定的黑盒应用转变为分布式、可复制的区块链。Tendermint Core 可以通过应用区块链接口(ABCI) [\[17\]][17]与其他区块链应用连接。而且，应用区块链接口(ABCI) 接口允许区块链应用以任何语言编程实现，而不仅仅是写这个共识引擎所使用的语言。此外，应用区块链接口(ABCI) 也让交换任何现有区块链栈的共识层成为可能。

We draw an analogy with the well-known cryptocurrency Bitcoin. Bitcoin is a
cryptocurrency blockchain where each node maintains a fully audited Unspent
Transaction Output (UTXO) database. If one wanted to create a Bitcoin-like
system on top of ABCI, Tendermint Core would be responsible for

* Sharing blocks and transactions between nodes
* Establishing a canonical/immutable order of transactions (the blockchain)

我们将其与知名加密货币比特币进行了类比。在比特币这种加密币区块链中，每个节点都维持着完整的审核过的 UTXO（未使用交易输出）数据库。如果您想要在应用区块链接口(ABCI)基础上，创建出类似比特币的系统，那么 Tendermint Core 可以做到：

* 在节点间共享区块及交易
* 创建规范或不可改变的交易顺序（区块链）

Meanwhile, the ABCI application would be responsible for

* Maintaining the UTXO database
* Validating cryptographic signatures of transactions
* Preventing transactions from spending non-existent funds
* Allowing clients to query the UTXO database

同时，ABCI应用也可以做到：

* 维护 UTXO 数据库
* 验证交易的加密签名
* 防止出现不存在的余额被交易
* 允许客户访问UTXO数据库

Tendermint is able to decompose the blockchain design by offering a very simple
API between the application process and consensus process.

Tendermint 通过在应用进程和共识进程之间提供简单的API来分解区块链设计。


## <P1-Reviewed> Cosmos Overview | Cosmos 概述 #############################################################

Cosmos is a network of independent parallel blockchains that are each powered by
classical BFT consensus algorithms like Tendermint
[1](http://github.com/tendermint/tendermint).


Cosmos是一个独立平行的区块链网络，其中每条区块链通过 Tendermint[1](http://github.com/tendermint/tendermint)这样的经典拜占庭容错共识算法来运行。

The first blockchain in this network will be the Cosmos Hub.  The Cosmos Hub
connects to many other blockchains (or _zones_) via a novel inter-blockchain
communication protocol.  The Cosmos Hub tracks numerous token types and keeps
record of the total number of tokens in each connected zone.  Tokens can be
transferred from one zone to another securely and quickly without the need for
a liquid exchange between zones, because all inter-zone coin transfers go
through the Cosmos Hub.

网络中第一条区块链将会是 Cosmos 枢纽。Cosmos 枢纽通过全新的区块链间通信协议来连接其他众多区块链（或将其称之为 _分区_）。Cosmos 枢纽可以追踪无数代币的种类，并且在各个连接的分区里记录各种代币总数。代币可以安全快速地从一个分区转移到另一个分区，两者之间无需体现汇兑流动性，因为所有分区之间的代币传输都会经过 Cosmos 枢纽。

This architecture solves many problems that the blockchain space faces today,
such as application interoperability, scalability, and seamless upgradability.
For example, zones derived from Bitcoind, Go-Ethereum, CryptoNote, ZCash, or any
blockchain system can be plugged into the Cosmos Hub.  These zones allow Cosmos
to scale infinitely to meet global transaction demand.  Zones are also a great
fit for a distributed exchange, which will be supported as well.

这一架构解决了当今区块链领域面临的许多问题，包括应用程序互操作性、可扩展性、以及可无缝升级的能力。比如，从Bitcoind、Go-Ethereum、CryptoNote、ZCash或其他区块链系统中衍生出来的分区，都能被锚定接入 Cosmos 枢纽。这些分区允许 Cosmos 实现无限扩展，从而满足全球交易的需求。此外，分区也完全适用于分布式交易所，反之交易所也支持分区运行。

Cosmos is not just a single distributed ledger, and the Cosmos Hub isn't a
walled garden or the center of its universe.  We are designing a protocol for
an open network of distributed ledgers that can serve as a new foundation for
future financial systems, based on principles of cryptography, sound economics,
consensus theory, transparency, and accountability.


Cosmos 不仅仅是单一的分布式账本，而 Cosmos 枢纽也不是封闭式庭院或宇宙的中心。我们正在为分布式账本的开放网络设计一套协议，这套协议将基于密码学、稳健经济学、共识理论、透明性及可追责制的原则，成为未来金融系统的全新基础。

### <P1-Reviewed> Tendermint-BFT |  Tendermint-拜占庭容错

The Cosmos Hub is the first public blockchain in the Cosmos Network, powered by
Tendermint's BFT consensus algorithm.  The Tendermint open-source project was
born in 2014 to address the speed, scalability, and environmental issues of
Bitcoin's proof-of-work consensus algorithm.  By using and improving upon
proven BFT algorithms developed at MIT in 1988 [\[20\]][20], the Tendermint
team was the first to conceptually demonstrate a proof-of-stake cryptocurrency
that addresses the nothing-at-stake problem suffered by first-generation
proof-of-stake cryptocurrencies such as NXT and BitShares1.0.

Cosmos 枢纽是 Cosmos 网络中第一条公共区块链，通过 Tendermint 的拜占庭共识算法运行。Tendermint 开源项目创立于2014年，旨在解决比特币工作量证明共识算法的速度、可扩展性以及造成的环境问题。通过采用并提高已经过验证的拜占庭算法（1988年在麻省理工学院开发）[\[20\]][20]，Tendermint 成为了首个在概念论证了权益证明加密货币的团队，这种机制可以解决 NXT 和 BitShares 这些第一代权益证明加密币面临的"无利害关系"的问题。

Today, practically all Bitcoin mobile wallets use trusted servers to provide
them with transaction verification.  This is because proof-of-work requires
waiting for many confirmations before a transaction can be considered
irreversibly committed.  Double-spend attacks have already been demonstrated on
services like CoinBase.

如今，实际上所有比特币移动钱包都要使用可靠的服务器来进行交易验证。这是因为工作量证明机制需要在交易被认定为无法逆转前进行多次确认。而在 CoinBase 之类的服务中也已经出现双重支付攻击。

Unlike other blockchain consensus systems, Tendermint offers instant and
provably secure mobile-client payment verification. Since the Tendermint is
designed to never fork at all, mobile wallets can receive instant transaction
confirmation, which makes trustless and practical payments a reality on
smartphones.  This has significant ramifications for Internet of Things applications as well.


和其他区块链共识系统不同，Tendermint 提供的是即时、可证明安全的移动客户端支付验证方式。因为 Tendermint 被设计为完全不分叉，所以移动钱包就可以实时接收交易确认，从而在智能手机上真正实现去信任的支付方式。这一点也大大影响了物联网应用程序。

Validators in Cosmos have a similar role to Bitcoin miners, but instead use
cryptographic signatures to vote. Validators are secure, dedicated machines
that are responsible for committing blocks.  Non-validators can delegate their
staking tokens (called "atoms") to any validator to earn a portion of block fees
and atom rewards, but they incur the risk of getting punished (slashed) if the
delegate validator gets hacked or violates the protocol.  The proven safety
guarantees of Tendermint BFT consensus, and the collateral deposit of
stakeholders--validators and delegators--provide provable, quantifiable
security for nodes and light clients.

Cosmos 中的验证人角色类似比特币矿工，但是他们采用加密签名来进行投票。验证人是专门用来提交区块的安全机器。非验证人可以将权益代币（也叫做"atom"币）委托给任何验证人来赚取一定的区块费用以及atom奖励，但是如果验证人被黑客攻击或者违反协议规定，那么代币就会面临被惩罚（削减）的风险。Tendermint 拜占庭共识的可证明安全机制，以及利益相关方（验证人和委托人）的抵押品保证，为节点甚至是轻客户端提供了可证明、可量化的安全性。

### <P1-Reviewed>  Governance | 治理 #################################################################

Distributed public ledgers should have a constitution and a governance system.
Bitcoin relies on the Bitcoin Foundation and mining to
coordinate upgrades, but this is a slow process.  Ethereum split into ETH and
ETC after hard-forking to address TheDAO hack, largely because there was no
prior social contract nor mechanism for making such decisions.

分布式公共账本应该要有一套章程与治理体系。比特币依靠比特币基金会以及挖矿来协作更新，但是这是一个反应缓慢的治理制度。以太坊在采用硬分叉成 ETH 和 ETC 来解决 The DAO 黑客，这主要是因为之前没有设定社会契约或机制来进行这类决定。

Validators and delegators on the Cosmos Hub can vote on proposals that can
change preset parameters of the system automatically (such as the block gas
limit), coordinate upgrades, as well as vote on amendments to the human-readable
constitution that govern the policies of the Cosmos Hub.  The constitution
allows for cohesion among the stakeholders on issues such as theft
and bugs (such as TheDAO incident), allowing for quicker and cleaner resolution.

Cosmos 枢纽的验证人与委托人可以对提案进行投票，从而改变预先默认设置好的系统参数（比如区块转账费用限制），协作更新，并对可读性的章程进行修订投票，从而治理 Cosmos 枢纽制度。这个章程允许权益相关者聚集到一起，来解决盗窃及漏洞等相关问题（比如The DAO事件），并得出更快更明确的解决方案。

Each zone can also have their own constitution and governance mechanism as well.
For example, the Cosmos Hub could have a constitution that enforces immutability
at the Hub (no roll-backs, save for bugs of the Cosmos Hub node implementation),
while each zone can set their own policies regarding roll-backs.


每个分区也可以制定自己的一套章程及治理机制。比如，Cosmos 枢纽的章程可以设置为强制实现枢纽的不可改变性（不能回滚，除了 Cosmos 枢纽节点产生的漏洞），而每个分区则可设置自己的回滚政策。

By enabling interoperability among differing policy zones, the Cosmos network
gives its users ultimate freedom and potential for permissionless
experimentation.

Cosmos 网络能够在制度不同的分区间实现互操作性，这一点给客户极高的自由度和潜力而无需许可即可实验（新技术）。

## <P1-Reviewed>The Hub and Zones| 枢纽与分区 ###########################################################

Here we describe a novel model of decentralization and scalability.  Cosmos is a
network of many blockchains powered by Tendermint.  While existing proposals aim
to create a "single blockchain" with total global transaction ordering, Cosmos
permits many blockchains to run concurrently with one another while retaining
interoperability.

这里我们将描述一个全新的去中心化与可扩展性模型。Cosmos 网络通过 Tendermint 机制来运行众多的区块链。虽然现存提案的目标是创建一个包含全球所有交易订单的"单一区块链"，但是 Cosmos 允许众多区块链在并行运行的同时，保持可互操作性。


At the basis, the Cosmos Hub manages many independent blockchains called "zones"
(sometimes referred to as "shards", in reference to the database scaling
technique known as "sharding").  A constant stream of recent block commits from
zones posted on the Hub allows the Hub to keep up with the state of each zone.
Likewise, each zone keeps up with the state of the Hub (but zones do not keep up
with each other except indirectly through the Hub).  Packets of information are
then communicated from one zone to another by posting Merkle-proofs as evidence
that the information was sent and received.  This mechanism is called
inter-blockchain communication, or IBC for short.

在这个基础上，Cosmos枢纽负责管理称之为“分区”的众多独立区块链（有时也叫做"分片"，参考自众所周知的数据库扩展技术"分片"）。枢纽上的分片会源源不断地提交最新区块，这一点可以让枢纽同步每一个分区的状态。同样地，每个分区也会和枢纽的状态保持一致（不过分区之间不会同彼此的同步，除非间接通过枢纽来实现）。通过发布默克尔证明来证明消息被接受和发送，来让消息从一个分区传递到另一个分区。这种机制叫做"区块链间通信"，或者简称为"IBC"机制。

![Figure of hub and zones
acknowledgement](https://raw.githubusercontent.com/gnuclear/atom-whitepaper/master/images/hub_and_zones.png)

Any of the zones can themselves be hubs to form an acyclic graph, but
for the sake of clarity we will only describe the simple configuration where
there is only one hub, and many non-hub zones.


任何分区都可以自行成为枢纽来建立非循环图表，但为了清楚起见，我们只描述这种只有一个枢纽和许多非枢纽的分区这样简单的配置


### <P1-Reviewed> The Hub | 枢纽

The Cosmos Hub is a blockchain that hosts a multi-asset distributed ledger,
where tokens can be held by individual users or by zones themselves.  These
tokens can be moved from one zone to another in a special IBC packet called a
"coin packet".  The hub is responsible for preserving the global invariance of
the total amount of each token across the zones. IBC coin packet transactions
must be committed by the sender, hub, and receiver blockchains.

Cosmos枢纽是承载多种分布式账本资产的区块链，其中代币可以由个人或分区自己持有。这些代币能够通过特殊的IBC数据包，即"代币数据包"（coin packet）从一个分区转移到另一个分区。枢纽负责保持各个分区中各类代币总量不变。IBC代币数据包交易必须由发送人、枢纽及区块接受者执行。

Since the Cosmos Hub acts as the central ledger for the whole
system, the security of the Hub is of paramount importance.  While each
zone may be a Tendermint blockchain that is secured by as few as 4 (or even
less if BFT consensus is not needed), the Hub must be secured by a globally
decentralized set of validators that can withstand the most severe attack
scenarios, such as a continental network partition or a nation-state sponsored
attack.


因为Cosmos枢纽在整个系统中扮演着中央代币账本的角色，其安全性极其重要。虽然每个分区可能都是一个Tendermint区块链——只需通过4个，(或者在无需拜占庭容错共识的情况下更少的验证人来保证安全），但是Cosmos枢纽必须通过全球去中心化验证组来保证安全，而且这个验证组要能够承受最严重的攻击，比如区域网络分裂或者由国家发起的攻击。

### <P1-Reviewed> The Zones | 分区

A Cosmos zone is an independent blockchain that exchanges IBC messages with the
Hub.  From the Hub's perspective, a zone is a multi-asset dynamic-membership
multi-signature account that can send and receive tokens using IBC packets. Like
a cryptocurrency account, a zone cannot transfer more tokens than it has, but
can receive tokens from others who have them. A zone may be designated as an
"source" of one or more token types, granting it the power to inflate that token
supply.

Cosmos分区是独立的区块链，能够和Cosmos枢纽进行IBC消息交换。从枢纽的角度上看，分区是一种多重资产、动态会员制的多重签名账户，可以通过IBC数据包用来发送和接受代币。就像加密币账户一样，分区不能转移超出其持有量的代币，不过可以从其他拥有代币的人那里接收代币。分区可能会被指定为一种或多种代币的"来源"，从而赋予其增加代币供应量的权力。

Atoms of the Cosmos Hub may be staked by validators of a zone connected to the
Hub.  While double-spend attacks on these zones would result in the slashing of
atoms with Tendermint's fork-accountability, a zone where >⅔ of the voting power
are Byzantine can commit invalid state.  The Cosmos Hub does not verify or
execute transactions committed on other zones, so it is the responsibility of
users to send tokens to zones that they trust.  In the future, the Cosmos Hub's
governance system may pass Hub improvement proposals that account for zone
failures.  For example, outbound token transfers from some (or all) zones may be
throttled to allow for the emergency circuit-breaking of zones (a temporary halt
of token transfers) when an attack is detected.

Cosmos 枢纽的 Atom 或可作为分区验证人连接到枢纽的筹码。虽然在Tendermint分叉责任制下，分区出现双重支付攻击会导致atom数量减少，但是如果分区中有超过⅔的选票都出现拜占庭问题的话，那这个分区就可以提交无效状态。Cosmos 枢纽不会验证或执行提交到其他分区的交易，因此将代币发送到可靠的分区间就是用户的责任了。未来 Cosmos 枢纽的管理系统可能会通过改善提案，来解决分区故障问题。比如，在检测到袭击时，可以将有些分区（或全部分区）发起的代币转账将被暂停，实现紧急断路（即暂时中止代币转账）。

## <P1-Reviewed> Inter-blockchain Communication (IBC) | 跨链通信（IBC） ########################################

Now we look at how the Hub and zones communicate with each other.  For example, if
there are three blockchains, "Zone1", "Zone2", and "Hub", and we wish for
"Zone1" to produce a packet destined for "Zone2" going through "Hub". To move a
packet from one blockchain to another, a proof is posted on the
receiving chain. The proof states that the sending chain published a packet for the alleged
destination. For the receiving chain to check this proof, it must be able keep
up with the sender's block headers.  This mechanism is similar to that used by
sidechains, which requires two interacting chains to be aware of one another via a
bidirectional stream of proof-of-existence datagrams (transactions).

现在我们来介绍下枢纽与分区之间通信的方法。假如现在有三个区块链，分别是"分区1"、“分区2"以及"枢纽”，我们想要"分区1"生成一个数据包，通过"枢纽"发送给"分区2"。为了让数据包从一个区块链转移到另一个区块链，需要在接收方区块链上发布一个证明，来明确发送方已经发起了一个数据包到指定目的地。接收方要验证的这个证明，必须和发送方区块头保持一致。这种机制就类似与侧链采用的机制，它需要两个相互作用的链，通过双向传送存在证明数据元（交易），来"知晓"另一方的情况。

The IBC protocol can naturally be defined using two types of transactions: an  `IBCBlockCommitTx` transaction, which allows a blockchain to prove to any
observer of its most recent block-hash, and an `IBCPacketTx` transaction, which
allows a blockchain to prove to any observer that the given packet was indeed
published by the sender's application, via a Merkle-proof to the recent
block-hash.

IBC协议可以自然定义为两种交易的使用：一种是IBCBlockCommitTx 交易，这种交易可以让区块链向任何观察员证明其最新区块哈希值；另一种是IBCPacketTx 交易，这种交易则可以证明某个数据包确实由发送者的应用程序，通过默克尔证明机制（Merkle-proof）传送到了最新区块的哈希值上。

By splitting the IBC mechanics into two separate transactions, we allow the
native fee market-mechanism of the receiving chain to determine which packets
get committed (i.e. acknowledged), while allowing for complete freedom on the
sending chain as to how many outbound packets are allowed.

通过将IBC机制分离成两个单独的交易，即IBCBlockCommitTx 交易与 IBCPacketTx 交易，我们可以让接收方链的本地费用市场机制，来决定承认哪个数据包，与此同时还能确保发送方的完全自由，让其自行决定能够传出的数据包数量。

![Figure of Zone1, Zone2, and Hub IBC without
acknowledgement](https://raw.githubusercontent.com/gnuclear/atom-whitepaper/master/msc/ibc_without_ack.png)

<CAPTION on a figure> In the example above, in order to update the block-hash of
"Zone1" on "Hub" (or of "Hub" on "Zone2"), an `IBCBlockCommitTx`
transaction must be posted on "Hub" with the block-hash of "Zone1" (or on
"Zone2" with the block-hash of "Hub").


在上述案例中，为了更新"枢纽"上"分区1"的区块哈希（或者说"分区2"上"枢纽"的区块哈希），必须将IBCBlockCommitTx交易的"分区1"区块哈希值发布到"枢纽"上（或者将该交易的"枢纽"区块哈希值发布到"分区2"中）。

_See [IBCBlockCommitTx](#ibcblockcommittx) and [IBCPacketTx](#ibcpacketcommit)
for for more information on the two IBC transaction types._

_关于两种IBC交易类型，详细请参见 [IBCBlockCommitTx](#ibcblockcommittx) 和 [IBCPacketTx](#ibcpacketcommit)

## Use Cases | 用例 ###################################################################

### <P1-Reviewed> Distributed Exchange | 分布式交易所

In the same way that Bitcoin is more secure by being a distributed,
mass-replicated ledger, we can make exchanges less vulnerable to external and
internal hacks by running it on the blockchain.  We call this a distributed
exchange.

比特币借助大量复制来增加分布式账本的安全性。用类似的方式，我们可以在区块链上运行交易所，来降低其受内部及外部攻击的可能性。我们称之为去中心化交易所。

What the cryptocurrency community calls a decentralized exchange today are
based on something called "atomic cross-chain" (AXC) transactions.  With an AXC
transaction, two users on two different chains can make two transfer
transactions that are committed together on both ledgers, or none at all (i.e.
atomically).  For example, two users can trade bitcoins for ether (or any two
tokens on two different ledgers) using AXC transactions, even though Bitcoin
and Ethereum are not connected to each other.  The benefit of running an
exchange on AXC transactions is that neither users need to trust each other or
the trade-matching service.  The downside is that both parties need to be
online for the trade to occur.

现今，加密货币社区认为的去中心化交易所基于"跨链原子事务"交易（ AXC 交易）。通过AXC交易，两条不同链上的两个用户可以发起两笔转账交易，交易在两个账本上要么一起提交执行，或者两个账本都不执行（即交易的原子性）。比如，两位用户可以通过AXC交易来实现比特币和以太币之间的交易（或是在不同账本上的任意两种代币），即使比特币和以太坊的区块链之间并没有彼此连接。AXC 交易模式下的交易所用户双方不需要彼此信任，也不用依赖交易匹配服务。其弊端是，交易双方必须同时在线才能进行交易。

Another type of decentralized exchange is a mass-replicated distributed
exchange that runs on its own blockchain.  Users on this kind of exchange can
submit a limit order and turn their computer off, and the trade can execute
without the user being online.  The blockchain matches and completes the trade
on behalf of the trader.

另一种去中心化交易所是进行大量复制的具有独立区块链的分布式交易所。该种交易所的用户可以提交限价订单并关闭他们的计算机，交易可以在用户离线状态下执行。区块链将会代表交易者去完成匹配和交易。

A centralized exchange can create a deep orderbook of limit orders and thereby
attract more traders.  Liquidity begets more liquidity in the exchange world,
and so there is a strong network effect (or at least a winner-take-most effect)
in the exchange business.  The current leader for cryptocurrency exchanges
today is Poloniex with a 24-hour volume of $20M, and in second place is
Bitfinex with a 24-hour volume of $5M.  Given such strong network effects, it
is unlikely for AXC-based decentralized exchanges to win volume over the
centralized exchanges.  For a decentralized exchange to compete with a
centralized exchange, it would need to support deep orderbooks with limit
orders.  Only a distributed exchange on a blockchain can provide that.

一个中心化的交易所可以构建一个有大交易量的限价交易的买卖盘账目，以此来吸引更多的交易者。在交易所领域，流动性会引发更多流动性，因此在交易所业务中，其具有的网络效应也愈发明显（或者说至少产生了"赢家通吃"效应）。目前加密货币交易所 Poloniex 以每24小时2,000万美元的交易量排名第一， Bitfinex 则每24小时500万美元的交易额位列第二。在这种强大的网络效应之下，基于AXC的去中心化交易所的成交量是不太可能超过中心化交易所。去中心化交易所要想和中心化交易所一争高下，就需要支持以限价订单构成的具有深度的交易买卖盘账目的运行。而只有基于区块链的去中心化交易所可以实现这一点。

Tendermint provides additional benefits of faster transaction commits.  By
prioritizing fast finality without sacrificing consistency, zones in Cosmos can
finalize transactions fast -- for both exchange order transactions as well as
IBC token transfers to and from other zones.

Tendermint提供的快速交易执行是另一大优势。Cosmos的内部网络可以在不牺牲一致性的前提下优先快速的确定最终性，来实现交易的快速完成 —— 同时针对交易订单交易，以及IBC（跨区块链通信）代币与其他网络的交易。

Given the state of cryptocurrency exchanges today, a great application for
Cosmos is the distributed exchange (aka the Cosmos DEX).  The transaction
throughput capacity as well as commit latency can be comparable to those of
centralized exchanges.  Traders can submit limit orders that can be executed
without both parties having to be online.  And with Tendermint, the Cosmos hub,
and IBC, traders can move funds in and out of the exchange to and from other
zones with speed.

综上，根据现有加密货币交易所的情况，Cosmos的一项重大应用就是去中心化交易所（称为 Cosmos DEX）。其交易吞吐能量和委托延时可以与那些中心化交易所媲美。交易者可以在各方离线的状态下提交限价订单。并且，基于Tendermint，Cosmos枢纽以及IBC的情况下，交易者可以快速地完成在交易所及其他网络的资金进出。

### Bridging to Other Cryptocurrencies | 和其他加密货币的纽带

A privileged zone can act as the source of a bridged token of another
cryptocurrency. A bridge is similar to the relationship between a
Cosmos hub and zone; both must keep up with the latest blocks of the
other in order to verify proofs that tokens have moved from one to the other.  A
"bridge-zone" on the Cosmos network keeps up with the Hub as well as the
other cryptocurrency.  The indirection through the bridge-zone allows the logic of
the Hub to remain simple and agnostic to other blockchain consensus strategies
such as Bitcoin's proof-of-work mining.

特权分区可以作为和其他加密货币挂钩的代币来源。这种挂钩类似Cosmos枢纽与分区之间的关系，两者都必须及时更新彼此最新的区块链，从而验证代币已经从一方转移到另一方。Cosmos网络上挂钩的”桥接分区“要和中心以及其他加密币保持同步。这种间接通过”桥接分区“可以保持枢纽逻辑的简洁。并且不必要了解其他的链上共识战略，如比特币工作量证明挖矿机制。

#### Sending Tokens to the Cosmos Hub | 向Cosmos中心发送代币

Each bridge-zone validator would run a Tendermint-powered blockchain with a special
ABCI bridge-app, but also a full-node of the "origin" blockchain.

每个挂钩桥接分区的验证人都会在基于Tendermint公式的区块链之上，运行带有特殊的ABCI桥接应用程序，但同时也会运行一个原有区块链的“全节点”。

When new blocks are mined on the origin, the bridge-zone validators will come
to agreement on committed blocks by signing and sharing their respective local
view of the origin's blockchain tip.  When a bridge-zone receives payment on the
origin (and sufficient confirmations were agreed to have been seen in the case
of a PoW chain such as Ethereum or Bitcoin), a corresponding account is created
on the bridge-zone with that balance.

在原有区块链挖出新区块时，桥接分区验证人员将通过签署和分享起始点区块链的提示，各自局部视角可以达成一致。当一个桥接分区收到原有区块链的支付时（如在以太坊或比特币等PoW机制的链上有足够数目的确认），则在该桥接分区上创建具有该对应账户的余额。

In the case of Ethereum, the bridge-zone can share the same validator-set as the
Cosmos Hub.  On the Ethereum side (the origin), a bridge-contract would allow
ether holders to send ether to the bridge-zone by sending it to the bridge-contract
on Ethereum.  Once ether is received by the bridge-contract, the ether cannot be
withdrawn unless an appropriate IBC packet is received by the bridge-contract from
the bridge-zone.  The bridge-contract tracks the validator-set of the bridge-zone, which
may be identical to the Cosmos Hub's validator-set.

就以太坊而言，桥接分区可以和Cosmos枢纽共享相同的验证人。以太坊方面（原本区块链），一个桥接合约将允许以太拥有者通过将以太币发送到以太坊的桥接分区的桥接合约上。一旦挂桥接合约接收到以太币，以太币就不能被撤回，除非从桥接分区接收到对应的IBC数据包。桥接合约跟随桥接分区的验证组，它可能与Cosmos枢纽的验证人组相同。

In the case of Bitcoin, the concept is similar except that instead of a single
bridge-contract, each UTXO would be controlled by a threshold multisignature P2SH
pubscript.  Due to the limitations of the P2SH system, the signers cannot be
identical to the Cosmos Hub validator-set.

就比特币而言，概念是相似，除了代替一个桥接合约，每个UTXO将由一个门限多重签名P2SH数据库限制。由于P2SH系统的限制，签名者不能与Cosmos枢纽的验证人组相同。

#### Withdrawing Tokens from Cosmos Hub | 从Cosmos枢纽提出代币

Ether on the bridge-zone ("bridged-ether") can be transferred to and from the
Hub, and later be destroyed with a transaction that sends it to a particular
withdrawal address on Ethereum. An IBC packet proving that the transaction
occurred on the bridge-zone can be posted to the Ethereum bridge-contract to
allow the ether to be withdrawn.

桥接分区上的以太币（“桥接以太币”）可以在枢纽间转进，转出，完成传送到特定以太坊提取地址后，转出的“桥接以太币”被彻底删除。一个IBC消息可以证明桥接分区上的交易，这个消息将被公布到以太坊桥接合约中，以便以太币被取出。

In the case of Bitcoin, the restricted scripting system makes it difficult to
mirror the IBC coin-transfer mechanism.  Each UTXO has its own independent
pubscript, so every UTXO must be migrated to a new UTXO when there is a change
in the set of Bitcoin escrow signers. One solution is to compress and
decompress the UTXO-set as necessary to keep the total number of UTXOs down.

就比特币而言，严谨的交易脚本系统让IBC币的镜像转换机制很难实现。每个UTXO都有自己的特定的脚本，所以当比特币履约签名者发生变化时，每个UTXO都必须迁移到新的UTXO。一个解决方案是根据需要，压缩和解压缩UTXO-set，以保持UTXO的总数量下降。

#### Total Accountability of Bridge Zones | 挂钩区完全责任制

The risk of such a bridgeging contract is a rogue validator set.  ≥⅓ Byzantine
voting power could cause a fork, withdrawing ether from the bridge-contract on
Ethereum while keeping the bridged-ether on the bridge-zone. Worse, >⅔ Byzantine
voting power can steal ether outright from those who sent it to the
bridge-contract by deviating from the original bridgeging logic of the bridge-zone.

这类挂钩合约存在风险的风险是，可能会出现恶意的验证人组。如果拜占庭投票权超过⅓，就会造成分叉，即从以太坊桥接合约中提取以太币的同时，还能保持桥接分区中的挂钩以太币不变。甚至，如果拜占庭投票权超过⅔，可能会有人直接通过脱离原始桥接分区的桥接逻辑，对发送以太币发到桥接合约中的帐户下手，盗取以太币。

It is possible to address these issues by designing the bridge to be totally
accountable.  For example, all IBC packets, from the hub and the origin, might
require acknowledgement by the bridge-zone in such a way that all state
transitions of the bridge-zone can be efficiently challenged and verified by
either the hub or the origin's bridge-contract.  The Hub and the origin should
allow the bridge-zone validators to post collateral, and token transfers out of
the bridge-contract should be delayed (and collateral unbonding period
sufficiently long) to allow for any challenges to be made by independent
auditors.  We leave the design of the specification and implementation of this
system open as a future Cosmos improvement proposal, to be passed by the Cosmos
Hub's governance system.

如果将这个桥接方法完全设计成责任制，就有可能解决这一问题。比如，枢纽及起始点的全部IBC包裹可能需要先通过桥接分区的认可，即让枢纽或起始点中的桥接合约对桥接分区的所有状态转换进行有效验证。枢纽及起始点要允许桥接分区的验证人提供抵押物，而侨界合约的代币转出需要延时（且抵押品解绑时间也要足够长），从而让单独的审计人有时间发起任何的质询。我们会把这一系统的设计说明以及执行方式开放，作为未来Cosmos改善的提议，以待Cosmos枢纽的管理系统审批通过。

### Ethereum Scaling | 以太坊的扩展

Solving the scaling problem is an open issue for Ethereum.  Currently,
Ethereum nodes process every single transaction and also store all the states.
[link](https://docs.google.com/presentation/d/1CjD0W4l4-CwHKUvfF5Vlps76fKLEC6pIwu1a_kC_YRQ/mobilepresent?slide=id.gd284b9333_0_28).

众所周知，扩展问题是一直困扰着以太坊的问题。目前以太坊节点会处理节点上每笔交易，并且存储所有的状态。

Since Tendermint can commit blocks much faster than Ethereum's proof-of-work,
EVM zones powered by Tendermint consensus and operating on bridged-ether can
provide higher performance to Ethereum blockchains.  Additionally, though the
Cosmos Hub and IBC packet mechanics does not allow for arbitrary contract logic
execution per se, it can be used to coordinate token movements between Ethereum
contracts running on different zones, providing a foundation for token-centric
Ethereum scaling via sharding.

Tendermint提交区块的速度比以太坊工作量证明要快，所以由Tendermint共识推动且使用桥接以太币运行的以太坊虚拟机分区能够强化太坊区块链的性能。此外，虽然Cosmos枢纽及IBC包裹机制不能实现每秒合约逻辑的执行，但是它可以用来协调不同分区中以太坊合约之间的代币流通，通过分片方式为以代币为中心的以太坊扩展奠定基础。

### Multi-Application Integration | 多用一体化

Cosmos zones run arbitrary application logic, which is defined at the beginning of the
zone's life and can potentially be updated over time by governance. Such flexibility
allows Cosmos zones to act as bridges to other cryptocurrencies such as Ethereum or
Bitcoin, and it also permits derivatives of those blockchains, utilizing the
same codebase but with a different validator set and initial distribution. This
allows many existing cryptocurrency frameworks, such as those of Ethereum,
Zerocash, Bitcoin, CryptoNote and so on, to be used with Tendermint Core,
which is a higher performance consensus engine, on a common network, opening tremendous
opportunity for interoperability across platforms.  Furthermore, as a
multi-asset blockchain, a single transaction may contain multiple inputs and
outputs, where each input can be any token type, enabling Cosmos to serve
directly as a platform for decentralized exchange, though orders are assumed to
be matched via other platforms. Alternatively, a zone can serve as a distributed
fault-tolerant exchange (with orderbooks), which can be a strict improvement
over existing centralized cryptocurrency exchanges which tend to get hacked over
time.

Cosmos分区可以运行任意的应用逻辑，应用在分区创建时设定好，可通过管理者可以不断更新。这种灵活度使得Cosmos分区可以作为其他加密币的挂钩载体，比如以太坊或比特币，并且它还能和这些区块链的衍生品挂钩，利用同样的代码库，而在验证程序及初始分配有所区分。这样就允许多种现有加密币框架得以运行，如以太坊、Zerocash、比特币、CryptoNote等等，将其同Tendermint Core结合，成为通用网络中性能更优的共识引擎，为平台之间提供更多的交互机会。此外，作为多资产区块链，每笔交易都有可能包含多个输入输出项，其中每个输入项都可以是任意代币，使Cosmos直接成为去中心化交易所，当然这里假设交易订单通过其他平台进行匹配。替代方案是，让分区作为分布式容错交易所（包含买卖盘账目），这算是对中心化加密币交易所之上的严格改进——现行交易所在过去时常发生被攻击的事件。

Zones can also serve as blockchain-backed versions of enterprise and government
systems, where pieces of a particular service that are traditionally run by an
organization or group of organizations are instead run as a ABCI application on
a certain zone, which allows it to inherit the security and interoperability of the
public Cosmos network without sacrificing control over the underlying service.
Thus, Cosmos may offer the best of both worlds for organizations looking to
utilize blockchain technology but who are wary of relinquishing control completely
to a distributed third party.

分区也可以作为区块链版的企业及政府系统，其原本由一个或多个组织运行的特定服务，现在作为ABCI应用在某个分区上运行，从而在不放弃对底层服务控制的前提下，维持公共Cosmos网络的安全性及交互性。所以，Cosmos或可为那些既想使用区块链技术，又不愿彻底放弃控制权给分布式第三方的人，提供绝佳的运行环境。

### Network Partition Mitigation | 缓解网络分区问题

Some claim that a major problem with consistency-favouring consensus algorithms
like Tendermint is that any network partition which causes there to be no single
partition with >⅔ voting power (e.g. ≥⅓ going offline) will halt consensus
altogether. The Cosmos architecture can help mitigate this problem by using a global
hub with regional autonomous zones, where voting power for each zone are
distributed based on a common geographic region.  For instance, a common
paradigm may be for individual cities, or regions, to operate their own zones
while sharing a common hub (e.g. the Cosmos Hub), enabling municipal activity to
persist in the event that the hub halts due to a temporary network partition.
Note that this allows real geological, political, and network-topological
features to be considered in designing robust federated fault-tolerant systems.

有人认为像Tendermint这种支持一致性的共识算法有一个重大问题，就是网络分区会导致没有任何一个分区会拥有超过⅔的投票权（比如超过⅓投票权在线下），这会中断共识。而Cosmos架构可以缓解这个问题，它可以使用全球中心，同时，各分区实行地区自治，然后让每个分区的投票权按照正常的地域位置进行分配。如，一般范例就有可能是针对个别城市或地区的，让他们各自运行自己分区的同时，还能共享共同的枢纽（比如Cosmos枢纽），并且在临时的网络分区导致的中断期间，也可以继续维持地区自治活动。注意，这样一来在设计稳健的联邦式容错系统过程中，就可以去考虑真实的地理、政治及网络拓扑的特征了。

### Federated Name Resolution System | 联邦式名称解析系统

NameCoin was one of the first blockchains to attempt to solve the
name-resolution problem by adapting the Bitcoin blockchain.  Unfortunately there
have been several issues with this approach.

NameCoin是首批试图通过比特币技术解决名称解析问题的区块链之一。不过，这个方案存在一些不足。

With Namecoin, we can verify that, for example, <em>@satoshi</em> was registered with a
particular public key at some point in the past, but we wouldn’t know whether
the public key had since been updated recently unless we download all the blocks
since the last update of that name.  This is due to the limitation of Bitcoin's
UTXO transaction Merkle-ization model, where only the transactions (but not
mutable application state) are Merkle-ized into the block-hash. This lets us
prove existence, but not the non-existence of later updates to a name.  Thus, we
can't know for certain the most recent value of a name without trusting a full
node, or incurring significant costs in bandwidth by downloading the whole
blockchain.

例如，我们可以通过Namecoin来验证@satoshi（聪）这个号是在过去某个时间点用特定公钥进行注册的。但是，该公钥是否更新过我们就不得而知了，除非将该名称最后一次更新之前的所有全部下载。这一点是比特币UTXO交易模式中默克尔化模型的局限性导致的，这类模型中只有交易（而非可变的应用状态）会以默克尔化加入到区块哈希中。它让我们得在之后用更新证明名称的存在，而非不存在。因此，我们必须依靠全节点才能明确这个名称的最近的值，或者花费大量资源下载整个区块链。

Even if a Merkle-ized search tree were implemented in NameCoin, its dependency
on proof-of-work makes light client verification problematic. Light clients must
download a complete copy of the headers for all blocks in the entire blockchain
(or at least all the headers since the last update to a name).  This means that
the bandwidth requirements scale linearly with the amount of time [\[21\]][21].
In addition, name-changes on a proof-of-work blockchain requires waiting for
additional proof-of-work confirmation blocks, which can take up to an hour on
Bitcoin.

即使在NameCoin上运用默克尔化的搜索树，其工作量证明的独立性还是会导致轻客戸端的验证出现问题。轻客戸端必须下载区块链中所有区块头的完整备份（或者至少是自其最后的名称更新的所有区块头）。这意味着带宽需要随着时间做线性的扩展。 [21]此外，在工作量证明制度使区块链上的名称更改需要等额外的工作量证明验证确认才能进行，它在比特币上可能要花费一个小时。

With Tendermint, all we need is the most recent block-hash signed by a quorum of
validators (by voting power), and a Merkle proof to the current value associated
with the name.  This makes it possible to have a succinct, quick, and secure
light-client verification of name values.

有了Tendermint，我们只需用到由法定数量验证人签署（通过投票权）的区块哈希，以及与名称相关的当前值的默克尔证明。这点让简易、快速、安全的轻客戸端名称值验证成为可能。

In Cosmos, we can take this concept and extend it further. Each
name-registration zone in Cosmos can have an associated top-level-domain
(TLD) name such as ".com" or ".org", and each name-registration zone can have
its own governance and registration rules.

在Cosmos中，我们可以利用这个概念并延伸。每一个在Cosmos上的名称注册都能有一个相关的最高级别域名（TLD），比如".com"或者".org"等，而且每个名称注册分区都有自己的管理和登记规则。

## Issuance and Incentives | 发行与激励 #####################################################

### <P1 Reviewd> The Atom Token | Atom 代币

While the Cosmos Hub is a multi-asset distributed ledger, there is a special
native token called the _atom_.  Atoms are the only staking token of the Cosmos
Hub.  Atoms are a license for the holder to vote, validate, or delegate to other
validators.  Like Ethereum's ether, atoms can also be used to pay for
transaction fees to mitigate spam.  Additional inflationary atoms and block
transaction fees are rewarded to validators and delegators who delegate to
validators.

Cosmos 枢纽是多资产分布式账本，它有自己的代币，是 Atom 。Atom 是 Cosmos 枢纽唯一的权益代币。Atom是持有人投票、验证或委托给其他验证人的许可证明，就像以太坊上的以太币一样，Atom也可以用来支付交易费以减少电子垃圾。额外的通胀 Atom 和区块交易费用就作为激励分给验证人及委托验证人。

The `BurnAtomTx` transaction can be used to recover any proportionate amount of
tokens from the reserve pool.

`BurnAtomTx`交易可以用来恢复储蓄池中任意比例的代币。

#### <P1 Reviewd> Fundraiser | 众筹

The initial distribution of atom tokens and validators on Genesis will go to the
donors of the Cosmos Fundraiser (75%), lead donors (5%), Cosmos Network
Foundation (10%), and ALL IN BITS, Inc (10%).  From genesis onward, 1/3 of the
total amount of atoms will be rewarded to bonded validators and delegators
every year.

创世块上的 Atom 代币及验证人的初次分发是 Cosmos 众售参与者持有75%，预售参与者持有5%，Cosmos网络基金会持有10%，ALL IN BITS, 集团持有10%。从创世块开始，Atom 总量的1/3将作为奖励发放给每年担保持有的验证人以及委托人。

See the [Cosmos Plan](https://github.com/cosmos/cosmos/blob/master/PLAN.md)
for additional details.

更多细节见 [Cosmos Plan](https://github.com/cosmos/cosmos/blob/master/PLAN.md)

### <P1 Reviewd> Limitations on the Number of Validators | 验证人的数量限制

Unlike Bitcoin or other proof-of-work blockchains, a Tendermint blockchain gets
slower with more validators due to the increased communication complexity.
Fortunately, we can support enough validators to make for a robust globally
distributed blockchain with very fast transaction confirmation times, and, as
bandwidth, storage, and parallel compute capacity increases, we will be able to
support more validators in the future.

与比特币或其他工作量证明区块链不同的是, 由于通信的复杂性增加, Tendermint 区块链会随着验证人的增加而变慢。幸运的是, 我们可以支持足够多的验证人来实现可靠的全球化分布式区块链， 并具有非常快的交易确认时间。 而且随着带宽、存储和并行计算容量的增加, 我们将来能够支持更多的验证人。

On genesis day, the maximum number of validators will be set to 100, and this
number will increase at a rate of 13% for 10 years, and settle at 300
validators.

在创世日, 验证人的最大数量将设置为 100, 这个数字将以13% 的速度增长10年, 最终达到300位。

```
Year 0: 100
Year 1: 113
Year 2: 127
Year 3: 144
Year 4: 163
Year 5: 184
Year 6: 208
Year 7: 235
Year 8: 265
Year 9: 300
Year 10: 300
...
```

### <P1 Reviewd> Becoming a Validator After Genesis Day | 成为创世日后的验证人

Atom holders who are not already can become validators by signing and
submitting a `BondTx` transaction.  The amount of atoms provided as collateral
must be nonzero.  Anyone can become a validator at any time, except when the
size of the current validator set is greater than the maximum number of
validators allowed.  In that case, the transaction is only valid if the amount
of atoms is greater than the amount of effective atoms held by the smallest
validator, where effective atoms include delegated atoms.  When a new validator
replaces an existing validator in such a way, the existing validator becomes
inactive and all the atoms and delegated atoms enter the unbonding state.

Atom 持有者可以通过签署和提交 `BondTx` 交易成为验证人。抵押的 atom 数量不能为零。任何人任何时候都成为验证人, 除非当前验证人组的数量超过了最大值。在这种情况下, 只有当持有 atom 的数量大于现有验证人中持有有效 atom 数量的最少者, 该交易才有效, 其中有效 atom 包括受委托的 atom。当一个新的验证人以这种方式替换现有的验证人时, 现有的验证人将离线， 其所有的 atom 和受委托的 atom 进入解绑状态。

### Penalties for Validators | 对验证人的惩罚

There must be some penalty imposed on the validators for any intentional
or unintentional deviation from the sanctioned protocol. Some evidence is
immediately admissible, such as a double-sign at the same height and round, or a
violation of "prevote-the-lock" (a rule of the Tendermint consensus protocol).
Such evidence will result in the validator losing its good standing and its
bonded atoms as well its proportionate share of tokens in the reserve pool --
collectively called its "stake" -- will get slashed.

对于任何有意或无意的偏离认可协议的验证人, 必须对其施加一定的惩罚。有些证据立即可予受理, 比如在同样高度和回合的双重签名, 或违反 "预投票锁定" (Tendermint 协商一致议定书的规则)。这样的证据将导致验证人失去其良好的声誉, 其绑定的 atom 以及在储备池中的比例份额 – 统称为 “权益” – 将被大幅削减。

Sometimes, validators will not be available, either due to regional network
disruptions, power failure, or other reasons.  If, at any point in the past
`ValidatorTimeoutWindow` blocks, a validator's commit vote is not included in
the blockchain more than `ValidatorTimeoutMaxAbsent` times, that validator will
become inactive, and lose `ValidatorTimeoutPenalty` (DEFAULT 1%) of its stake.

有时, 由于区域网络中断、电源故障或其他原因, 验证人将不可用。如果在过去任意时间点的 `ValidatorTimeoutWindow` 块中, 验证人的提交投票不包括在区块链中超过  `ValidatorTimeoutMaxAbsent` 次, 该验证人将离线, 并减少 `ValidatorTimeoutPenalty` (默认 1%) 的权益。

Some "malicious" behavior does not produce obviously discernible evidence on the
blockchain. In these cases, the validators can coordinate out of band to force
the timeout of these malicious validators, if there is a supermajority
consensus.

一些 "恶意" 行为在区块链上并没有产生明显的证据。在这些情况下, 如果存在多数的协商一致, 则验证人可以在带外协调,强制将这些恶意验证人超时。

In situations where the Cosmos Hub halts due to a ≥⅓ coalition of voting power
going offline, or in situations where a ≥⅓ coalition of voting power censor
evidence of malicious behavior from entering the blockchain, the hub must
recover with a hard-fork reorg-proposal.  (Link to "Forks and Censorship
Attacks").

如果 Cosmos 枢纽 因为超过⅓的投票权离线而出现了中止情况，或者说超过⅓的投票权审查到进入区块链的恶意行为，这时候枢纽就必须借助硬分叉重组协议来恢复。（详见“分叉与审查攻击”）

### <P1 Reviewd> Transaction Fees | 交易费用

Cosmos Hub validators can accept any token type or combination of types as fees
for processing a transaction.  Each validator can subjectively set whatever
exchange rate it wants, and choose whatever transactions it wants, as long as
the `BlockGasLimit` is not exceeded.  The collected fees, minus any taxes
specified below, are redistributed to the bonded stakeholders in proportion to
their bonded atoms, every `ValidatorPayoutPeriod` (DEFAULT 1 hour).

Cosmos 枢纽验证人可以接受任何种类的代币或组合作为处理交易的费用。每个验证人可自行设置兑换率， 并选择其想要的交易, 只要不超过 `BlockGasLimit`,  每隔 `ValidatorPayoutPeriod` (默认为1小时) 时间会根据权益相关人绑定的 Atom 比例进行分配。

Of the collected transaction fees, `ReserveTax` (DEFAULT 2%) will go toward the
reserve pool to increase the reserve pool and increase the security and value of
the Cosmos network. These funds can also be distributed in accordance with the
decisions made by the governance system.

在所收取的交易费用中, `ReserveTax` (默认 2%) 将存入储备池来增加储备量, 增加 Cosmos 枢纽的安全性和价值。这些资金也可以按照治理系统的决策进行分配。

Atom holders who delegate their voting power to other validators pay a
commission to the delegated validator.  The commission can be set by each
validator.

将投票权委托给其他验证人的 Atom 持有人会支付一定佣金给委托方，而这笔费用可以由每个验证人进行设置。

### <P1 Reviewd> Incentivizing Hackers | 激励黑客

The security of the Cosmos Hub is a function of the security of the underlying
validators and the choice of delegation by delegators.  In order to encourage
the discovery and early reporting of found vulnerabilities, the Cosmos Hub
encourages hackers to publish successful exploits via a `ReportHackTx`
transaction that says, "This validator got hacked.  Please send
bounty to this address".  Upon such an exploit, the validator and delegators
will become inactive, `HackPunishmentRatio` (default 5%) of everyone's atoms
will get slashed, and `HackRewardRatio` (default 5%) of everyone's atoms will
get rewarded to the hacker's bounty address.  The validator must recover the
remaining atoms by using their backup key.

Cosmos 枢纽的安全取决于底层验证人的安全性和委托人的委托选择。为了鼓励发现和早期报告发现的漏洞, Cosmos 枢纽鼓励黑客通过 `ReportHackTx` 交易发布成功的漏洞, 说, "这个验证人被入侵了，请把赏金发送到这个地址"。这种情况下, 验证人和委托人将挂起闲置, 每个人 `HackPunishmentRatio` (默认 5%) 的 atom 将被削减,  `HackRewardRatio` (默认 5%) 的 atom 将发送到黑客的赏金地址作为奖励。验证人必须使用其备份密钥来恢复剩余的 atom。

In order to prevent this feature from being abused to transfer unvested atoms,
the portion of vested vs unvested atoms of validators and delegators before and
after the `ReportHackTx` will remain the same, and the hacker bounty will
include some unvested atoms, if any.

为了防止这一特性被滥用于转移未授权的 atom, 在 `ReportHackTx` 前后，Atom的比例（授权的与未授权的） 将保持不变, 而黑客的赏金将包括一些未授权的 atom (如果有的话)。

### <P1 Reviewd>  Governance Specification | 治理规范 ###################################################

The Cosmos Hub is operated by a distributed organization that requires a well-defined
governance mechanism in order to coordinate various changes to the blockchain,
such as the variable parameters of the system, as well as software upgrades and
constitutional amendments.

Cosmos 枢纽是由一个分布式组织管理的, 需要一个明确的治理机制, 以协调对区块链的各种变化, 如系统的参数变量, 以及软件升级和宪法修订.

All validators are responsible for voting on all proposals.  Failing to vote on
a proposal in a timely manner will result in the validator being deactivated
automatically for a period of time called the `AbsenteeismPenaltyPeriod`
(DEFAULT 1 week).

所有验证人负责对所有提案进行表决。如果未能及时对提案进行表决, 将导致验证人被自动停用一段时间。 这段时间被称为 `AbsenteeismPenaltyPeriod` (默认1周)。

Delegators automatically inherit the vote of the delegated validator.  This vote
may be overridden manually.  Unbonded atoms get no vote.

委托人自动继承其委托的验证人的投票权。这一投票可以被手动覆盖掉。而未绑定的 Atom 是没有投票权的。

Each proposal requires a deposit of `MinimumProposalDeposit` tokens, which may
be a combination of one or more tokens including atoms.  For each proposal, the
voters may vote to take the deposit. If more than half of the voters choose to
take the deposit (e.g. because the proposal was spam), the deposit goes to the
reserve pool, except any atoms which are burned.

每个提案都需要 `MinimumProposalDeposit` 代币的保证金, 这可能是一个或多个代币 (包括atom) 的组合。对于每项提案, 投票者可以投票表决取走保证金。如果超过半数的投票者选择取走保证金 (例如, 因为提案是垃圾信息), 那么保证金就会存入储备池, 除了被燃烧的 atoms。

For each proposal, voters may vote with the following options:

* Yea
* YeaWithForce
* Nay
* NayWithForce
* Abstain

对于每项提案, 投票人可以选择下列方案:

* 同意
* 强烈同意
* 反对
* 强烈反对
* 弃权

A strict majority of Yea or YeaWithForce votes (or Nay or NayWithForce votes) is
required for the proposal to be decided as passed (or decided as failed), but
1/3+ can veto the majority decision by voting "with force".  When a strict
majority is vetoed, everyone gets punished by losing `VetoPenaltyFeeBlocks`
(DEFAULT 1 day's worth of blocks) worth of fees (except taxes which will not be
affected), and the party that vetoed the majority decision will be additionally
punished by losing `VetoPenaltyAtoms` (DEFAULT 0.1%) of its atoms.

决定采纳（或不采纳）提案需要严格的多数投“同意”或“强烈同意”（或者“反对”及“强烈反对”），但是超过1/3的人投“强烈反对”或“强烈支持”的话就可以否决大多数人的决定。如果大多数人票被否决，那么他们每个人都会失去 `VetoPenaltyFeeBlocks` (默认是一天的区块值 ，税费除外) 作为惩罚，而否决大多数决定的那一方还将额外失去 `VetoPenaltyAtoms` 默认为0.1%）的 Atom 作为惩罚。

### <P1 Reviewd> Parameter Change Proposal | 参数变更提案

Any of the parameters defined here can be changed with the passing of a
`ParameterChangeProposal`.

这里定义的任何参数都可在 `ParameterChangeProposal` 通过后改变。

### <P1 Reviewd>  Bounty Proposal | 赏金提案

Atoms can be inflated and reserve pool funds spent with the passing of a `BountyProposal`.

通过 `BountyProposal` 后， Atom 可以增发和预留储备池资金作为赏金。

### <P1 Reviewd>  Text Proposal | 文本提案

All other proposals, such as a proposal to upgrade the protocol, will be
coordinated via the generic `TextProposal`.

所有其他提案，比如用来更新协议的提案，都会通过通用的 TextProposal 来协调。


## Roadmap | 路线图 #####################################################################

See [the Plan](https://github.com/cosmos/cosmos/blob/master/PLAN.md).

详见[计划](https://github.com/cosmos/cosmos/blob/master/PLAN.md).

## Related Work | 相关工作 ################################################################

There have been many innovations in blockchain consensus and scalability in the
past couple of years.  This section provides a brief survey of a select number
of important ones.

过去几年涌现了很多区块链共识及扩展性方面的创新。在这一部分中将挑选一些重要的创新进行简单分析。

### Consensus Systems | 共识系统

#### Classic Byzantine Fault Tolerance | 经典拜占庭容错

Consensus in the presence of malicious participants is a problem dating back to
the early 1980s, when Leslie Lamport coined the phrase "Byzantine fault" to
refer to arbitrary process behavior that deviates from the intended behavior,
in contrast to a "crash fault", wherein a process simply crashes. Early
solutions were discovered for synchronous networks where there is an upper
bound on message latency, though practical use was limited to highly controlled
environments such as airplane controllers and datacenters synchronized via
atomic clocks.  It was not until the late 90s that Practical Byzantine Fault
Tolerance (PBFT) [\[11\]][11] was introduced as an efficient partially
synchronous consensus algorithm able to tolerate up to ⅓ of processes behaving
arbitrarily.  PBFT became the standard algorithm, spawning many variations,
including most recently one created by IBM as part of their contribution to
Hyperledger.

二十世纪八十年代早期就开始研究存在恶意参与者的共识机制，当时Leslie Lamport创造了”拜占庭容错”这个词，用来指那些图谋不轨参与者做出的恶意的行为，与”死机故障”不同，后者只是处理过程崩溃而已。早期针对同步网络也探索出了一些解决方案，网络信息滞后有一个上限，但实际使用是在高度受控的环境下进行，比如精密飞行仪器以及使用原子钟同步的数据中心。直到九十年代后期，实用拜占庭容错（ Practical Byzantine Fault Tolerance ,PBFT）[\[11\]][11]才作为有效的、部分同步的共识算法被逐步推广。它可以容忍⅓参与者有恶意行为。PBFT成为标准算法，催生了各种版本，包括最近由IBM提出并使用于Hyperledger超级账本中的算法。

The main benefit of Tendermint consensus over PBFT is that Tendermint has an
improved and simplified underlying structure, some of which is a result of
embracing the blockchain paradigm.  Tendermint blocks must commit in order,
which obviates the complexity and communication overhead associated with PBFT's
view-changes.  In Cosmos and many cryptocurrencies, there is no need to allow
for block <em>N+i</em> where <em>i >= 1</em> to commit, when block <em>N</em>
itself hasn't yet committed. If bandwidth is the reason why block <em>N</em>
hasn't committed in a Cosmos zone, then it doesn't help to use bandwidth sharing
votes for blocks <em>N+i</em>. If a network partition or offline nodes is the
reason why block <em>N</em> hasn't committed, then <em>N+i</em> won't commit
anyway.

和PBFT相比，Tendermint共识的主要好处在于其改善且简化了的底层结构，其中有些是遵循了区块链典范的结果。Tendermint中，区块必须按顺序提交，这就消除复杂性，节省PBFT中状态变化相关的通信开支。在Cosmos和众多加密币中，如果区块N本身没有提交，那么就不能让它之后的区块N+i（i>=1）提交。如果是通信带宽限制导致了区块N未提交到Cosmos Zone上，那么将通信带宽用于分享选票给区块N+i是一种浪费。如果由于网络分区或者节点掉线导致的区块N未提交，那么N+i就无论如何也不能提交。

In addition, the batching of transactions into blocks allows for regular
Merkle-hashing of the application state, rather than periodic digests as with
PBFT's checkpointing scheme.  This allows for faster provable transaction
commits for light-clients and faster inter-blockchain communication.

此外，将交易打包成块可以用默克尔哈希纪录应用程序的状态，而不是用PBFT检查机制进行定时摘要。这可以让轻客戸端更快的提交交易证明，以及更快的跨链通信。

Tendermint Core also includes many optimizations and features that go above and
beyond what is specified in PBFT.  For example, the blocks proposed by
validators are split into parts, Merkle-ized, and gossipped in such a way that
improves broadcasting performance (see LibSwift [\[19\]][19] for inspiration).
Also, Tendermint Core doesn't make any assumption about point-to-point
connectivity, and functions for as long as the P2P network is weakly connected.

Tendermint Core中也优化了很多PBFT特性以外的功能。比如，验证人提交的区块被分割多个部分，对其默克尔化后，然后在节点间广播。这种方式可以提高其广播性能（具体请查看LibSwift [19]）。而且，Tendermint Core不会对点对点连接做任何假设，只要点对点间的网络不断开，那么它就能正常运行。

#### BitShares delegated stake | BitShare委托权益

While not the first to deploy proof-of-stake (PoS), BitShares1.0 [\[12\]][12]
contributed considerably to research and adoption of PoS blockchains,
particularly those known as "delegated" PoS.  In BitShares, stake holders elect
"witnesses", responsible for ordering and committing transactions, and
"delegates", responsible for coordinating software updates and parameter
changes.  BitShares2.0 aims to achieve high performance (100k tx/s, 1s latency)
in ideal conditions, with each block signed by a single signer, and transaction
finality taking quite a bit longer than the block interval.  A canonical
specification is still in development.  Stakeholders can remove or replace
misbehaving witnesses on a daily basis, but there is no significant collateral
of witnesses or delegators in the likeness of Tendermint PoS that get slashed
in the case of a successful double-spend attack.

BitShares [\[12\]][12]不是第一个采用权益证明机制（proof-of-stake,PoS）的区块链，但是其对PoS在区块链上的研究与推进做出了巨大的贡献，尤其是在DPoS，即受委托权益证明方面。在BitShares中，相关方选择”见证者”负责提交交易顺序并提交；相关方选择”委托人”负责协调软件更新与参数变化。尽管BitShare在理想环境下能达到很高的性能：100k tx/s，1秒的滞后。每一块只有一个单独的签名，得到交易的最终性的时间比区块时间略长。一个标准的协议仍在开发中。利益相关者可以每天去除或者替换有恶意行为的验证人，但是不同于Tendermint PoS的保证金机制，BitShares没有要求验证人或者代理人的提交押金，如果发生双花攻击的话，押金不会被削减。

#### Stellar

Building on an approach pioneered by Ripple, Stellar [\[13\]][13] refined a
model of Federated Byzantine Agreement wherein the processes participating in
consensus do not constitute a fixed and globally known set.  Rather, each
process node curates one or more "quorum slices", each constituting a set of
trusted processes. A "quorum" in Stellar is defined to be a set of nodes that
contain at least one quorum slice for each node in the set, such that agreement
can be reached.

Stellar [\[13\]][13]是以Ripple推行的解决方案为基础，它优化了联邦拜占庭协议模型，其中参与共识过程的并不构成一个固定的全局过程。 相反，每个进程节点组织一个或多个“仲裁片”，每个“仲裁片”构成一组可信进程。 Stellar中的“法定人数”被定义为一组包含至少个个节点的一个仲裁片。从而可达成一致。

The security of the Stellar mechanism relies on the assumption that the
intersection of *any* two quorums is non-empty, while the availability of a node
requires at least one of its quorum slices to consist entirely of correct nodes,
creating a trade-off between using large or small quorum-slices that may be
difficult to balance without imposing significant assumptions about trust.
Ultimately, nodes must somehow choose adequate quorum slices for there to be
sufficient fault-tolerance (or any "intact nodes" at all, of which much of the
results of the paper depend on), and the only provided strategy for ensuring
such a configuration is hierarchical and similar to the Border Gateway Protocol
(BGP), used by top-tier ISPs on the internet to establish global routing tables,
and by that used by browsers to manage TLS certificates; both notorious for
their insecurity.

恒星机制的安全性依赖于任何两个仲裁的交集都是非空的假设，同时，节点的可用性要求至少一个“仲裁片”完全由诚实节点组成。这就需要在“法定人数”的大小上作出妥协：人数过少难以获得共识，人数过多则难以信任所有人。可能难以平衡而不对信任做出重大假设。 除此之外，节点必须维护一定数量的仲裁片以获得足够的容错能力（或者任何“完整节点”，大部分结果的依赖），并且提供一种层级化的配置策略，类似于边界网关协议（Border Gateway Protocol,BGP）。BGP被互联网服务供应商（Internet Service Provider，ISP）用来建立全球路由表，也被浏览器用来管理传输层安全协议（Transport Layer Security，TLS）证书。他们都因为不安全而臭名昭着。

The criticism in the Stellar paper of the Tendermint-based proof-of-stake
systems is mitigated by the token strategy described here, wherein a new type of
token called the _atom_ is issued that represent claims to future portions of
fees and rewards. The advantage of Tendermint-based proof-of-stake, then, is its
relative simplicity, while still providing sufficient and provable security
guarantees.

对于Stellar论文中基于Tendermint的PoS批评可以通过本文描述的代币策略来缓解。本文提出了名为 _atom_ 的新的代币，它代表未来交易过程中产生的费用和奖励。 基于Tendermint PoS的优势在于其原理相对简单，同时仍可充分保证和证明安全性。

#### BitcoinNG

BitcoinNG is a proposed improvement to Bitcoin that would allow for forms of
vertical scalability, such as increasing the block size, without the negative
economic consequences typically associated with such a change, such as the
disproportionately large impact on small miners.  This improvement is achieved
by separating leader election from transaction broadcast: leaders are first
elected by proof-of-work in "key-blocks", and then able to broadcast
transactions to be committed until a new key-block is found. This reduces the
bandwidth requirements necessary to win the PoW race, allowing small miners to
more fairly compete, and allowing transactions to be committed more regularly by
the last miner to find a key-block.

BitcoinNG是对比特币的改进，允许垂直扩展，比如增加块的大小而避免带来负面的经济后果，例如对矿工造成的影响严重不成比例。 这种改进是通过将leader选举与交易广播分开来实现的：leader首先通过“微块micro-blocks”的PoW选举，然后leader能够广播交易直至下一个新的“微块”。 这减少了赢得PoW比赛所需的带宽要求，使矿主竞争更公平，并通过允许最后一名矿工提交微块以加快交易提交频率。

#### Casper

Casper [\[16\]][16] is a proposed proof-of-stake consensus algorithm for
Ethereum.  Its prime mode of operation is "consensus-by-bet".  By letting
validators iteratively bet on which block they believe will become committed
into the blockchain based on the other bets that they have seen so far,
finality can be achieved eventually.
[link](https://blog.ethereum.org/2015/12/28/understanding-serenity-part-2-casper/).
This is an active area of research by the Casper team.  The challenge is in
constructing a betting mechanism that can be proven to be an evolutionarily
stable strategy.  The main benefit of Casper as compared to Tendermint may be in
offering "availability over consistency" -- consensus does not require a >⅔
quorum of voting power -- perhaps at the cost of commit speed or
implementation complexity.

Casper [\[16\]][16]是以太坊提出的PoS共识算法。 它的主要操作模式是“预测押注”的一致。 通过让验证者基于他们目前看到的其他投注来迭代地下注他们认为哪个块将提交入区块链。最终性可以立即实现。 [链接](https://blog.ethereum.org/2015/12/28/understanding-serenity-part-2-casper/). 这是Casper团队研究的一个热点领域。 挑战在于构建一个可以证明是一个演化稳定策略的投注机制。 与Tendermint相比，Casper的主要优势可能在于提供“可用性超越一致性” — 达成共识不需要超过50％的投票权 — 可能是以提交速度或实现复杂性为代价的。

### Horizontal Scaling | 水平扩展

#### Interledger Protocol | Interledger 协议

The Interledger Protocol [\[14\]][14] is not strictly a scalability solution.
It provides an ad hoc interoperation between different ledger systems through a
loosely coupled bilateral relationship network.  Like the Lightning Network,
the purpose of ILP is to facilitate payments, but it specifically focuses on
payments across disparate ledger types, and extends the atomic transaction
mechanism to include not only hash-locks, but also a quorum of notaries (called
the Atomic Transport Protocol).  The latter mechanism for enforcing atomicity
in inter-ledger transactions is similar to Tendermint's light-client SPV
mechanism, so an illustration of the distinction between ILP and Cosmos/IBC is
warranted, and provided below.

Interledger协议（The Interledger Protocol，ILP）[\[14\]][14]不是一种严格的扩展方案。 它通过一个松散耦合的双边关系网络，提供一种指定的跨不同账本系统交互操作。像闪电网络一样，ILP的目的是实现支付，但是它特别关注跨账本类型的支付，并扩展原子事务的处理机制，使得事务的处理不仅支持哈希锁，而且还包括法定人数的公证人（称为原子运输协议）。 后者在账本间交易中实施原子性的机制与Tendermint的轻客户SPV机制类似，因此比较ILP和Cosmos / IBC之间的区别是有必要的，具体见下文。

1.The notaries of a connector in ILP do not support membership changes, and
   do not allow for flexible weighting between notaries.  On the other hand,
IBC is designed specifically for blockchains, where validators can have
different weights, and where membership can change over the course of the
blockchain.

1.ILP不支持连接器公证员的变更，也不允许公证员之间有灵活的权重。 另一方面，IBC是专门为区块链设计的，验证人可以拥有不同的权重，并且随着区块链的发展，成员可以随时更改。

2.As in the Lightning Network, the receiver of payment in ILP must be online to
   send a confirmation back to the sender.  In a token transfer over IBC, the
validator-set of the receiver's blockchain is responsible for providing
confirmation, not the receiving user.

2.与闪电网络一样，ILP中的接收人必须在线才能向发起人发送确认。 在IBC代币传输中，接收者所在区块链的验证人集合负责提供确认，而不是接收用户本人。

3.The most striking difference is that ILP's connectors are not responsible or
   keeping authoritative state about payments, whereas in Cosmos, the validators
of a hub are the authority of the state of IBC token transfers as well as the
authority of the amount of tokens held by each zone (but not the amount of
tokens held by each account within a zone).  This is the fundamental innovation
that allows for secure asymmetric transfer of tokens from zone to zone; the
analog to ILP's connector in Cosmos is a persistent and maximally secure
blockchain ledger, the Cosmos Hub.

3.最大的不同在于ILP连接器不需要负责对支付状态保持权威性，然而在Cosmos中，hub的验证人负责IBC代币传输状态和每个zone内持有代币数量的权威性。允许从zone之间的安全地不对称地交换代币是本质的创新。在cosmos中的ILP连接器可以看作是一个持久和最安全的区块链分类账：cosmos hub。

4.The inter-ledger payments in ILP need to be backed by an exchange orderbook,
   as there is no asymmetric transfer of coins from one ledger to another, only
the transfer of value or market equivalents.

4.ILP内的跨账本支付需要一个交易所的指令集的支持。因为不存在从一个分类账到另一个分类账不对称的代币转移，只能做到市场等价物的转移。

#### Sidechains | 侧链

Sidechains [\[15\]][15] are a proposed mechanism for scaling the Bitcoin
network via alternative blockchains that are "two-way pegged" to the Bitcoin
blockchain. (Two-way pegging is equivalent to bridging. In Cosmos we say
"bridging" to distinguish from market-pegging).  Sidechains allow bitcoins to
effectively move from the Bitcoin blockchain to the sidechain and back, and
allow for experimentation in new features on the sidechain.  As in the Cosmos
Hub, the sidechain and Bitcoin serve as light-clients of each other, using SPV
proofs to determine when coins should be transferred to the sidechain and back.
Of course, since Bitcoin uses proof-of-work, sidechains centered around Bitcoin
suffer from the many problems and risks of proof-of-work as a consensus
mechanism.  Furthermore, this is a Bitcoin-maximalist solution that doesn't
natively support a variety of tokens and inter-zone network topology as Cosmos
does. That said, the core mechanism of the two-way peg is in principle the same
as that employed by the Cosmos network.

Sidechains [\[15\]][15]是一种通过使用与比特币区块链“双向挂钩”替代区块链来扩展比特币网络性能的机制。（双向挂钩相当于桥接，在cosmos中被称为“桥接”与市场挂钩区分）。侧链使得比特币可以方便的在比特币区块链和侧链间移动，并允许在侧链上实验新功能。在Cosmos Hub中，侧链和比特币是彼此的轻客户端，在比特币区块链和侧链间移动时使用SPV证明。当然，由于比特币使用PoW，以比特币为中心的侧链遭受许多由于PoW作为共识机制的引起的问题和风险。而且，这是一个比特币利益最大化的解决方案，并不像Cosmos那样本地支持各种代币和zone间网络拓扑结构。但是，双向挂钩的核心机制原则上与Cosmos所采用的机制相同。

#### Ethereum Scalability Efforts | 以太坊扩展性的努力

Ethereum is currently researching a number of different strategies to shard the
state of the Ethereum blockchain to address scalability needs. These efforts
have the goal of maintaining the abstraction layer offered by the current
Ethereum Virtual Machine across the shared state space. Multiple research
efforts are underway at this time. [\[18\]][18][\[22\]][22]

以太坊目前正在研究许多不同的战略，将以太坊区块链的状态分区化，以解决可扩展性的需求。 这些努力的目标是在共享状态空间之上，维持当前以太坊虚拟机提供抽象层。 目前，多项研究工作正在进行。[\[18\]][18][\[22\]][22]

##### Cosmos vs Ethereum 2.0 Mauve

Cosmos and Ethereum 2.0 Mauve [\[22\]][22] have different design goals.

* Cosmos is specifically about tokens.  Mauve is about scaling general computation.
* Cosmos is not bound to the EVM, so even different VMs can interoperate.
* Cosmos lets the zone creator determine who validates the zone.
* Anyone can start a new zone in Cosmos (unless governance decides otherwise).
* The hub isolates zone failures so global token invariants are preserved.

Cosmos 和 Ethereum 2.0 Mauve [\[22\]][[22] 有不同的设计理念。

* Cosmos是针对代币儿 Mauve 是关于扩大计算能力。
* Cosmos不仅限于 EVM, 所以即使不同的VM也可以交互。
* Cosmos 让zone的创建者决定验证人。
* 任何人都可以在Cosmos中建立新的zone（除非管理者另做决定）。
* hub与zone的失效隔离，所以全局的代币不变量可以保持。

### General Scaling | 普遍扩展

#### Lightning Network | 闪电网络

The Lightning Network is a proposed token transfer network operating at a layer
above the Bitcoin blockchain (and other public blockchains), enabling improvement of many
orders of magnitude in transaction throughput by moving the majority
of transactions outside of the consensus ledger into so-called "payment
channels". This is made possible by on-chain cryptocurrency scripts, which
enable parties to enter into bilateral stateful contracts where the state can
be updated by sharing digital signatures, and contracts can be closed by finally
publishing evidence onto the blockchain, a mechanism first popularized by
cross-chain atomic swaps.  By opening payment channels with many parties,
participants in the Lightning Network can become focal points for routing the
payments of others, leading to a fully connected payment channel network, at the
cost of capital being tied up on payment channels.

闪电网络被设计成一种代币传输网络，在比特币区块链（及其他公有区块链）上一层运行，通过把大部分交易从共识分类账之外转移到所谓的“ 付款渠道“。 这通过链上加密货币脚本来实现，这些脚本使双方能够进入双方持有的状态化合同，通过共享数字签名来更新状态，并且在合同结束后最终通过在区块链上发布证据，这种机制首先受到跨链原子互换交易的欢迎。 通过与多方开通支付渠道，闪电网络的参与者可以成为集中点，为其他人的支付提供路由，从而导致支付渠道网络的完全联通，其代价是绑定在支付渠道上的资金。

While the Lightning Network can also easily extend across multiple independent
blockchains to allow for the transfer of _value_ via an exchange market, it
cannot be used to asymmetrically transfer _tokens_ from one blockchain to
another.  The main benefit of the Cosmos network described here is to enable
such direct token transfers.  That said, we expect payment channels and the
Lightning Network to become widely adopted along with our token transfer
mechanism, for cost-saving and privacy reasons.

虽然闪电网络也可以轻松地跨越多个独立的区块链，并借助交易市场实现价值转移，但它不能实现从一个区块链到另一个区块链的非对称代币交易。 这里描述的Cosmos网络的主要优点是实现直接的代币交换。 也就是说，我们希望支付渠道和闪电网络将会与我们的代币传输机制一起被广泛采用，从而节省成本和保护隐私。

#### Segregated Witness | 隔离验证人

Segregated Witness is a Bitcoin improvement proposal
[link](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki) that aims
to increase the per-block transaction throughput 2X or 3X, while simultaneously
making block syncing faster for new nodes.  The brilliance of this solution is
in how it works within the limitations of Bitcoin's current protocol and allows
for a soft-fork upgrade (i.e. clients with older versions of the software will
continue to function after the upgrade).  Tendermint, being a new protocol, has no
design restrictions, so it has a different scaling priorities.  Primarily,
Tendermint uses a BFT round-robin algorithm based on cryptographic signatures
instead of mining, which trivially allows horizontal scaling through multiple
parallel blockchains, while regular, more frequent block commits allow for
vertical scaling as well.

隔离见证是一个比特币改进建议BIP，[链接](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki) 旨在将每块交易吞吐量提高2倍或3倍，同时使新节点能更快同步区块。 这个解决方案的亮点在于它如何在比特币当前协议的局限下，允许软分叉升级（例如，具有旧版本软件的客户端将在升级后仍可继续运行）。 Tendermint作为一个新的协议没有设计的限制，所以它具有不同的扩展优先级。 TendermintBFT的循环算法，主要基于加密签名而不是采矿，这种算法允许通过多个并行区块链进行水平扩展，而更常规、更频繁的区块提交也允许垂直扩展。

<hr/>

## Appendix | 附录 ####################################################################

### Fork Accountability | 分叉问责制

A well designed consensus protocol should provide some guarantees in the event that the tolerance
capacity is exceeded and the consensus fails.  This is especially necessary in
economic systems, where Byzantine behaviour can have substantial financial
reward.  The most important such guarantee is a form of _fork-accountability_,
where the processes that caused the consensus to fail (ie.  caused clients of
the protocol to accept different values - a fork) can be identified and punished
according to the rules of the protocol, or, possibly, the legal system.  When
the legal system is unreliable or excessively expensive to invoke, validators can be forced to make security
deposits in order to participate, and those deposits can be revoked, or slashed,
when malicious behaviour is detected [\[10\]][10].

经过精心设计的共识协议应该能够在超出容错能力或者共识出错的情况下为系统提供一定的保障。这在能够通过拜占庭行为获取实质经济回报的金融系统中显得尤为必要。_分叉问责制_ 就属于这种非常重要的保障机制，这种制度会使一个导致共识出错的进程（例如使协议客户端开始接受不同值——即分叉）被识别出来，并且根据协议规则对其进行惩罚，甚至将其移送至司法系统处置。但当司法系统不可靠或者诉讼费极其昂贵时，为了让验证人们都参与到这个机制当中，该制度会强制要求他们建立安全保证金，一旦检测到恶意行为，那么这些保证金将会被罚没或者削减[\[10\]][10].

Note this is unlike Bitcoin, where forking is a regular occurrence due to
network asynchrony and the probabilistic nature of finding partial hash
collisions.  Since in many cases a malicious fork is indistinguishable from a
fork due to asynchrony, Bitcoin cannot reliably implement fork-accountability,
other than the implicit opportunity cost paid by miners for mining an orphaned
block.

注意这与比特币有所不同，由于网络非同步与局部哈希碰撞的概率特性，比特币的分叉是定期出现的。因为在很多情况下，恶意分叉与非同步引起的分叉是无法分辨的，除非让矿工为挖到的孤立区块支付隐性的机会成本，否则比特币就很难准确地执行分叉问责制。

### Tendermint Consensus | 共识

We call the voting stages _PreVote_ and _PreCommit_. A vote can be for a
particular block or for _Nil_.  We call a collection of >⅔ PreVotes for a single
block in the same round a _Polka_, and a collection of >⅔ PreCommits for a
single block in the same round a _Commit_.  If >⅔ PreCommit for Nil in the same
round, they move to the next round.

我们把投票阶段分为 _预投票_ 和 _预提交_ 两个阶段。一个投票可以是用于特定区块，也可以用于 _Nil_。我们把同一轮超过⅔的单个区块的预投票总和称为 _Polka_ ，把同一轮超过⅔的单个区块的预提交总和称为 _Commit_。如果在同一轮中对Nil的预提交超过⅔，他们就进入到下一轮中。

Note that strict determinism in the protocol incurs a weak synchrony assumption
as faulty leaders must be detected and skipped.  Thus, validators wait some
amount of time, _TimeoutPropose_, before they Prevote Nil, and the value of
TimeoutPropose increases with each round.  Progression through the rest of a
round is fully asynchronous, in that progress is only made once a validator hears
from >⅔ of the network.  In practice, it would take an extremely strong
adversary to indefinitely thwart the weak synchrony assumption (causing the
consensus to fail to ever commit a block), and doing so can be made even more
difficult by using randomized values of TimeoutPropose on each validator.

注意到协议中的严格的确定性会引发一个弱同步假设，因为错误的发起者必须被检测到并将其略过。验证人们在为Nil预投票之前会等待一段时间，称之为超时提议，这个超时提议的hi等待时间也会随着每一轮的进行而递增。每一轮的进行都是完全不同步的，在这个过程中，只有当验证人收听到超过⅔的网络投票才能进入到下一轮中。实际上，它需要极其强大的阻碍才能阻挠这个弱同步假设（导致无达成共识，无法提交区块），而且通过每个验证人超时提议（TimeoutPropose）的随机值还可更加提高这么做的难度。

An additional set of constraints, or Locking Rules, ensure that the network will
eventually commit just one block at each height. Any malicious attempt to cause
more than one block to be committed at a given height can be identified.  First,
a PreCommit for a block must come with justification, in the form of a Polka for
that block. If the validator has already PreCommit a block at round
<em>R_1</em>, we say they are _locked_ on that block, and the Polka used to
justify the new PreCommit at round <em>R_2</em> must come in a round
<em>R_polka</em> where <em>R_1 &lt; R_polka &lt;= R_2</em>.  Second, validators
must Propose and/or PreVote the block they are locked on.  Together, these
conditions ensure that a validator does not PreCommit without sufficient
evidence as justification, and that validators which have already PreCommit
cannot contribute to evidence to PreCommit something else.  This ensures both
safety and liveness of the consensus algorithm.

另一个附加的约束条件，或者叫锁定条例，它能确保网络最终在每个高度只提交一个区块。任何试图在给定高度提交超过一个区块的恶意行为都会被识别出来。首先，一个区块的预提交必须被认为是正当的，并且以Polka的形式提交。如果验证人已经准备在R_1轮中预提交一个区块，我们称他们锁定了这个区块，并且然后用于验证R_2轮新预提交动作的Polka必须进入R_polka轮，其中 R_1 < R_polka <= R_2。其次，验证人们必须为他们锁定的区块提议并且（或者）预投票。这两个条件共同作用，确保了验证人在对其正当性没有充分论证的情况下不能进行预提交操作，并且保证已经完成预提交的验证人不能再为其他东西的预提交贡献证明投票。这样不但可以保证共识算法的安全性，还能保证它的活跃度。

The full details of the protocol are described
[here](https://github.com/tendermint/tendermint/wiki/Byzantine-Consensus-Algorithm).

关于这一协议的全部细节参考[这里](https://github.com/tendermint/tendermint/wiki/Byzantine-Consensus-Algorithm).

### Tendermint Light Clients | Tendermint轻客户端

The need to sync all block headers is eliminated in Tendermint-PoS as the
existence of an alternative chain (a fork) means ≥⅓ of bonded stake can be
slashed.  Of course, since slashing requires that _someone_ share evidence of a
fork, light clients should store any block-hash commits that it sees.
Additionally, light clients could periodically stay synced with changes to the
validator set, in order to avoid [long range
attacks](#preventing-long-range-attacks) (but other solutions are possible).

因为生成侧链（一个分叉）意味着至少⅓的担保权益被罚没，Tendermint权益证明（Tendermint-PoS）取消了同步所有区块头的要求。当然，罚没保证金也是需要有人共享分叉证据的，所以轻客戸端就要存储任何它见证的区块哈希提交。另外，轻客戸端可以定期地与验证人组的改变保持同步，以避免出现远程攻击（但是其他解决方案也具有可能性）。

In spirit similar to Ethereum, Tendermint enables applications to embed a
global Merkle root hash in each block, allowing easily verifiable state queries
for things like account balances, the value stored in a contract, or the
existence of an unspent transaction output, depending on the nature of the
application.

与以太坊类似，Tendermint能够让应用程序在每个区块中嵌入一个全局梅克尔根的哈希值，可以简单方便的验证的状态查询，比如查询账户余额、在智能合约中的值，或者未使用交易输出（UTXO）的存在，具体由应用程序的特性决定。

### Preventing Long Range Attacks | 远程攻击的防御

Assuming a sufficiently resilient collection of broadcast networks and a static
validator set, any fork in the blockchain can be detected and the deposits of
the offending validators slashed.  This innovation, first suggested by Vitalik
Buterin in early 2014, solves the nothing-at-stake problem of other
proof-of-stake cryptocurrencies (see [Related Work](#related-work)). However,
since validator sets must be able to change, over a long range of time the
original validators may all become unbonded, and hence would be free to create a
new chain from the genesis block, incurring no cost as they no longer have
deposits locked up.  This attack came to be known as the Long Range Attack (LRA),
in contrast to a Short Range Attack, where validators who are currently bonded
cause a fork and are hence punishable (assuming a fork-accountable BFT algorithm
like Tendermint consensus). Long Range Attacks are often thought to be a
critical blow to proof-of-stake.

假设有一个足够有弹性的广播网络采集与一组静态验证人组存在，那么任何区块链分叉都可以被检测到，而且发起攻击的验证人提交的保证金会被罚没。这个由Vitalik Buterin在2014年首次提出的新方法，解决了其他权益证明加密货币的"没有任何相关权益"问题（详见 [相关工作部分](#related-work)）。但由于验证人组必须是能够变化的，在较长的时间段内最初的一些验证人会解除j押金绑定，这就使他们可以自由地从创世区块中创建新链，并且因为他们不再有被锁定的保证金，他们将不需要为这个行为支付任何费用。这类攻击被称为远程攻击（LRA），与短程攻击相比，后者对处在押金绑定中的验证人发起的分叉是可以对其进行惩罚的（假设有类似Tendermint共识这样的分叉问责制拜占庭容错算法）。所以远程攻击经常被认为是对权益证明机制的一种危险打击。

Fortunately, the LRA can be mitigated as follows.  First, for a validator to
unbond (thereby recovering their collateral deposit and no longer earning fees
to participate in the consensus), the deposit must be made untransferable for an
amount of time known as the "unbonding period", which may be on the order of
weeks or months.  Second, for a light client to be secure, the first time it
connects to the network it must verify a recent block-hash against a trusted
source, or preferably multiple sources.  This condition is sometimes referred to
as "weak subjectivity".  Finally, to remain secure, it must sync up with the
latest validator set at least as frequently as the length of the unbonding
period. This ensures that the light client knows about changes to the validator
set before a validator has its capital unbonded and thus no longer at stake,
which would allow it to deceive the client by carrying out a long range attack
by creating new blocks beginning back at a height where it was bonded (assuming
it has control of sufficiently many of the early private keys).

幸运的是，远程攻击（LRA）可以用以下的途径来缓解。第一，对于解除绑定的验证人而言（取回抵押保证金并且不再从参与共识中获取费用），保证金在一定时间内不能转移，也可以称其为"解绑周期"，这个周期可能长达数周或数月。第二，对于轻客户端的安全性而言，其首次连接到网络时必须根据可信源验证最新的一个区块哈希或者多个最好的区块哈希。这种情况有时被称为"弱主观性"。最后，为了保证安全，必须与最新的验证人组进行频繁的同步，时长与解绑周期一致。这样就确保了轻客戸端在因验证人解绑资金而失去任何权益之前，知道验证人组的变化情况，否则解绑的验证人就会在其绑定的高度后面开始创建新区块来实施远程攻击，以此来欺骗客户端（假设它可以控制足够多的早期私钥）。

Note that overcoming the LRA in this way requires an overhaul of the original
security model of proof-of-work. In PoW, it is assumed that a light client can
sync to the current height from the trusted genesis block at any time simply by
processing the proof-of-work in every block header.  To overcome the LRA,
however, we require that a light client come online with some regularity to
track changes in the validator set, and that the first time they come online
they must be particularly careful to authenticate what they hear from the
network against trusted sources. Of course, this latter requirement is similar
to that of Bitcoin, where the protocol and software must also be obtained from a
trusted source.

注意到用这样的方式来对抗远程攻击（LRA）需要对工作量证明（proof-of-work）的原始安全模块进行彻底检查。在工作量证明中（PoW），一个轻客户端可以通过在每一个区块头中运行工作量证明，以此简便地在任何时候与可信任的创始区块的当前高度进行同步。但是，为了对抗远程攻击（LRA），我们需要轻客戸端定期上线追踪验证人组的变动，其中在首次上线时必须格外仔细地根据可靠源来验证从网络采集的信息。诚然，后面这个要求和比特币类似，其协议和软件也必须从可靠源获得。

The above method for preventing LRA is well suited for validators and full nodes
of a Tendermint-powered blockchain because these nodes are meant to remain
connected to the network.  The method is also suitable for light clients that
can be expected to sync with the network frequently.  However, for light clients
that are not expected to have frequent access to the internet or the blockchain
network, yet another solution can be used to overcome the LRA.  Non-validator
token holders can post their tokens as collateral with a very long unbonding
period (e.g. much longer than the unbonding period for validators) and serve
light clients with a secondary method of attesting to the validity of current
and past block-hashes. While these tokens do not count toward the security of
the blockchain's consensus, they nevertheless can provide strong guarantees for
light clients.  If historical block-hash querying were supported in Ethereum,
anyone could bond their tokens in a specially designed smart contract and
provide attestation services for pay, effectively creating a market for
light-client LRA security.

以上这些为了防止远程攻击的方法，比较好地适用于由Tendermint驱动下的区块链验证人节点以及全节点，因为这些节点需要保持与网络的连接。这些方法同样适用于希望频繁地与网络同步的轻客户端。但是，对于那些不希望频繁接入互联网或者区块链网络的轻客户端来说，还有另一种方法可以解决远程攻击的问题。非验证人节点可以在很长的解绑期内（比如比验证人的解绑期更久）使用代币作为保证金，并且为轻客戸端提供二级证明当前有效性以及过去区块哈希的解决方案。虽然这些代币对于区块链共识的安全性并没有价值，不过他们还是可以为轻客戸端提供强大的保障。如果在以太坊中支持历史区块哈希查询，那么任何人都可以用特定的智能合约来绑定他们的代币，并且提供付费证明服务，从而有效地针对轻客戸端LRA安全问题开发出一个市场。

### Overcoming Forks and Censorship Attacks | 克服分叉与审查攻击

Due to the definition of a block commit, any ≥⅓ coalition of voting power can
halt the blockchain by going offline or not broadcasting their votes. Such a
coalition can also censor particular transactions by rejecting blocks that
include these transactions, though this would result in a significant proportion
of block proposals to be rejected, which would slow down the rate of block
commits of the blockchain, reducing its utility and value. The malicious
coalition might also broadcast votes in a trickle so as to grind blockchain
block commits to a near halt, or engage in any combination of these attacks.
Finally, it can cause the blockchain to fork, by double-signing or violating the
locking rules.

由于提交区块流程的定义，任何联合后不少于⅓的投票权的节点都可以通过下线或者不广播选票来中止区块链运行。这样的联合也可以通过拒绝包含这些交易的区块来审查特定的交易，尽管这将导致大多数区块提案被拒绝，致使区块提交速率减缓，降低了它的实用性与价值。恶意的联合或许仍然会陆陆续续地广播选票，用阻挠区块链的区块提交来将其逼停，或者使用任何这些攻击的组合攻击。最终，它会通过双重签名或者违反锁定规则来造成区块链分叉。

If a globally active adversary were also involved, it could partition the network in
such a way that it may appear that the wrong subset of validators were
responsible for the slowdown. This is not just a limitation of Tendermint, but
rather a limitation of all consensus protocols whose network is potentially
controlled by an active adversary.

如果一个全球活跃的作恶者也参与进来，就会用可能出现错误的验证组人子集导致速度降低的方法来分割网络。这不只是Tendermint面临的局限性，更确切地说是所有被活跃敌对者控制了网络的共识协议所面临的局限性1。

For these types of attacks, a subset of the validators should coordinate through
external means to sign a reorg-proposal that chooses a fork (and any evidence
thereof) and the initial subset of validators with their signatures. Validators
who sign such a reorg-proposal forego their collateral on all other forks.
Clients should verify the signatures on the reorg-proposal, verify any evidence,
and make a judgement or prompt the end-user for a decision.  For example, a
phone wallet app may prompt the user with a security warning, while a
refrigerator may accept any reorg-proposal signed by +½ of the original
validators by voting power.

对于这些类型的攻击，验证人的子集应该通过外部的方式进行协调，以签署选择一个分叉（及关系到的所有证据）与带有签名的验证人的初始子集的重组提案。签署了这样一份重组提案的验证者，将放弃在所有其他分叉上属于他们的保证金。客户端应在重组提案中验证签名以及任何相关的证据，并作出判断或提示终端用户作出决定。例如，一个手机钱包app应该在可能接受任何由一半以上的初始验证人们通过投票权利签署的重组提案时，给予用户安全警告提示。

No non-synchronous Byzantine fault-tolerant algorithm can come to consensus when
≥⅓ of voting power are dishonest, yet a fork assumes that ≥⅓ of voting power
have already been dishonest by double-signing or lock-changing without
justification.  So, signing the reorg-proposal is a coordination problem that
cannot be solved by any non-synchronous protocol (i.e. automatically, and
without making assumptions about the reliability of the underlying network).
For now, we leave the problem of reorg-proposal coordination to human
coordination via social consensus on internet media.  Validators must take care
to ensure that there are no remaining network partitions prior to signing a
reorg-proposal, to avoid situations where two conflicting reorg-proposals are
signed.

当多于⅓的投票权益是不诚实的时候，一个非同步的拜占庭容错算法步能够达成共识，然而，分叉假设不少于⅓的投票权益已经因不正当的双重签名或者锁定改变而成为不诚实的。因此，签署重组提案是一个协调问题，任何非同步协议都无法解决这个问题（也就是自动的，并且不考虑底层网络的可靠性）。目前，我们通过互联网媒体的社会共识，把重组提案的协调问题留给了用户去协调。验证人必须在签署重组提案之前就确保没有出现网络分割的问题，以此来避免签署两个相冲突的重组提议的情况发生。

Assuming that the external coordination medium and protocol is robust, it
follows that forks are less of a concern than censorship attacks.

假设外部协调媒介和协议是可靠的，对于分叉的担心会比审查攻击要少许多。

In addition to forks and censorship, which require ≥⅓ Byzantine voting power, a
coalition of >⅔ voting power may commit arbitrary, invalid state.  This is
characteristic of any (BFT) consensus system. Unlike double-signing, which
creates forks with easily verifiable evidence, detecting commitment of an
invalid state requires non-validating peers to verify whole blocks, which
implies that they keep a local copy of the state and execute each transaction,
computing the state root independently for themselves.  Once detected, the only
way to handle such a failure is via social consensus.  For instance, in
situations where Bitcoin has failed, whether forking due to software bugs (as in
March 2013), or committing invalid state due to Byzantine behavior of miners (as
in July 2015), the well connected community of businesses, developers, miners,
and other organizations established a social consensus as to what manual actions
were required by participants to heal the network.  Furthermore, since
validators of a Tendermint blockchain may be expected to be identifiable,
commitment of an invalid state may even be punishable by law or some external
jurisprudence, if desired.

除了需要大于⅓的拜占庭投票权益才能启动的分叉和审查制度以外，超过⅔的联合投票权益可能会提交任意、无效的状态。这是任何拜占庭容错算法的共识系统所特有的问题。与利用简单可验证证明来创建分叉的双重签名不同，检测无效状态的提交需要非验证节点来验证整个区块，这意味着非验证节点会保留一份本地的状态副本并执行每一笔交易，然后为他们自己独立计算出状态的根源。一旦检测出来，处理这类故障的唯一方法就是社会共识。打一个比方，在比特币出现问题的情况下，无论是由于软件漏洞造成的分叉（正如2013年3月），还是由于矿工拜占庭行为提交的无效状态（正如2015年7月），由商户、开发者、矿工和其他组织组成的联系紧密的社区所建立起来的社会共识会让他们按照分工来参与到修复网络的工作当中去。此外，由于Tendermint区块链的验证人身份是可识别的，那么如果需要的话，无效状态的提交实际上是可以被法律或其他外部法律体系惩治的。

### ABCI Specification | ABCI说明书

ABCI consists of 3 primary message types that get delivered from the core to the
application. The application replies with corresponding response messages.

ABCI由3种主要的信息类型组成，这三类信息从共识引擎传递到应用程序上，然后应用程序用相应回复信息做出应答。

The `AppendTx` message is the work horse of the application. Each transaction in
the blockchain is delivered with this message. The application needs to validate
each transactions received with the AppendTx message against the current state,
application protocol, and the cryptographic credentials of the transaction. A
validated transaction then needs to update the application state — by binding a
value into a key values store, or by updating the UTXO database.

`AppendTx` 信息是应用程序的主要传递媒介。区块链中的每一笔交易都通过这个信息来传递。应用程序需要验证每笔交易，这将通过接收针对当前状态、应用协议和交易密码证书的AppendTx信息来实现。验证过的交易将需要通过添加数值到键值存储或者更新UTXO数据库的方式来更新应用状态。

The `CheckTx` message is similar to AppendTx, but it’s only for validating
transactions. Tendermint Core’s mempool first checks the validity of a
transaction with CheckTx, and only relays valid transactions to its peers.
Applications may check an incrementing nonce in the transaction and return an
error upon CheckTx if the nonce is old.

`CheckTx`信息与AppendTx信息类似，但它只是为了交易验证。Tendermint Core的内存池会先用CheckTx验证交易有效性，并且只会将有效的交易传递给其他的节点。应用程序会检查交易序列号，如果序列号过期就会根据CheckTx返回一个错误。

The `Commit` message is used to compute a cryptographic commitment to the
current application state, to be placed into the next block header. This has
some handy properties. Inconsistencies in updating that state will now appear as
blockchain forks which catches a whole class of programming errors. This also
simplifies the development of secure lightweight clients, as Merkle-hash proofs
can be verified by checking against the block-hash, and the block-hash is signed
by a quorum of validators (by voting power).

`Commit`信息是用来计算之后会存入到下一区块头中的当前应用状态的加密提交项。这具有便利的特性。状态的前后矛盾性将会像引起程序错误，从而导致区块链分叉。这也简化了安全轻客戸端的开发，因为梅克尔哈希证明可以通过检查区块哈希来加以验证，而区块哈希是由规定人数的验证人们签署的（通过投票权益）。

Additional ABCI messages allow the application to keep track of and change the
validator set, and for the application to receive the block information, such as
the height and the commit votes.

此外，ABCI信息允许应用程序保持追踪验证人组的改变，并让应用程序接收诸如高度和提交选票之类的区块信息。

ABCI requests/responses are simple Protobuf messages.  Check out the [schema
file](https://github.com/tendermint/abci/blob/master/types/types.proto).

ABCI请求/响应是简单的Protobuf信息。请参考这里的[模式文件](https://github.com/tendermint/abci/blob/master/types/types.proto)。

##### AppendTx
  * __Arguments__:
    * `Data ([]byte)`: The request transaction bytes 交易请求信息
  * __Returns__:
    * `Code (uint32)`: Response code 回复代码
    * `Data ([]byte)`: Result bytes, if any
    * `Log (string)`: Debug or error message 错误信息
  * __Usage__:<br/>
    Append and run a transaction.  If the transaction is valid, returns
CodeType.OK 用途：提交并执行一笔交易
##### 附加交易
  * __命令行参数__:
    * `Data ([]byte)`: 交易请求信息
  * __Returns__:
    * `Code (uint32)`: 回复代码
    * `Data ([]byte)`: 结果字节，如果有的话
    * `Log (string)`:  错误信息
  * __使用__:<br/>
    提交并执行一笔交易，如果交易有效，那返回CodeType.OK

##### CheckTx
  * __Arguments__:
    * `Data ([]byte)`: The request transaction bytes 交易请求信息
  * __Returns__:
    * `Code (uint32)`: Response code 回复代码
    * `Data ([]byte)`: Result bytes, if any
    * `Log (string)`: Debug or error message 错误信息
  * __Usage__:<br/>
    Validate a transaction.  This message should not mutate the state.
    Transactions are first run through CheckTx before broadcast to peers in the
mempool layer.
    You can make CheckTx semi-stateful and clear the state upon `Commit` or
`BeginBlock`,
    to allow for dependent sequences of transactions in the same block. 
    
##### 检查交易
  * __命令行参数__:
    * `Data ([]byte)`: 交易请求信息
  * __Returns__:
    * `Code (uint32)`: 回复代码
    * `Data ([]byte)`: 结果字节，如果有的话
    * `Log (string)`:  错误信息
  * __使用__:<br/>
    验证一笔交易。这个信息不应该改变应用状态。交易在广播给其他节点前，首先通过CheckTx运行。你可以发起半状态化CheckTx，并在Commit or BeginBlock上清算状态，以允许序列执行同一区块中相关的交易。
    
##### Commit
  * __Returns__:
    * `Data ([]byte)`: The Merkle root hash 默克尔根值
    * `Log (string)`: Debug or error message
  * __Usage__:<br/>
    Return a Merkle root hash of the application state.返回当前应用状态。

##### 提交
  * __返回值__:
    * `Data ([]byte)`:  默克尔根值
    * `Log (string)`:  调试或出错信息
  * __使用__:<br/>
    返回当前应用状态。

##### Query
  * __Arguments__:
    * `Data ([]byte)`: The query request bytes 
  * __Returns__:
    * `Code (uint32)`: Response code
    * `Data ([]byte)`: The query response bytes
    * `Log (string)`: Debug or error message
    
##### 查询
  * __命令行参数__:
    * `Data ([]byte)`:  请求数据
  * __返回值__:
    * `Code (uint32)`: 回复代码
    * `Data ([]byte)`: 查询回复字节
    * `Log (string)`: 调试或出错信息

##### Flush
  * __Usage__:<br/>
    Flush the response queue.  Applications that implement `types.Application`
need not implement this message -- it's handled by the project.

##### 刷新
  * __使用__:<br/>
    刷新回复队列。应用types.Application的应用程序无需实施这条信息——这个由项目进行处理。

##### Info
  * __Returns__:
    * `Data ([]byte)`: The info bytes
  * __Usage__:<br/>
    Return information about the application state.  Application specific.

##### 信息
  * __返回值__:
    * `Data ([]byte)`: 信息字节串
  * __使用__:<br/>
    返回关于应用程序状态的信息.  应用指定。

##### SetOption
  * __Arguments__:
    * `Key (string)`: Key to set
    * `Value (string)`: Value to set for key
  * __Returns__:
    * `Log (string)`: Debug or error message
  * __Usage__:<br/>
    Set application options.  E.g. Key="mode", Value="mempool" for a mempool
connection, or Key="mode", Value="consensus" for a consensus connection.
    Other options are application specific.

#### 设置选项
  * __参数__:
    * `Key (string)`: 设置参数
    * `Value (string)`: 参数值
  * __返回值__:
    * `Log (string)`: Debug or error message
  * __使用__:<br/>
    比如，针对内存池的连接可以将键设置为"mode"（模式），值为"mempool"（内存池）。或者针对共识连接，将键设置为"mode"，值设置为"consensus"（共识）。其他选项根据可具体应用进行专门设置。
    
##### InitChain
  * __Arguments__:
    * `Validators ([]Validator)`: Initial genesis-validators
  * __Usage__:<br/>
    Called once upon genesis 在创世区块创建时调用
    
##### 初始链
  * __参数__:
    * `Validators ([]Validator)`: 初始化创世验证人
  * __使用__:<br/>
    在创世区块创建时调用

##### BeginBlock
  * __Arguments__:
    * `Height (uint64)`: The block height that is starting
  * __Usage__:<br/>
    Signals the beginning of a new block. Called prior to any AppendTxs.
    
##### BeginBlock
  * __参数__:
    * `Height (uint64)`: 区块刚开始的高度
  * __使用__:<br/>
    为新区块的开始提供信号。在附加交易（AppendTxs）前进行调用。

##### EndBlock
  * __Arguments__:
    * `Height (uint64)`: The block height that ended
  * __Returns__:
    * `Validators ([]Validator)`: Changed validators with new voting powers (0
      to remove)
  * __Usage__:<br/>
    Signals the end of a block.  Called prior to each Commit after all
transactions
##### EndBlock
  * __参数__:
    * `Height (uint64)`: 结束时的区块高度
  * __返回值__:
    * `Validators ([]Validator)`: 具有新选票的变动后的验证人（归零就去除）
  * __使用__:<br/>
     为区块结束提供信号。在每次提交前所有交易后调用。

See [the ABCI repository](https://github.com/tendermint/abci#message-types) for more details

更多细节请参考 [ABCI知识库](https://github.com/tendermint/abci#message-types)。

### IBC Packet Delivery Acknowledgement | IBC数据包交付确认

There are several reasons why a sender may want the acknowledgement of delivery
of a packet by the receiving chain.  For example, the sender may not know the
status of the destination chain, if it is expected to be faulty.  Or, the sender
may want to impose a timeout on the packet (with the `MaxHeight` packet field),
while any destination chain may suffer from a denial-of-service attack with a
sudden spike in the number of incoming packets.

发送者有很多需要接收链提供数据包交付确认的原因。比如，如果预计目的链会出错，那发送者就可能无法了解目的链的状态。或者，当目的链可能遇到因接收数据包猛烈增多而形成的拒绝服务攻击时，发送者会想要设定数据包超时时限（借助`MaxHeight` 即最大值包域）。

In these cases, the sender can require delivery acknowledgement by setting the
initial packet status to `AckPending`.  Then, it is the receiving chain's
responsibility to confirm delivery by including an abbreviated `IBCPacket` in the
app Merkle hash.

在这些案例中，发送人可以通过在`AckPending`上设置初始数据包状态来要求提供交付确认。然后就由接收链通过包含一个简化的`IBCPacket`的应用梅克尔哈希来确认交付。

![Figure of Zone1, Zone2, and Hub IBC with
acknowledgement](https://raw.githubusercontent.com/gnuclear/atom-whitepaper/master/msc/ibc_with_ack.png)

First, an `IBCBlockCommit` and `IBCPacketTx` are posted on "Hub" that proves
the existence of an `IBCPacket` on "Zone1".  Say that `IBCPacketTx` has the
following value:

首先，一个`IBCBlockCommit`和`IBCPacketTx`是被上传到“枢纽”上用来证明"分区1"上的`IBCPacket`的存在的。假设`IBCPacketTx`的值如下：

- `FromChainID`: "Zone1"
- `FromBlockHeight`: 100 (假设)
- `Packet`: an `IBCPacket`:
  - `Header`: an `IBCPacketHeader`:
    - `SrcChainID`: "Zone1"
    - `DstChainID`: "Zone2"
    - `Number`: 200 (say)
    - `Status`: `AckPending`
    - `Type`: "coin"
    - `MaxHeight`: 350 (假设 "枢纽" 当前高度为 300)
  - `Payload`: &lt;一个"代币"的有效负荷字节&gt;


- `FromChainID`: "Zone1"
- `FromBlockHeight`: 100 (假设)
- `Packet`: an `IBCPacket`:
  - `Header`: an `IBCPacketHeader`:
    - `SrcChainID`: "Zone1"
    - `DstChainID`: "Zone2"
    - `Number`: 200 (假设)
    - `Status`: `AckPending`
    - `Type`: "coin"
    - `MaxHeight`: 350 (假设"枢纽"目前的高度是300)
  - `Payload`: &lt;一个"代币"的有效负荷字节&gt;

Next, an `IBCBlockCommit` and `IBCPacketTx` are posted on "Zone2" that proves
the existence of an `IBCPacket` on "Hub".  Say that `IBCPacketTx` has the
following value:

其次，一个`IBCBlockCommit` 和 `IBCPacketTx`被传输都“分区2”上用来证明`IBCPacket`在“枢纽”上的存在。假设`IBCPacketTx`的值如下：

- `FromChainID`: "Hub"
- `FromBlockHeight`: 300
- `Packet`: an `IBCPacket`:
  - `Header`: an `IBCPacketHeader`:
    - `SrcChainID`: "Zone1"
    - `DstChainID`: "Zone2"
    - `Number`: 200
    - `Status`: `AckPending`
    - `Type`: "coin"
    - `MaxHeight`: 350
  - `Payload`: &lt;一个"代币"相同的有效负荷字节&gt;


- `FromChainID`: "Hub"
- `FromBlockHeight`: 300
- `Packet`: an `IBCPacket`:
  - `Header`: an `IBCPacketHeader`:
    - `SrcChainID`: "Zone1"
    - `DstChainID`: "Zone2"
    - `Number`: 200
    - `Status`: `AckPending`
    - `Type`: "coin"
    - `MaxHeight`: 350
  - `Payload`: &lt;一个"代币"相同的有效负荷字节&gt;

Next, "Zone2" must include in its app-hash an abbreviated packet that shows the
new status of `AckSent`.  An `IBCBlockCommit` and `IBCPacketTx` are posted back
on "Hub" that proves the existence of an abbreviated `IBCPacket` on
"Zone2".  Say that `IBCPacketTx` has the following value:

接下来，"Zone2"必须将缩写的来显示`AckSent`的最新状态包添加到应用程序状态哈希中。
`IBCBlockCommitand` 和`IBCPacketTx` 会传输到“枢纽"上来证明简化的`IBCPacket` 存在于"分区2"上。假设`IBCPacketTx` 的值如下：

- `FromChainID`: "Zone2"
- `FromBlockHeight`: 400 (假设)
- `Packet`: an `IBCPacket`:
  - `Header`: an `IBCPacketHeader`:
    - `SrcChainID`: "Zone1"
    - `DstChainID`: "Zone2"
    - `Number`: 200
    - `Status`: `AckSent`
    - `Type`: "coin"
    - `MaxHeight`: 350
  - `PayloadHash`: &lt;一个"代币"相同的有效负荷字节的哈希值&gt;


- `FromChainID`: "Zone2"
- `FromBlockHeight`: 400 (假设)
- `Packet`: an `IBCPacket`:
  - `Header`: an `IBCPacketHeader`:
    - `SrcChainID`: "Zone1"
    - `DstChainID`: "Zone2"
    - `Number`: 200
    - `Status`: `AckSent`
    - `Type`: "coin"
    - `MaxHeight`: 350
  - `PayloadHash`: &lt;一个"代币"相同的有效负荷字节的哈希值&gt;

Finally, "Hub" must update the status of the packet from `AckPending` to
`AckReceived`.  Evidence of this new finalized status should go back to
"Zone2".  Say that `IBCPacketTx` has the following value:

最后，“枢纽”必须更新从`AckPending` 到`AckReceived`的数据包状态。这个新完成状态的证明应该返回到“分区2”上。假设`IBCPacketTx`的值如下：

- `FromChainID`: "Hub"
- `FromBlockHeight`: 301
- `Packet`: an `IBCPacket`:
  - `Header`: an `IBCPacketHeader`:
    - `SrcChainID`: "Zone1"
    - `DstChainID`: "Zone2"
    - `Number`: 200
    - `Status`: `AckReceived`
    - `Type`: "coin"
    - `MaxHeight`: 350
  - `PayloadHash`: &lt;The hash bytes of the same "coin" payload&gt;


- `FromChainID`: "Hub"
- `FromBlockHeight`: 301
- `Packet`: an `IBCPacket`:
  - `Header`: an `IBCPacketHeader`:
    - `SrcChainID`: "Zone1"
    - `DstChainID`: "Zone2"
    - `Number`: 200
    - `Status`: `AckReceived`
    - `Type`: "coin"
    - `MaxHeight`: 350
  - `PayloadHash`: &lt;相同"代币"有效负荷的哈希字节&gt;
  -
Meanwhile, "Zone1" may optimistically assume successful delivery of a "coin"
packet unless evidence to the contrary is proven on "Hub".  In the example
above, if "Hub" had not received an `AckSent` status from "Zone2" by block
350, it would have set the status automatically to `Timeout`.  This evidence of
a timeout can get posted back on "Zone1", and any tokens can be returned.

与此同时，“分区1”会假设“代币”包的交付已经成功，除非"枢纽"上有证据给出相反的证明。在上述例子中，如果"枢纽"没有从"分区2"接收到第350个区块的`AckSent` 状态，那么它就会自动将其设置为`Timeout`（超时）。这个超时的证据可以贴回到"Zone1"上，然后所有代币都会被返还。

![Figure of Zone1, Zone2, and Hub IBC with acknowledgement and
timeout](https://raw.githubusercontent.com/gnuclear/atom-whitepaper/master/msc/ibc_with_ack_timeout.png)

### Merkle Tree & Proof Specification | 梅克尔树及梅克尔证明的说明

There are two types of Merkle trees supported in the Tendermint/Cosmos
ecosystem: The Simple Tree, and the IAVL+ Tree.

Tendermint/Cosmos生态支持的两种m默克尔树：简单树和IAVL+树。

#### Simple Tree | 简易版梅克尔树

The Simple Tree is a Merkle tree for a static list of elements.  If the number
of items is not a power of two, some leaves will be at different levels.  Simple
Tree tries to keep both sides of the tree the same height, but the left may be
one greater.  This Merkle tree is used to Merkle-ize the transactions of a
block, and the top level elements of the application state root.

简易版梅克尔树针对基础的静态列表。如果项目的数量不是2的次方，那么有些树叶就会在不同的层上。简易树试图让树的两侧在同一高度，但是左边可能会稍大一点。这种梅克尔树就是用于一个区块交易的梅克尔化的，而顶层元素就是应用状态的根。

```
                *
               / \
             /     \
           /         \
         /             \
        *               *
       / \             / \
      /   \           /   \
     /     \         /     \
    *       *       *       h6
   / \     / \     / \
  h0  h1  h2  h3  h4  h5

  A SimpleTree with 7 elements
```

#### IAVL+ Tree | IAVL+树

The purpose of the IAVL+ data structure is to provide persistent storage for
key-value pairs in the application state such that a deterministic Merkle root
hash can be computed efficiently.  The tree is balanced using a variant of the
[AVL algorithm](http://en.wikipedia.org/wiki/AVL_tree), and all operations are
O(log(n)).

IAVL+数据结构的目的是永久储存应用状态的键值对，这样就可以对确定的梅克尔根哈希进行高效的运算。这个树的平衡通过AVL算法的变体来实现，所有运行复杂度都是O(log(n))。

In an AVL tree, the heights of the two child subtrees of any node differ by at
most one.  Whenever this condition is violated upon an update, the tree is
rebalanced by creating O(log(n)) new nodes that point to unmodified nodes of the
old tree.  In the original AVL algorithm, inner nodes can also hold key-value
pairs.  The AVL+ algorithm (note the plus) modifies the AVL algorithm to keep
all values on leaf nodes, while only using branch-nodes to store keys.  This
simplifies the algorithm while keeping the merkle hash trail short.

无论在什么时候这种情况因为更新导致违背，这个树都会通过创造O(log(n))新节点（指向旧树上未修改的节点）来再次达到平衡。在初始的AVL算法中，内部节点也可以保留健值对。AVL+算法（注意这里有个"+"号）对AVL算法进行了修改，来维持所有数值都在树叶节点上，同时还只需采用分支节点来存储键。这样在维持较短的梅克尔哈希轨迹对的同时，还简化了算法。

The AVL+ Tree is analogous to Ethereum's [Patricia
tries](http://en.wikipedia.org/wiki/Radix_tree).  There are tradeoffs.  Keys do
not need to be hashed prior to insertion in IAVL+ trees, so this provides faster
ordered iteration in the key space which may benefit some applications.  The
logic is simpler to implement, requiring only two types of nodes -- inner nodes
and leaf nodes.  The Merkle proof is on average shorter, being a balanced binary
tree.  On the other hand, the Merkle root of an IAVL+ tree depends on the order
of updates.

AVL+树类似于以太坊的[帕氏树](http://en.wikipedia.org/wiki/Radix_tree)。其中也有一定的折中。键不需要在嵌入到IAVL+树之前生成哈希，所以这就为键空间提供了较快的顺序迭代，这会为很多应用程序带来好处。逻辑实现更简单，只需要内部节点和树叶节点这两种节点类型。作为一个平衡的二叉树，其梅克尔证明平均更短。而另一方面，IAVL+树的梅克尔根有取决于命令的更新。
我们将支持额外有效的梅克尔树，比如当二元变量可用时的以太坊帕氏树。

We will support additional efficient Merkle trees, such as Ethereum's Patricia
Trie when the binary variant becomes available.

我们将支持格外有效的梅克尔树，比如以太坊的帕氏树，同时实现二进制变量的可用性。


### Transaction Types |交易类型

In the canonical implementation, transactions are streamed to the Cosmos hub
application via the ABCI interface.

The Cosmos Hub will accept a number of primary transaction types, including
`SendTx`, `BondTx`, `UnbondTx`, `ReportHackTx`, `SlashTx`, `BurnAtomTx`,
`ProposalCreateTx`, and `ProposalVoteTx`, which are fairly self-explanatory and
will be documented in a future revision of this paper.  Here we document the two
primary transaction types for IBC: `IBCBlockCommitTx` and `IBCPacketTx`.

在标准的执行中，交易通过ABCI界面涌入Cosmos Hub的应用程序中。

Cosmos Hub将接收几类主要的交易类型，包括`SendTx`, `BondTx`, `UnbondTx`, `ReportHackTx`, `SlashTx`, `BurnAtomTx`,
`ProposalCreateTx`，以及`ProposalVoteTx`（发送交易、绑定交易、解绑交易、攻击报告交易、削减交易、Atom燃烧交易，创建提案交易、以及提案投票交易），这些都不需要加以说明，会在未来版本的文档中加以备案。这里我们主要列举两个主要的IBC交易类型： `IBCBlockCommitTx` 以及`IBCPacketTx`（即IBC区块提交交易以及IBC数据包交易）

#### IBCBlockCommitTx | IBC区块提交交易

An `IBCBlockCommitTx` transaction is composed of:

- `ChainID (string)`: The ID of the blockchain
- `BlockHash ([]byte)`: The block-hash bytes, the Merkle root which includes the
  app-hash
- `BlockPartsHeader (PartSetHeader)`: The block part-set header bytes, only
  needed to verify vote signatures
- `BlockHeight (int)`: The height of the commit
- `BlockRound (int)`: The round of the commit
- `Commit ([]Vote)`: The >⅔ Tendermint `Precommit` votes that comprise a block
  commit
- `ValidatorsHash ([]byte)`: A Merkle-tree root hash of the new validator set
- `ValidatorsHashProof (SimpleProof)`: A SimpleTree Merkle-proof for proving the
  `ValidatorsHash` against the `BlockHash`
- `AppHash ([]byte)`: A IAVLTree Merkle-tree root hash of the application state
- `AppHashProof (SimpleProof)`: A SimpleTree Merkle-proof for proving the
  `AppHash` against the `BlockHash`

IBCBlockCommitTx 交易主要由这些组成：

- `ChainID (string)`: 区块链ID
- `BlockHash ([]byte)`: 区块哈希字节，就是包括应用程序哈希梅克尔根
- `BlockPartsHeader (PartSetHeader)`: 区块部分设置的头字节，只用于验证投票签名
- `BlockHeight (int)`: 提交高度
- `BlockRound (int)`: 提交回合
- `Commit ([]Vote)`: 超过⅔的包括区块提交的Tendermint预提交投票
- `ValidatorsHash ([]byte)`: 新验证组的梅克尔树根哈希
- `ValidatorsHashProof (SimpleProof)`: 在区块哈希中证明验证人哈希的简易树梅克尔证明
- `AppHash ([]byte)`: IAVL树，应用程序状态的梅克尔树根哈希
- `AppHashProof (SimpleProof)`: 在区块哈希中验证应用程序哈希的简易版梅克尔树证明`AppHash` against the `BlockHash`

#### IBCPacketTx | IBC包交易

An `IBCPacket` is composed of:

- `Header (IBCPacketHeader)`: The packet header
- `Payload ([]byte)`: The bytes of the packet payload. _Optional_
- `PayloadHash ([]byte)`: The hash for the bytes of the packet. _Optional_

`IBCPacket` 由下列项组成：

- `Header (IBCPacketHeader)`: 数据包头
- `Payload ([]byte)`: 数据包有效负荷字节。可选择。
- `PayloadHash ([]byte)`: 数据包字节哈希。可选择。

Either one of `Payload` or `PayloadHash` must be present.  The hash of an
`IBCPacket` is a simple Merkle root of the two items, `Header` and `Payload`.
An `IBCPacket` without the full payload is called an _abbreviated packet_.

有效负荷`Payload`或有效负荷哈希`PayloadHash`必须存在一个。`IBCPacket` 的哈希就是两个项的简易版梅克尔根，即头和有效负荷。没有完整有效负荷的`IBCPacket` 被称作 _缩写版包_ 。


An `IBCPacketHeader` is composed of:

- `SrcChainID (string)`: The source blockchain ID
- `DstChainID (string)`: The destination blockchain ID
- `Number (int)`: A unique number for all packets
- `Status (enum)`: Can be one of `AckPending`, `AckSent`, `AckReceived`,
  `NoAck`, or `Timeout`
- `Type (string)`: The types are application-dependent.  Cosmos reserves the
  "coin" packet type
- `MaxHeight (int)`: If status is not `NoAckWanted` or `AckReceived` by this
  height, status becomes `Timeout`. _Optional_


`IBCPacketHeader`由下列项组成：

`SrcChainID (string)`: 源区块链 ID
DstChainID (string)`: 目标区块链ID
Number (int)`: 所有数据包的唯一数字
Status (enum)`:可以是AckPending，AckSent，AckReceived，NoAck，或Timeout任意一个
Type (string)`: 种类根据应用程序决定。Cosmos保留"coin"（币）包种类。
`MaxHeight (int)`: 如果状态不是这个高度给出的`NoAckWanted` 或者`AckReceived` ，那么状态就算超时。可选择。

An `IBCPacketTx` transaction is composed of:

- `FromChainID (string)`: The ID of the blockchain which is providing this
  packet; not necessarily the source
- `FromBlockHeight (int)`: The blockchain height in which the following packet
  is included (Merkle-ized) in the block-hash of the source chain
- `Packet (IBCPacket)`: A packet of data, whose status may be one of
  `AckPending`, `AckSent`, `AckReceived`, `NoAck`, or `Timeout`
- `PacketProof (IAVLProof)`: A IAVLTree Merkle-proof for proving the packet's
  hash against the `AppHash` of the source chain at given height

`IBCPacketTx` 交易有下列项组成：
- `FromChainID (string)`: 提供给这个数据包的区块链ID，不是源所必须的
- `FromBlockHeight (int)`:  区块链高度，其中接下来的包会包含在（梅克尔化的）源链的区块哈希中
- `Packet (IBCPacket)`:数据包, 其状态可能是`AckPending`, `AckSent`, `AckReceived`, `NoAck`, 或者 `Timeout`其中的一个
- `PacketProof (IAVLProof)`:  IAVL树梅克尔证明，用于验证在给定高度下的源链应用哈希中的数据包哈希

The sequence for sending a packet from "Zone1" to "Zone2" through the
"Hub" is depicted in {Figure X}.  First, an `IBCPacketTx` proves to
"Hub" that the packet is included in the app-state of "Zone1".  Then,
another `IBCPacketTx` proves to "Zone2" that the packet is included in the
app-state of "Hub".  During this procedure, the `IBCPacket` fields are
identical: the `SrcChainID` is always "Zone1", and the `DstChainID` is always
"Zone2".

通过"枢纽"，将数据包从"分区1"发送到"分区2"的序列，描述在{Figure X}函数中。首先，一个`IBCPacketTx`会向"Hub"证明数据包是包含在"分区1"的应用程序状态中。然后，另一个`IBCPacketTx` 会向"分区2"证明数据包包含在"Hub"的应用程序状态中。在这个过程中，`IBCPacketTx` 的字段是相同的：`SrcChainID`永远是"Zone1"，而`DstChainID` 永远是"Zone2"。

The `PacketProof` must have the correct Merkle-proof path, as follows:

`PacketProof` 必须有正确的梅克尔证明路径，如下：

```
IBC/<SrcChainID>/<DstChainID>/<Number>

```

When "Zone1" wants to send a packet to "Zone2" through "Hub", the
`IBCPacket` data are identical whether the packet is Merkle-ized on "Zone1",
the "Hub", or "Zone2".  The only mutable field is `Status` for tracking
delivery.

当“分区1”要通过“枢纽”将数据包传送到“分区2”中，无论数据包是否在“分区1”、“枢纽”、或者“分区2”中梅克尔化了，`IBCPacket`数据都是相同的。唯一易变的字段是为追踪交付的`Status`。

## Acknowledgements |鸣谢 ############################################################

We thank our friends and peers for assistance in conceptualizing, reviewing, and
providing support for our work with Tendermint and Cosmos.

我们为所有朋友欲同行们在概念成型与检查方面给予的帮助，以及对我们在Tendermint与Cosmos工作中的大力支持，表示衷心地感谢。

* [Zaki Manian](https://github.com/zmanian) of
  [SkuChain](https://www.skuchain.com/) provided much help in formatting and
wording, especially under the ABCI section
* [Jehan Tremback](https://github.com/jtremback) of Althea and Dustin Byington
  for helping with initial iterations
* [Andrew Miller](http://soc1024.com/) of [Honey
  Badger](https://eprint.iacr.org/2016/199) for feedback on consensus
* [Greg Slepak](https://fixingtao.com/) for feedback on consensus and wording
* Also thanks to [Bill Gleim](https://github.com/gleim) and [Seunghwan
  Han](http://www.seunghwanhan.com) for various contributions.
* __Your name and organization here for your contribution__

* [SkuChain](https://www.skuchain.com/)的[Zaki Manian](https://github.com/zmanian)在格式与措辞方面提供了很多帮助，尤其在ABCI部分。
* Althea的[Jehan Tremback](https://github.com/jtremback)和Dustin Byington在初始迭代方面的帮助。
* [Honey
  Badger](https://eprint.iacr.org/2016/199)的 [Andrew Miller](http://soc1024.com/) 在共识部分给予的反馈。
* [Greg Slepak](https://fixingtao.com/)对共识部分的反馈以及在措辞方面的帮助。
* 同时还要感谢 [Bill Gleim](https://github.com/gleim)和 [Seunghwan
  Han](http://www.seunghwanhan.com)在多方面的支持与贡献。
* **此处还有您及您的组织对本文的贡献。

## Citations | 引用 ###################################################################

[1]: https://bitcoin.org/bitcoin.pdf
[2]: http://zerocash-project.org/paper
[3]: https://github.com/ethereum/wiki/wiki/White-Paper
[4]: https://download.slock.it/public/DAO/WhitePaper.pdf
[5]: https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki
[6]: https://arxiv.org/pdf/1510.02037v2.pdf
[7]: https://lightning.network/lightning-network-paper-DRAFT-0.5.pdf
[8]: https://github.com/tendermint/tendermint/wiki
[9]: https://groups.csail.mit.edu/tds/papers/Lynch/jacm85.pdf
[10]: https://blog.ethereum.org/2014/01/15/slasher-a-punitive-proof-of-stake-algorithm/
[11]: http://pmg.csail.mit.edu/papers/osdi99.pdf
[12]: https://bitshares.org/technology/delegated-proof-of-stake-consensus/
[13]: https://www.stellar.org/papers/stellar-consensus-protocol.pdf
[14]: https://interledger.org/rfcs/0001-interledger-architecture/
[15]: https://blockstream.com/sidechains.pdf
[16]: https://blog.ethereum.org/2015/08/01/introducing-casper-friendly-ghost/
[17]: https://github.com/tendermint/abci
[18]: https://github.com/ethereum/EIPs/issues/53
[19]: http://www.ds.ewi.tudelft.nl/fileadmin/pds/papers/PerformanceAnalysisOfLibswift.pdf
[20]: http://groups.csail.mit.edu/tds/papers/Lynch/jacm88.pdf
[21]: https://en.bitcoin.it/wiki/Thin_Client_Security
[22]: http://vitalik.ca/files/mauve_paper.html

* [1] Bitcoin: https://bitcoin.org/bitcoin.pdf
* [2] ZeroCash: http://zerocash-project.org/paper
* [3] Ethereum: https://github.com/ethereum/wiki/wiki/White-Paper
* [4] TheDAO: https://download.slock.it/public/DAO/WhitePaper.pdf
* [5] Segregated Witness: https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki
* [6] BitcoinNG: https://arxiv.org/pdf/1510.02037v2.pdf
* [7] Lightning Network: https://lightning.network/lightning-network-paper-DRAFT-0.5.pdf
* [8] Tendermint: https://github.com/tendermint/tendermint/wiki
* [9] FLP Impossibility: https://groups.csail.mit.edu/tds/papers/Lynch/jacm85.pdf
* [10] Slasher: https://blog.ethereum.org/2014/01/15/slasher-a-punitive-proof-of-stake-algorithm/
* [11] PBFT: http://pmg.csail.mit.edu/papers/osdi99.pdf
* [12] BitShares: https://bitshares.org/technology/delegated-proof-of-stake-consensus/
* [13] Stellar: https://www.stellar.org/papers/stellar-consensus-protocol.pdf
* [14] Interledger: https://interledger.org/rfcs/0001-interledger-architecture/
* [15] Sidechains: https://blockstream.com/sidechains.pdf
* [16] Casper: https://blog.ethereum.org/2015/08/01/introducing-casper-friendly-ghost/
* [17] ABCI: https://github.com/tendermint/abci
* [18] Ethereum Sharding: https://github.com/ethereum/EIPs/issues/53
* [19] LibSwift: http://www.ds.ewi.tudelft.nl/fileadmin/pds/papers/PerformanceAnalysisOfLibswift.pdf
* [20] DLS: http://groups.csail.mit.edu/tds/papers/Lynch/jacm88.pdf
* [21] Thin Client Security: https://en.bitcoin.it/wiki/Thin_Client_Security
* [22] Ethereum 2.0 Mauve Paper: http://vitalik.ca/files/mauve_paper.html

#### Unsorted links |未分类链接

* https://www.docdroid.net/ec7xGzs/314477721-ethereum-platform-review-opportunities-and-challenges-for-private-and-consortium-blockchains.pdf.html
