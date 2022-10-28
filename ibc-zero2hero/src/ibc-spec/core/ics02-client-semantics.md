# Client Semantics

| ics | title | stage | category | kind | requires | required-by | author | created | modified |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Client Semantics | draft | IBC/TAO | interface | 23, 24 | 3 | Juwoon Yun <joon@tendermint.com>, Christopher Goes <cwgoes@tendermint.com> | 2019-5-25 | 2019-7-13 |

## **Synopsis**

This standard specifies the properties that consensus algorithms of state machines implementing the inter-blockchain communication (IBC) protocol are required to satisfy. These properties are necessary for efficient and safe verification in the higher-level protocol abstractions. The algorithm utilised in IBC to verify the state updates of a remote state machine is referred to as a *validity predicate*. Pairing a validity predicate with a trusted state (i.e., a state that the verifier assumes to be correct), implements the functionality of a *light client* (often shortened to *client*) for a remote state machine on the host state machine. In addition to state update verification, every light client is able to detect consensus misbehaviours through a *misbehaviour predicate*.

本标准规定了实现区块链间通信（IBC）协议的状态机的共识算法需要满足的属性。这些属性对于高层协议抽象中的高效和安全验证是必要的。IBC中用来验证远程状态机的状态更新的算法被称为有效性谓词。将有效性谓词与可信状态（即验证者假定为正确的状态）配对，实现了主机状态机上远程状态机的轻客户端（通常简称为客户端）的功能。除了状态更新验证外，每个轻客户端都能通过一个错误行为谓词来检测共识的错误行为。

Beyond the properties described in this specification, IBC does not impose any requirements on the internal operation of the state machines and their consensus algorithms. A state machine may consist of a single process signing operations with a private key (the so-called "solo machine"), a quorum of processes signing in unison, many processes operating a Byzantine fault-tolerant consensus algorithm (e.g., Tendermint), or other configurations yet to be invented — from the perspective of IBC, a state machine is defined entirely by its light client validation and misbehaviour detection logic.

除了本规范中描述的属性外，IBC对状态机的内部操作及其共识算法没有任何要求。一个状态机可以由一个用私钥签署操作的单一进程（所谓的 "单机"）、一个一致签署的法定人数的进程、许多运行拜占庭容错共识算法（例如Tendermint）的进程，或者其他尚未发明的配置组成--从IBC的角度来看，一个状态机完全由其轻度客户验证和错误行为检测逻辑定义。

This standard also specifies how the light client's functionality is registered and how its data is stored and updated by the IBC protocol. The stored client instances can be introspected by a third party actor, such as a user inspecting the state of the state machine and deciding whether or not to send an IBC packet.

该标准还规定了轻型客户端的功能是如何注册的，其数据是如何被IBC协议存储和更新的。存储的客户端实例可以由第三方行为者进行反省，例如用户检查状态机的状态，并决定是否发送IBC数据包。

### **Motivation**

In the IBC protocol, an actor, which may be an end user, an off-chain process, or a module on a state machine, needs to be able to verify updates to the state of another state machine (i.e., the *remote state machine*). This entails accepting *only* the state updates that were agreed upon by the remote state machine's consensus algorithm. A light client of the remote state machine is the algorithm that enables the actor to verify state updates of that state machine. Note that light clients will generally not include validation of the entire state transition logic (as that would be equivalent to simply executing the other state machine), but may elect to validate parts of state transitions in particular cases. This standard formalises the light client model and requirements. As a result, the IBC protocol can easily be integrated with new state machines running new consensus algorithms, as long as the necessary light client algorithms fulfilling the listed requirements are provided.

在IBC协议中，一个行为者，可能是终端用户、链外进程或状态机上的一个模块，需要能够验证另一个状态机（即远程状态机）的状态更新。这就需要只接受由远程状态机的共识算法同意的状态更新。远程状态机的轻客户端是使行为人能够验证该状态机的状态更新的算法。请注意，轻客户端一般不包括对整个状态转换逻辑的验证（因为这相当于简单地执行其他状态机），但在特殊情况下可以选择验证部分状态转换。这个标准正式确定了轻客户端的模型和要求。因此，IBC协议可以很容易地与运行新共识算法的新状态机集成，只要提供满足所列要求的必要轻客户端算法即可。

