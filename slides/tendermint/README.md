title: Tendermint
author:
  name: modood
  url: https://github.com/modood
output: index.html

--

# Tendermint

<br/><center>Warner Ma</center>

--

### What is Tendermint?

Tendermint is software for <u>**securely**</u> and <u>**consistently**</u>
replicating an application on many machines.

**securely**

```
works even if up to 1/3 of machines fail in arbitrary ways
```

**consistently**

```
every non-faulty machine sees the same transaction log and computes the same state
```

--

### Tendermint vs ZK, etcd, consul

**zookeeper, etcd, consul**

```
1.  non-BFT: Zookeeper Atomic Broadcast(Paxos), Raft
    can tolerate crash failures in up to 1/2 of the machines,
    but even a single Byzantine fault can destroy the system
2.  focused around providing basic services to distributed systems:
    kv store, dynamic configuration, service discovery,
    locking, leader-election, and so on.
```

**tendermint**

```
1.  BFT: Byzantine Fault Tolerant
    can only tolerate up to a 1/3 of failures,
    but those failures can include arbitrary behaviour
2.  focuses on arbitrary state machine replication
    It does not specify a particular application.
    so developers can build the application logic that's right for them:
    from key-value store to cryptocurrency to e-voting platform and beyond.
```

--

### Tendermint vs Bitcoin, Ethereum

**bitcoin, ethereum**

```
1.  cryptocurrencies
2.  Proof of Work
```

**tendermint**

```
A general purpose blockchain consensus engine.

It can be used as a plug-and-play replacement for the consensus engines
of other blockchain software
```

--

### Tendermint vs Bitcoin, Ethereum

