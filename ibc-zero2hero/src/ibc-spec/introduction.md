# Ibc From Zero to Hero

包含ibc协议的spec的翻译，中英文对照。还有ibc relayer的源码阅读，代码拆解分析。

## **Synopsis**

This repository is the canonical location for development and documentation of the inter-blockchain communication protocol (IBC).

It shall be used to consolidate design rationale, protocol semantics, and encoding descriptions for IBC, including both the core transport, authentication, & ordering layer (IBC/TAO) and the application layers describing packet encoding & processing semantics (IBC/APP).

Contributions are welcome. See [CONTRIBUTING.md](https://github.com/cosmos/ibc/blob/master/meta/CONTRIBUTING.md) for contribution guidelines.

See [ROADMAP.md](https://github.com/cosmos/ibc/blob/master/meta/ROADMAP.md) for a public up-to-date version of our roadmap.

## **Interchain Standards**

All standards at or past the "Draft" stage are listed here in order of their ICS numbers, sorted by category.

### **Meta**

- ics01 specification Standard

### **Core**

- ics02 Clients Semantics
- ics03 Connection Semantics
- ics04 Channel & Packet Semantics
- ics05 Port Allocation
- ics23 Vector Commitments
- ics24 Host Requirements
- ics25 Handle Interface
- ics26 Routing Module 
  
### **Client**

- ics06 Solo Machine Client
- ics07 Tendermint Client
- ics09 Loopback Client 
- ics10 grandpa Client 
- ics08 wasm Client


### **relayer**

- ics18 Relayer Algorithms

### App

- ics20 Fungible token Transfer
- ics27 Interchain Accounts
- ics29 Free Payment
- ics30 Ibc application middleware
- ics31 Cross-chain Queries
- ics721 Non-Fungible Token transfer