The IBC protocol can be used to interact with probabilistic-finality consensus algorithms. In such cases, different validity predicates may be required by different applications. For probabilistic-finality consensus, a validity predicate is defined by a finality threshold (e.g., the threshold defines how many block needs to be on top of a block in order to consider it finalized). As a result, clients could act as *thresholding views* of other clients: One *write-only* client could be used to store state updates (without the ability to verify them), while many *read-only* clients with different finality thresholds (confirmation depths after which state updates are considered final) are used to verify state updates.

IBC协议可用于与概率-极限共识算法进行交互。在这种情况下，不同的应用可能需要不同的有效性谓词。对于概率终结性共识，有效性谓词由终结性阈值定义（例如，阈值定义了一个区块上面需要有多少个区块才能认为它已经终结）。因此，客户端可以作为其他客户端的阈值视图。一个只写的客户端可以用来存储状态更新（没有验证它们的能力），而许多具有不同最终性阈值（确认深度之后，状态更新被认为是最终的）的只读客户端被用来验证状态更新。

The client protocol should also support third-party introduction. For example, if `A`, `B`, and `C` are three state machines, with Alice a module on `A`, Bob a module on `B`, and Carol a module on `C`, such that Alice knows both Bob and Carol, but Bob knowns only Alice and not Carol, then Alice can utilise an existing channel to Bob to communicate the canonically-serialisable validity predicate for Carol. Bob can then use this validity predicate to open a connection and channel so that Bob and Carol can talk directly. If necessary, Alice may also communicate to Carol the validity predicate for Bob, prior to Bob's connection attempt, so that Carol knows to accept the incoming request.

客户端协议还应该支持第三方引入。例如，如果A、B、C是三个状态机，Alice是A上的一个模块，Bob是B上的一个模块，Carol是C上的一个模块，这样Alice既知道Bob也知道Carol，但Bob只知道Alice而不知道Carol，那么Alice可以利用现有的通道向Bob传达Carol的可序列化的有效性谓语。然后Bob可以使用这个有效性谓词来打开一个连接和通道，这样Bob和Carol就可以直接对话。如果有必要，Alice也可以在Bob的连接尝试之前向Carol传达Bob的有效性谓词，这样Carol就知道要接受传入的请求。

Client interfaces should also be constructed so that custom validation logic can be provided safely to define a custom client at runtime, as long as the underlying state machine can provide an appropriate gas metering mechanism to charge for compute and storage. On a host state machine which supports WASM execution, for example, the validity predicate and misbehaviour predicate could be provided as executable WASM functions when the client instance is created.

客户端接口也应该被构造成可以安全地提供自定义验证逻辑，以便在运行时定义一个自定义客户端，只要底层状态机可以提供适当的气体计量机制来收取计算和存储费用。例如，在支持WASM执行的主机状态机上，有效性谓词和错误行为谓词可以在创建客户端实例时作为可执行的WASM函数提供。

### **Definitions**

