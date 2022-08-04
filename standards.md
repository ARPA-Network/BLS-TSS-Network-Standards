# ARPA BLS Threshold Signature Network Standards

Core architecture, components, protocol, and token-economics

## Abstract

ARPA BLS Threshold Signature Network (ARPA) is a decentralized (permission-less) network consisting of multiple, dynamic groups of nodes that can securely generate BLS threshold signatures. ARPA can serve as the infrastructure of many Blockchain applications such as verifiable random beacons, secure (keyless) wallets, cross-chain bridges, decentralized custody, etc.

## 1. Introduction

### 1.1 Design Goals

| Goals | Description |
| --- | --- |
| Decentralization | Eliminate the requirement for centralized authorities or single point of failure in nodes/groups management within the network. |
| Control | Give entities, both human and non-human, the power to directly control the generation and usage of the BLS signature without the need to rely on external authorities. |
| Security | Enable sufficient security for requesting parties to depend on the generated BLS signature for their required level of assurance |
| Proof-based | Enable BLS signatures to provide cryptographic proof when used in their applications |
| Discoverability | Make it possible for entities to discover the state of nodes and the network, to learn more about or interact with the nodes and the network |
| Interoperability | Use interoperable standards so the ARPA infrastructure can make use of existing programming language, tools, and software libraries designed for interoperability |
| Portability | Be platform- and network-independent and enable entities to run a node on any operating system with internet connection |
| Simplicity | Favor a modular design where each module is simple to make the technology easier to understand, implement, and deploy |
| Extensibility | Where possible, enable extensibility provided it does not greatly hinder interoperability, portability, or simplicity. |

### 1.2 Architectural Overview

ARPA network relies on a smart-contract-capable Blockchain to manage its dynamic global states. On a high level, these dynamic global states include ___node___ information, ___group___ information, and all BLS signature tasks.

This is achieved by implementing and deploying a single "___controller___" smart contract on a certain Blockchain, each unregistered ___node___ directly interacts with the "___controller___" to register themselves into the ARPA network and then discover the information needed to communicate with other registered ___nodes___.

For higher throughput and better service availability, the ___nodes___ in the ARPA network are split into multiple ___groups___ to handle BLS signature tasks in parallel. The "___controller___" is also responsible for initiating the ___grouping___ of the ___nodes___ and storing the ___group___ information.

The ___grouping___ process is essentially a Distributed Key Generation process (DKG), a "___coordinator___" smart contract is implemented and deployed ad-hoc to coordinate a subset of ___nodes___ through the different phases of the DKG process, then submits the information of the new "___group___" to the "___controller___".

To provide BLS services for all smart-contract-capable Blockchains, an "___adapter___" smart contract is implemented and deployed to each of these Blockchains, the "___adapter___" acts as the APIs for other DApp clients to request BLS signatures.

The DApp client requests the BLS signature by calling the "___adapter___" APIs, the "___adapter___" assigns the BLS signature task to a specific ___group___, each ___grouped node___ monitors the BLS signature task event emitted by the "___adapter___" and starts a BLS signature task if it belongs to the assigned ___group___ then submits the signature to the "___adapter___" upon completion. The "___adapter___" then returns the results to the caller DApps. There is also a backup mechanism in place if the assigned ___group___ fails to fulfill the request within a reasonable amount of time.

<!--
Since the "___adapter___" need the global states (available ___nodes___ and ___groups___) to assign BLS tasks, but first only the "___controller___" has these states, second the "___adapter___" and the "___controller___" are not necessarily on the same Blockchain, thus the registered ___nodes___ need to relay the global states from the "___controller___" to the "___adapter___" so that these states are always in-sync among the network, the "___controller___", and the "___adapter___".

The "___adapter___" are also the source of truth to keep track of reward of the BLS tasks, the ARPA network also relays this information to the "___controller___" later used for ___nodes___ to claim their rewards.
-->

          +------------------------------------+                                           
          |    Blockchain (Smart Contract)     |                                           
          +------------+-------------+---------+                                           
          | Controller | Coordinator | Adapter |                                           
          +------------+-------------+---------+                                           
                                                                                           
          +------+ +------+ +------+ +------+ -+                                           
          | Node | | Node | | Node | | Node |  |                                           
          +------+ +------+ +------+ +------+  |                                           
          | Node | | Node | | Node | | Node |  +-- Group                                   
          +------+ +------+ +------+ +------+  |                                           
          | Node | | Node | | Node | | Node |  |                                           
          +------+ +------+ +------+ +------+ -+                                           

___Figure A:___ _A high-level architectural view;_
_Please note that_ ___each group___ _can have_ ___more than 3 nodes.___

## 2. Core