**[cosmos/ethermint](https://github.com/cosmos/ethermint)**

```
Run Ethereum as a ABCI application using Tendermint consensus
```

**[hyperledger/burrow](https://github.com/hyperledger/burrow)**

```
An implementation of the Ethereum Virtual Machine and Ethereum transaction mechanics,
with additional features for a name-registry, permissions, and native contracts,
and an alternative blockchain API.

It uses Tendermint as its consensus engine, and provides a particular application state.
```

--

### ABCI

**ABCI: Application BlockChain Interface**

```
*   Decouple the consensus engine and P2P layers
*   Abstracting away the details of the particular blockchain application
    to an interface

Thus we have an interface, the Application BlockChain Interface (ABCI),
and its primary implementation, the Tendermint Socket Protocol (TSP, or Teaspoon).

Message Types: DeliverTx, CheckTx, Commit
```

--

### ABCI

**communicate with the application**

```
Tendermint Core (the "consensus engine") communicates with the application
via a socket protocol that satisfies the ABCI.

Tendermint acts as an ABCI client with respect to the application and
maintains 3 connections: mempool, consensus and query.

*   one for the validation of transactions when broadcasting in the mempool
*   one for the consensus engine to run block proposals
*   one for querying the application state

application: application process
tendermint core: consensus process
```

--

### ABCI

<img src="images/abci.png" style="width: 80%; height: auto; max-width: 100%; display: block; margin: 0 auto;"/>

--

### Listening

**default**

```
26656     the listening address of the tendermint peer
26657     the listening address of the RPC server
26658     the socket address of the abci application
```

--

### Peers

**about**

```
p2p Provides an abstraction around peer-to-peer communication.

peer exchange protocol (PeX)

*  Seed
*  Persistent Peer

*  Validator
*  Non-Validator
```

--

### RPC server

**about**

```
Tendermint supports the following RPC protocols:

*   URI over HTTP
*   JSONRPC over HTTP
*   JSONRPC over websockets

Broadcast API:

/broadcast_tx_async     will return right away
                        without waiting to hear if the transaction is even valid
/broadcast_tx_sync      will return with the result
                        of running the transaction through `CheckTx`
/broadcast_tx_commit    will wait until the transaction is committed in a block
                        or until some timeout is reached

Documentation: https://tendermint.com/rpc/
```

--

### RPC server

**subscribing to events via websocket**

```
const (
    EventCompleteProposal    = "CompleteProposal"
    EventLock                = "Lock"
    EventNewBlock            = "NewBlock"
    EventNewBlockHeader      = "NewBlockHeader"
    EventNewRound            = "NewRound"
    EventNewRoundStep        = "NewRoundStep"
    EventPolka               = "Polka"
    EventProposalHeartbeat   = "ProposalHeartbeat"
    EventRelock              = "Relock"
    EventTimeoutPropose      = "TimeoutPropose"
    EventTimeoutWait         = "TimeoutWait"
    EventTx                  = "Tx"
    EventUnlock              = "Unlock"
    EventValidatorSetUpdates = "ValidatorSetUpdates"
    EventVote                = "Vote"
)
```

--

### Mempool

**about**

```
mempool module handles all incoming transactions,
whenever they are coming from peers or the application.

A tx passed `CheckTx` (ie. was accepted into the mempool),


option: mempool.recheck = true

After every block, Tendermint rechecks every transaction left in the mempool
to see if transactions committed in that block affected the application state,
so some of the transactions left may become invalid.
```

--

### Consensus

**standard block creation cycle**

```
1.  enter new round
2.  enter `propose`
    signed proposal
    received complete proposal block
3.  enter `prevote`
    signed and pushed vote
    added to prevote
4.  enter `precommit`
    signed and pushed vote
    added to precommit
5.  enter `commit`
    executed block
    committed state
    recheck txs

blocks are produced regularly, even if there are no transactions.
```

--

### Consensus

<img src="images/consensus.png" style="width: 100%; height: auto; max-width: 100%; display: block; margin: 0 auto;"/>

--

### WAL

**WAL: Write Ahead Logs**

```
wal: for ensuring data integrity
replay: to replay blocks and messages on recovery from a crash

tendermint uses write ahead logs for:

*   the mempool (mempool.wal): writes all incoming txs before running `CheckTx`
*   the consensus (cs.wal): writes all consensus messages
```

--

### Fast Sync

~~run the real-time consensus gossip protocol~~

```
option: fast_sync = true

`fast-sync` mode is enabled by default

1.  just download blocks
2.  check the merkle tree of validators
3.  the node is considered `caught up` if it has at least one peer and
    it's height is at least as high as the max reported peer height
4.  once `caught up`, the daemon will switch out of fast sync and
    into the normal consensus mode
5.  if we're lagging sufficiently, we should go back to fast syncing
```

--

### Configuration

**[config.toml](https://tendermint.com/docs/tendermint-core/running-in-production.html#configuration-parameters)**

```
▾ config/                       +---------------------+
    config.toml --------------> |   base              |
    genesis.json                |   rpc               |
    node_key.json               |   p2p               |
    priv_validator.json         |   mempool           |
                                |   consensus         |
▾ data/                         |   tx_index          |
    blockstore.db               |   instrumentation   |
    evidence.db                 +---------------------+
    state.db
    tx_index.db
    cs.wal
    mempool.wal
```

--

### Tools

**tools for working with tendermint**

```
*   tools/tm-bench
*   tools/tm-monitor
*   scripts/wal2json
*   scripts/json2wal
```

--

### Miscellaneous

**non-determinism**

```
*   random number generators (without deterministic seeding)
*   race conditions on threads (or avoiding threads altogether)
*   the system clock
*   uninitialized memory (in unsafe programming languages like C or C++)
*   floating point arithmetic
*   language features that are random (e.g. map iteration in Go)
```

--

### Workflow

```

1.  create three connections (mempool, consensus and query) to the application
2.  tendermint core and the application perform a handshake
3.  start a few more things like the event switch, reactors,
    and perform UPNP discover in order to detect the IP address
4.  replay all the messages from the wal
5.  "started node" message signals that everything is ready for work
6.  next follows a standard block creation cycle, where we enter a new round,
    1) propose a block,
    2) receive more than 2/3 of prevotes,
    3) then precommits
    4) finally have a chance to commit a block
```

--

### References

*   [github - tendermint](https://github.com/tendermint/tendermint)
*   [tendermint core docs](https://tendermint.com/docs/)
*   [tendermint rpc reference](https://tendermint.com/rpc/)

--

# Thanks

<br/><center>2018.09.26</center>