- `get`, `set`, `Path`, and `Identifier` are as defined in [ICS 24](https://github.com/cosmos/ibc/blob/main/spec/core/ics-024-host-requirements).
- `Consensus` is a state update generating algorithm. It takes the previous state of a state machine together with a set of messages (i.e., state machine transactions) and generates a valid state update of the state machine. Every state machine MUST have a `Consensus` that generates a unique, ordered list of state updates starting from a genesis state.
    
    This specification expects that the state updates generated by `Consensus` satisfy the following properties:
    
    “共识 ”是一种状态更新的生成算法。它将一个状态机的前一个状态与一组消息（即状态机事务）结合起来，生成一个有效的状态机更新。每个状态机都必须有一个 "共识"，从创世状态开始生成一个唯一的、有序的状态更新列表。
    
    本规范希望由`Consensus`生成的状态更新满足以下属性。
    
    - Every state update MUST NOT have more than one direct successor in the list of state updates. In other words, the state machine MUST guarantee *finality* and *safety*.
    - Every state update MUST eventually have a successor in the list of state updates. In other words, the state machine MUST guarantee *liveness*.
    - Every state update MUST be valid (i.e., valid state transitions). In other words, `Consensus` MUST be *honest*, e.g., in the case `Consensus` is a Byzantine fault-tolerant consensus algorithm, such as Tendermint, less than a third of block producers MAY be Byzantine.
    
    Unless the state machine satisfies all of the above properties, the IBC protocol may not work as intended, e.g., users' assets might be stolen. Note that specific client types may require additional properties.
    
- `Height` specifies the order of the state updates of a state machine, e.g., a sequence number. This entails that each state update is mapped to a `Height`.
- `CommitmentRoot` is as defined in [ICS 23](https://github.com/cosmos/ibc/blob/main/spec/core/ics-023-vector-commitments). It provides an efficient way for higher-level protocol abstractions to verify whether a particular state transition has occurred on the remote state machine, i.e., it enables proofs of inclusion or non-inclusion of particular values at particular paths in the state of the remote state machine at particular `Height`s.
- `ClientMessage` is an arbitrary message defined by the client type that relayers can submit in order to update the client. The ClientMessage may be intended as a regular update which may add new consensus state for proof verification, or it may contain misbehaviour which should freeze the client.
- `ValidityPredicate` is a function that validates a ClientMessage sent by a relayer in order to update the client. Using the `ValidityPredicate` SHOULD be more computationally efficient than executing `Consensus`.
- `ConsensusState` is the *trusted view* of the state of a state machine at a particular `Height`. It MUST contain sufficient information to enable the `ValidityPredicate` to validate state updates, which can then be used to generate new `ConsensusState`s. It MUST be serialisable in a canonical fashion so that remote parties, such as remote state machines, can check whether a particular `ConsensusState` was stored by a particular state machine. It MUST be introspectable by the state machine whose view it represents, i.e., a state machine can look up its own `ConsensusState`s at past `Height`s.
- `ClientState` is the state of a client. It MUST expose an interface to higher-level protocol abstractions, e.g., functions to verify proofs of the existence of particular values at particular paths at particular `Height`s.
- `MisbehaviourPredicate` is a that checks whether the rules of `Consensus` were broken, in which case the client MUST be *frozen*, i.e., no subsequent `ConsensusState`s can be generated.
- `Misbehaviour` is the proof needed by the `MisbehaviourPredicate` to determine whether a violation of the consensus protocol occurred. For example, in the case the state machine is a blockchain, a `Misbehaviour` might consist of two signed block headers with different `CommitmentRoot`s, but the same `Height`.

### **Desired Properties**

Light clients MUST provide state verification functions that provide a secure way to verify the state of the remote state machines using the existing `ConsensusState`s. These state verification functions enable higher-level protocol abstractions to verify sub-components of the state of the remote state machines.

`ValidityPredicate`s MUST reflect the behaviour of the remote state machine and its `Consensus`, i.e., `ValidityPredicate`s accept *only* state updates that contain state updates generated by the `Consensus` of the remote state machine.

In case of misbehavior, the behaviour of the `ValidityPredicate` might differ from the behaviour of the remote state machine and its `Consensus` (since clients do not execute the `Consensus` of the remote state machine). In this case, a `Misbehaviour` SHOULD be submitted to the host state machine, which would result in the client being frozen and higher-level intervention being necessary.

## **Technical Specification**

This specification outlines what each *client type* must define. A client type is a set of definitions of the data structures, initialisation logic, validity predicate, and misbehaviour predicate required to operate a light client. State machines implementing the IBC protocol can support any number of client types, and each client type can be instantiated with different initial consensus states in order to track different consensus instances. In order to establish a connection between two state machines (see [ICS 3](https://github.com/cosmos/ibc/blob/main/spec/core/ics-003-connection-semantics)), the state machines must each support the client type corresponding to the other state machine's consensus algorithm.

Specific client types shall be defined in later versions of this specification and a canonical list shall exist in this repository. State machines implementing the IBC protocol are expected to respect these client types, although they may elect to support only a subset.

### **Data Structures**

### **Height**

`Height` is an opaque data structure defined by a client type. It must form a partially ordered set & provide operations for comparison.

```typescript
type Height

enum Ord {
  LT
  EQ
  GT
}

type compare = (h1: Height, h2: Height) => Ord`
```

A height is either `LT` (less than), `EQ` (equal to), or `GT` (greater than) another height.

`>=`, `>`, `===`, `<`, `<=` are defined through the rest of this specification as aliases to `compare`.

There must also be a zero-element for a height type, referred to as `0`, which is less than all non-zero heights.

### **ConsensusState**

`ConsensusState` is an opaque data structure defined by a client type, used by the validity predicate to verify new commits & state roots. Likely the structure will contain the last commit produced by the consensus process, including signatures and validator set metadata.

`ConsensusState` MUST be generated from an instance of `Consensus`, which assigns unique heights for each `ConsensusState` (such that each height has exactly one associated consensus state). Two `ConsensusState`s on the same chain SHOULD NOT have the same height if they do not have equal commitment roots. Such an event is called an "equivocation" and MUST be classified as misbehaviour. Should one occur, a proof should be generated and submitted so that the client can be frozen and previous state roots invalidated as necessary.

The `ConsensusState` of a chain MUST have a canonical serialisation, so that other chains can check that a stored consensus state is equal to another (see [ICS 24](https://github.com/cosmos/ibc/blob/main/spec/core/ics-024-host-requirements) for the keyspace table).

```rust
type ConsensusState = bytes
```

The `ConsensusState` MUST be stored under a particular key, defined below, so that other chains can verify that a particular consensus state has been stored.

The `ConsensusState` MUST define a `getTimestamp()` method which returns the timestamp associated with that consensus state:

```rust
type getTimestamp = ConsensusState => uint64
```

### **ClientState**

`ClientState` is an opaque data structure defined by a client type. It may keep arbitrary internal state to track verified roots and past misbehaviours.

Light clients are representation-opaque — different consensus algorithms can define different light client update algorithms — but they must expose this common set of query functions to the IBC handler.

```typescript
type ClientState = bytes
```

Client types MUST define a method to initialise a client state with a provided consensus state, writing to internal state as appropriate.

```typescript
type initialise = (consensusState: ConsensusState) => ClientState
```

Client types MUST define a method to fetch the current height (height of the most recent validated state update).

```typescript
type latestClientHeight = (
  clientState: ClientState
) => Height
```

Client types MUST define a method on the client state to fetch the timestamp at a given height

```typescript
type getTimestampAtHeight = (
  clientState: ClientState,
  height: Height
) => uint64
```

### **ClientMessage**

A `ClientMessage` is an opaque data structure defined by a client type which provides information to update the client. `ClientMessages` can be submitted to an associated client to add new `ConsensusState(s)` and/or update the `ClientState`. They likely contain a height, a proof, a commitment root, and possibly updates to the validity predicate.

```typescript
type ClientMessage = bytes
```

### **Validity predicate**

A validity predicate is an opaque function defined by a client type to verify `ClientMessage`s depending on the current `ConsensusState`. Using the validity predicate SHOULD be far more computationally efficient than replaying the full consensus algorithm for the given parent `ClientMessage` and the list of network messages.

The validity predicate is defined as:

```typescript
type VerifyClientMessage = (ClientMessage) => Void
```

`VerifyClientMessage` MUST throw an exception if the provided ClientMessage was not valid.

### **Misbehaviour predicate**

A misbehaviour predicate is an opaque function defined by a client type, used to check if a ClientMessage constitutes a violation of the consensus protocol. For example, if the state machine is a blockchain, this might be two signed headers with different state roots but the same height, a signed header containing invalid state transitions, or other proof of malfeasance as defined by the consensus algorithm.

The misbehaviour predicate is defined as

```typescript
type checkForMisbehaviour = (ClientMessage) => bool
```

`checkForMisbehaviour` MUST throw an exception if the provided proof of misbehaviour was not valid.

### **UpdateState**

UpdateState will update the client given a verified `ClientMessage`. Note that this function is intended for **non-misbehaviour** `ClientMessages`.

```typescript
type UpdateState = (ClientMessage) => Void
```

`verifyClientMessage` must be called before this function, and `checkForMisbehaviour` must return false before this function is called.

The client MUST also mutate internal state to store now-finalised consensus roots and update any necessary signature authority tracking (e.g. changes to the validator set) for future calls to the validity predicate.

Clients MAY have time-sensitive validity predicates, such that if no ClientMessage is provided for a period of time (e.g. an unbonding period of three weeks) it will no longer be possible to update the client, i.e., the client is being frozen. In this case, a permissioned entity such as a chain governance system or trusted multi-signature MAY be allowed to intervene to unfreeze a frozen client & provide a new correct ClientMessage.

### **UpdateStateOnMisbehaviour**

UpdateStateOnMisbehaviour will update the client upon receiving a verified `ClientMessage` that is valid misbehaviour.

```typescript
type UpdateStateOnMisbehaviour = (ClientMessage) => Void
```

`verifyClientMessage` must be called before this function, and `checkForMisbehaviour` must return `true` before this function is called.

The client MUST also mutate internal state to mark appropriate heights which were previously considered valid as invalid, according to the nature of the misbehaviour.

Once misbehaviour is detected, clients SHOULD be frozen so that no future updates can be submitted. A permissioned entity such as a chain governance system or trusted multi-signature MAY be allowed to intervene to unfreeze a frozen client & provide a new correct ClientMessage which updates the client to a valid state.

### **CommitmentProof**

`CommitmentProof` is an opaque data structure defined by a client type in accordance with [ICS 23](https://github.com/cosmos/ibc/blob/main/spec/core/ics-023-vector-commitments). It is utilised to verify presence or absence of a particular key/value pair in state at a particular finalised height (necessarily associated with a particular commitment root).

### **State verification**

Client types must define functions to authenticate internal state of the state machine which the client tracks. Internal implementation details may differ (for example, a loopback client could simply read directly from the state and require no proofs).

- The `delayPeriodTime` is passed to the verification functions for packet-related proofs in order to allow packets to specify a period of time which must pass after a consensus state is added before it can be used for packet-related verification.
- The `delayPeriodBlocks` is passed to the verification functions for packet-related proofs in order to allow packets to specify a period of blocks which must pass after a consensus state is added before it can be used for packet-related verification.

`verifyMembership` is a generic proof verification method which verifies a proof of the existence of a value at a given `CommitmentPath` at the specified height. The caller is expected to construct the full `CommitmentPath` from a `CommitmentPrefix` and a standardized path (as defined in [ICS 24](https://github.com/cosmos/ibc/blob/main/spec/core/ics-024-host-requirements/README.md#path-space)). If the caller desires a particular delay period to be enforced, then it can pass in a non-zero `delayPeriodTime` or `delayPeriodBlocks`. If a delay period is not necessary, the caller must pass in 0 for `delayPeriodTime` and `delayPeriodBlocks`, and the client will not enforce any delay period for verification.

```typescript
type verifyMembership = (
  clientState: ClientState,
  height: Height,
  delayPeriodTime: uint64,
  delayPeriodBlocks: uint64,
  proof: CommitmentProof,
  path: CommitmentPath,
  value: bytes
) => boolean
```

`verifyNonMembership` is a generic proof verification method which verifies a proof of absence of a given `CommitmentPath` at the specified height. The caller is expected to construct the full `CommitmentPath` from a `CommitmentPrefix` and a standardized path (as defined in [ICS 24](https://github.com/cosmos/ibc/blob/main/spec/core/ics-024-host-requirements/README.md#path-space)). If the caller desires a particular delay period to be enforced, then it can pass in a non-zero `delayPeriodTime` or `delayPeriodBlocks`. If a delay period is not necessary, the caller must pass in 0 for `delayPeriodTime` and `delayPeriodBlocks`, and the client will not enforce any delay period for verification.

Since the verification method is designed to give complete control to client implementations, clients can support chains that do not provide absence proofs by verifying the existence of a non-empty sentinel `ABSENCE` value. Thus in these special cases, the proof provided will be an ICS-23 Existence proof, and the client will verify that the `ABSENCE` value is stored under the given path for the given height.

```typescript
type verifyNonMembership = (
  clientState: ClientState,
  height: Height,
  delayPeriodTime: uint64,
  delayPeriodBlocks: uint64,
  proof: CommitmentProof,
  path: CommitmentPath
) => boolean
```

### **Query interface**

### **Chain queries**

These query endpoints are assumed to be exposed over HTTP or an equivalent RPC API by nodes of the chain associated with a particular client.

`queryUpdate` MUST be defined by the chain which is validated by a particular client, and should allow for retrieval of clientMessage for a given height. This endpoint is assumed to be untrusted.

```typescript
type queryUpdate = (height: Height) => ClientMessage
```

`queryChainConsensusState` MAY be defined by the chain which is validated by a particular client, to allow for the retrieval of the current consensus state which can be used to construct a new client. When used in this fashion, the returned `ConsensusState` MUST be manually confirmed by the querying entity, since it is subjective. This endpoint is assumed to be untrusted. The precise nature of the `ConsensusState` may vary per client type.

```typescript
type queryChainConsensusState = (height: Height) => ConsensusState
```

Note that retrieval of past consensus states by height (as opposed to just the current consensus state) is convenient but not required.

`queryChainConsensusState` MAY also return other data necessary to create clients, such as the "unbonding period" for certain proof-of-stake security models. This data MUST also be verified by the querying entity.

### **On-chain state queries**

This specification defines a single function to query the state of a client by-identifier.

```typescript
function queryClientState(identifier: Identifier): ClientState {
  return provableStore.get(clientStatePath(identifier))
}
```

The `ClientState` type SHOULD expose its latest verified height (from which the consensus state can then be retrieved using `queryConsensusState` if desired).

```typescript
type latestHeight = (state: ClientState) => Height
```

Client types SHOULD define the following standardised query functions in order to allow relayers & other off-chain entities to interface with on-chain state in a standard API.

`queryConsensusState` allows stored consensus states to be retrieved by height.

```typescript
type queryConsensusState = (
  identifier: Identifier,
  height: Height,
) => ConsensusState
```

### **Proof construction**

Each client type SHOULD define functions to allow relayers to construct the proofs required by the client's state verification algorithms. These may take different forms depending on the client type. For example, Tendermint client proofs may be returned along with key-value data from store queries, and solo client proofs may need to be constructed interactively on the solo state machine in question (since the user will need to sign the message). These functions may constitute external queries over RPC to a full node as well as local computation or verification.

```typescript
type queryAndProveClientConsensusState = (
  clientIdentifier: Identifier,
  height: Height,
  prefix: CommitmentPrefix,
  consensusStateHeight: Height
) => ConsensusState, Proof

type queryAndProveConnectionState = (
  connectionIdentifier: Identifier,
  height: Height,
  prefix: CommitmentPrefix
) => ConnectionEnd, Proof

type queryAndProveChannelState = (
  portIdentifier: Identifier,
  channelIdentifier: Identifier,
  height: Height,
  prefix: CommitmentPrefix
) => ChannelEnd, Proof

type queryAndProvePacketData = (
  portIdentifier: Identifier,
  channelIdentifier: Identifier,
  height: Height,
  prefix: CommitmentPrefix,
  sequence: uint64
) => []byte, Proof

type queryAndProvePacketAcknowledgement = (
  portIdentifier: Identifier,
  channelIdentifier: Identifier,
  height: Height,
  prefix: CommitmentPrefix,
  sequence: uint64
) => []byte, Proof

type queryAndProvePacketAcknowledgementAbsence = (
  portIdentifier: Identifier,
  channelIdentifier: Identifier,
  height: Height,
  prefix: CommitmentPrefix,
  sequence: uint64
) => Proof

type queryAndProveNextSequenceRecv = (
  portIdentifier: Identifier,
  channelIdentifier: Identifier,
  height: Height,
  prefix: CommitmentPrefix
) => uint64, Proof
```

### **Implementation strategies**

### **Loopback**

A loopback client of a local state machine merely reads from the local state, to which it must have access.

### **Simple signatures**

A client of a solo state machine with a known public key checks signatures on messages sent by that local state machine, which are provided as the `Proof` parameter. The `height` parameter can be used as a replay protection nonce.

Multi-signature or threshold signature schemes can also be used in such a fashion.

### **Proxy clients**

Proxy clients verify another (proxy) state machine's verification of the target state machine, by including in the proof first a proof of the client state on the proxy state machine, and then a secondary proof of the sub-state of the target state machine with respect to the client state on the proxy state machine. This allows the proxy client to avoid storing and tracking the consensus state of the target state machine itself, at the cost of adding security assumptions of proxy state machine correctness.

### **Merklized state trees**

For clients of state machines with Merklized state trees, these functions can be implemented by calling the [ICS-23](https://github.com/cosmos/ibc/blob/main/spec/core/ics-023-vector-commitments/README.md) `verifyMembership` or `verifyNonMembership` methods, using a verified Merkle root stored in the `ClientState`, to verify presence or absence of particular key/value pairs in state at particular heights in accordance with [ICS 23](https://github.com/cosmos/ibc/blob/main/spec/core/ics-023-vector-commitments).

```rust
type verifyMembership = (ClientState, Height, CommitmentProof, Path, Value) => boolean

type verifyNonMembership = (ClientState, Height, CommitmentProof, Path) => boolean
```

### **Sub-protocols**

IBC handlers MUST implement the functions defined below.

### **Identifier validation**

Clients are stored under a unique `Identifier` prefix. This ICS does not require that client identifiers be generated in a particular manner, only that they be unique. However, it is possible to restrict the space of `Identifier`s if required. The validation function `validateClientIdentifier` MAY be provided.

```rust
type validateClientIdentifier = (id: Identifier) => boolean
```

If not provided, the default `validateClientIdentifier` will always return `true`.

### **Utilising past roots**

To avoid race conditions between client updates (which change the state root) and proof-carrying transactions in handshakes or packet receipt, many IBC handler functions allow the caller to specify a particular past root to reference, which is looked up by height. IBC handler functions which do this must ensure that they also perform any requisite checks on the height passed in by the caller to ensure logical correctness.

为了避免客户端更新（改变状态根）和握手或数据包接收中的证明交易之间的竞赛条件，许多IBC处理函数允许调用者指定一个特定的过去根来引用，该根被按高度查询。这样做的IBC处理函数必须确保它们也对调用者传入的高度进行任何必要的检查，以确保逻辑上的正确性。

### **Create**

Calling `createClient` with the specified identifier & initial consensus state creates a new client.

用指定的标识符和初始共识状态调用`createClient`，可以创建一个新的客户端。

```rust
function createClient(
  id: Identifier,
  clientType: ClientType,
  consensusState: ConsensusState
) {
    abortTransactionUnless(validateClientIdentifier(id))
    abortTransactionUnless(provableStore.get(clientStatePath(id)) === null)
    abortSystemUnless(provableStore.get(clientTypePath(id)) === null)
    clientType.initialise(consensusState)
    provableStore.set(clientTypePath(id), clientType)
}
```

### **Query**

Client consensus state and client internal state can be queried by identifier, but the specific paths which must be queried are defined by each client type.

客户端共识状态和客户端内部状态可以通过标识符进行查询，但必须查询的具体路径由每个客户端类型定义。

### **Update**

Updating a client is done by submitting a new `ClientMessage`. The `Identifier` is used to point to the stored `ClientState` that the logic will update. When a new `ClientMessage` is verified with the stored `ClientState`'s validity predicate and `ConsensusState`, the client MUST update its internal state accordingly, possibly finalising commitment roots and updating the signature authority logic in the stored consensus state.

更新一个客户是通过提交一个新的 `ClientMessage` 来完成的。Identifier "用于指向存储的 "ClientState"，该逻辑将被更新。当一个新的 `ClientMessage` 与存储的 `ClientState` 的有效性谓词和 `ConsensusState` 进行验证时，客户端必须相应地更新其内部状态，可能会最终确定承诺根基并更新存储的共识状态中的签名授权逻辑。

If a client can no longer be updated (if, for example, the trusting period has passed), it will no longer be possible to send any packets over connections & channels associated with that client, or timeout any packets in-flight (since the height & timestamp on the destination chain can no longer be verified). Manual intervention must take place to reset the client state or migrate the connections & channels to another client. This cannot safely be done completely automatically, but chains implementing IBC could elect to allow governance mechanisms to perform these actions (perhaps even per-client/connection/channel in a multi-sig or contract).

如果一个客户端不能再被更新（例如，如果信任期已过），它将不再可能通过与该客户端相关的连接和通道发送任何数据包，或在飞行中的任何数据包超时（因为目的地链上的高度和时间戳不能再被验证）。人工干预必须发生，以重置客户端的状态或将连接和通道迁移到另一个客户端。这不能安全地完全自动完成，但实施IBC的链可以选择允许治理机制来执行这些行动（甚至可能是在多签名或合同中的每个客户/连接/通道）。

```rust
function updateClient(
  id: Identifier,
  clientMessage: ClientMessage
) {
    // get clientState from store with id
    clientState = provableStore.get(clientStatePath(id))
    abortTransactionUnless(clientState !== null)

    clientState.VerifyClientMessage(clientMessage)
    
    foundMisbehaviour := clientState.CheckForMisbehaviour(clientMessage)
    if foundMisbehaviour {
        clientState.UpdateStateOnMisbehaviour(clientMessage)
        // emit misbehaviour event
    }
    else {    
        clientState.UpdateState(clientMessage) // expects no-op on duplicate clientMessage
        // emit update event
    }
}
```

### **Misbehaviour**

A relayer may alert the client to the misbehaviour directly, possibly invalidating previously valid state roots & preventing future updates.

中继器可能会直接提醒客户端的错误行为，可能会使以前有效的状态根基无效，并阻止未来的更新。

```rust
function submitMisbehaviourToClient(
  id: Identifier,
  clientMessage: ClientMessage
) {
    clientState = provableStore.get(clientStatePath(id))
    abortTransactionUnless(clientState !== null)
    // authenticate client message
    clientState.verifyClientMessage(clientMessage)
    // check that client message is valid instance of misbehaviour
    abortTransactionUnless(clientState.checkForMisbehaviour(clientMessage))
    // update state based on misbehaviour
    clientState.UpdateStateOnMisbehaviour(misbehaviour)
}
```

### **Properties & Invariants**

- Client identifiers are immutable & first-come-first-serve. Clients cannot be deleted (allowing deletion would potentially allow future replay of past packets if identifiers were re-used).

客户端标识符是不可改变的，而且是先到先得的。客户端不能被删除（如果标识符被重新使用，允许删除有可能允许未来重放过去的数据包）。

## **Backwards Compatibility**

Not applicable.

## **Forwards Compatibility**

New client types can be added by IBC implementations at-will as long as they conform to this interface.

## **Example Implementation**

Please see the ibc-go implementations of light clients for examples of how to implement your own: [https://github.com/cosmos/ibc-go/blob/main/modules/light-clients](https://github.com/cosmos/ibc-go/blob/main/modules/light-clients)

## **Other Implementations**

Coming soon.

## **History**

Mar 5, 2019 - Initial draft finished and submitted as a PR

May 29, 2019 - Various revisions, notably multiple commitment-roots

Aug 15, 2019 - Major rework for clarity around client interface

Jan 13, 2020 - Revisions for client type separation & path alterations

Jan 26, 2020 - Addition of query interface

Jul 27, 2022 - Addition of `verifyClientState` function, and move `ClientState` to the `provableStore`

August 4, 2022 - Changes to ClientState interface and associated handler to align with changes in 02-client-refactor ADR: [cosmos/ibc-go#1871](https://github.com/cosmos/ibc-go/pull/1871)

## **Copyright**

All content herein is licensed under [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0).