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

**Zookeeper, etcd, Consul**

```
1.  non-BFT: Zookeeper Atomic Broadcast(Paxos), Raft
    can tolerate crash failures in up to 1/2 of the machines,
    but even a single Byzantine fault can destroy the system
2.  focused around providing basic services to distributed systems:
    kv store, dynamic configuration, service discovery,
    locking, leader-election, and so on.
```

**Tendermint**

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

**Bitcoin, Ethereum**

```
1.  cryptocurrencies
2.  Proof of Work
```

**Tendermint**

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

**Communicate with the application**

```
Tendermint Core (the "consensus engine") communicates with the application
via a socket protocol that satisfies the ABCI.

Tendermint acts as an ABCI client with respect to the application and
maintains 3 connections: mempool, consensus and query.

*   one for the validation of transactions when broadcasting in the mempool
*   one for the consensus engine to run block proposals
*   one for querying the application state
```

--

### ABCI

<img src="images/abci.png" style="width: 80%; height: auto; max-width: 100%; display: block; margin: 0 auto;"/>