This section describes and defines the fundamental attributes of the ARPA network. It provides crucial information on what the network achieves and the general principles of how it works.

### 2.1 Network Responsibilities

- __To fulfill__ BLS threshold signature and randomness requests
- __To manage__ ___nodes___ for joining and exiting the network
- __To group__ ___nodes___ dynamically via the DKG process
- __To maintain__ the network global states
- __To sustain__ the token economics and handle rewards for the participating ___nodes___
- __To track__ the historical BLS signature tasks
- __To verify__ any BLS signature task completed by the network

### 2.2 Network Composition

The ARPA network is formed of many ___groups___ of ___nodes___, the ___groups___ are formed dynamically by the DKG process.

___Nodes___ --> ___Groups___ --> ARPA Network

The major components of the ARPA network are:

- ___Node___
- ___Controller___
- ___Coordinator___
- ___Adapter___

Note that the ___group___ is not considered a ARPA network component because it is an ephemeral entity that just consists of many ___nodes___.

#### 2.2.1 Node Composition & Responsibilities

- Blockchain Event Listeners
  - Listen to new block events
  - Listen to new DKG task events
  - Listen to new randomness task
- Context Cache
  - Maintain the Blockchain information
  - Maintain the DKG private key
  - Maintain the DKG public key
  - Maintain the networking information
  - Track the unhandled events
- DKG Module
  - Run DKG process
  - Detect DKG phase
- BLS Module
  - Sign partial signature
  - Verify partial signature
  - Aggregate partial signatures
  - Verify signature
- Committer Module
  - Commit the BLS signature result

#### 2.2.2 Controller Responsibilities

- Manage network constraints
  - Node token staking amount (50,000)
  - Node disqualification penalty amount (1,000)
  - TODO: COORDINATOR_STATE_TRIGGER_REWARD
  - Default minimum signature threshold (3)
  - Default number of committers (3)
  - Default duration of each DKG phase (10 blocks)
  - Maximum group capacity (10)
  - Number of groups in an equilibrium stage
  - Timeout (100 blocks)

<!--
TODO: <https://github.com/kafeikui/BLS-TSS-Network/blob/first-commit/crates/randcast-contract-mock/src/contract/controller.rs>
-->
#### 2.2.3 Coordinator Responsibilities
<!--
TODO: <https://github.com/kafeikui/BLS-TSS-Network/blob/first-commit/crates/randcast-contract-mock/src/contract/coordinator.rs>
-->
#### 2.2.4 Adapter Responsibilities
<!--
TODO: <https://github.com/kafeikui/BLS-TSS-Network/blob/first-commit/crates/randcast-contract-mock/src/contract/adapter.rs>
-->
### 2.3 Network Stages

#### 2.3.1 Bootstrapping Stage

#### 2.3.2 Equilibrium Stage

### 2.4 Network Scaling

#### 2.4.1 Up-scaling

#### 2.4.2 Down-scaling

### 2.N Requests & Tasks

### 2.N Multi-chain Support

## N. Networking

This section describes and defines the communication protocol between the nodes in the ARPA network.

## N. Interface

This section defines the interface of interacting with the ARPA network.

## N. Application & SDKs

This section describes the possible application of the ARPA network and the example SDKs for generating randomness via a BLS signature.

## N. Testing

This section describes the methodology for testing the network and the test scenarios.

## N. Security & Attack Vectors

## N. Terminology

TODO: re-organize the words below

TODO: define each word

### threshold signature

### Boneh-Lynn-Shacham (BLS) digital signature

#### BLS signature task

### distributed key generation (DKG)

#### DKG Phase 1

#### DKG Phase 2

#### DKG Phase 3

#### DKG Phase 4

#### DKG Task

##### threshold

##### DKG task epoch

#### DKG communication private key

#### DKG communication public key

#### partial private key

#### partial public key

#### DKG group public key

#### aggregated public key

### verification

### controller (smart contract)

#### global epoch

### coordinator (smart contract)

### adapter (smart contract)

### block height

### node

#### disqualified node

#### address

#### node state

#### staking

#### node index

#### node epoch

#### committer

#### member

### group

#### group capacity

#### group state

#### group index

#### group epoch

#### grouping

#### re-balance

#### pre-grouping

#### post-grouping

#### group relay

#### group relay task

## N. Cryptographic Protocols

### Aggregate and Verifiably Encrypted Signatures from Bilinear Maps

### Short Signatures from the Weil Pairing

### Compact Multi-Signatures for Smaller Blockchains

### BLS Multi-Signatures With Public-Key Aggregation

### Secure Distributed Key Generation for Discrete-Log Based Crypto-systems

key expiration  

___Components___ are the network-level entities.
___Modules___ are the node-level entities.
