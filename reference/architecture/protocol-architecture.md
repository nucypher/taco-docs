# TACo Protocol Architecture

TACo (Threshold Access Control) is a decentralized protocol that enables end-to-end encrypted data sharing with programmable access control. This document provides a comprehensive architectural overview of the protocol's components, layers, and their interactions.

<figure><img src="../../.gitbook/assets/architecture/taco-layers-and-components-diagram.svg" alt=""><figcaption><p>TACo Protocol Layers & Components</p></figcaption></figure>

## Architectural Overview

TACo's architecture consists of three primary layers that work together to provide secure, decentralized access control:

1. **Applications Layer** - Web and mobile applications that integrate with TACo
2. **Application Integration Layer** - SDK and interfaces for developers to implement TACo functionality
3. **Protocol Layer** - Core services and infrastructure for network operations
4. **Cryptographic Layer** - Fundamental cryptographic operations enabling secure data access control

These layers are not strictly hierarchical but rather interconnected systems that collaborate to achieve the protocol's objectives.

## Core Functional Components

### Cryptographic Layer

The cryptographic layer forms the security backbone of the entire protocol, enabling truly decentralized data control through four sequential operations:

#### 1. Distributed Key Generation (DKG)

**One-time setup operation:**

- A cohort of 30-100 nodes participates in a DKG ritual during network initialization
- Generates both public material (for encryption) and private key fragments (for later decryption)
- Each node holds only its own fragment, with no single entity having complete key access
- DKG happens only once at network setup, not periodically

#### 2. Encryption

**Client-side operation by data providers:**

- Encryption is performed locally by data owners using the public key
- Access conditions are cryptographically bound to the encrypted data
- No server involvement is required for the encryption process

#### 3. Decryption Fragment Generation

**Node-side condition verification:**

- Nodes independently verify that access conditions have been met
- When conditions are fulfilled, nodes generate partial decryption fragments
- Fragments are securely transmitted to authorized requestors

#### 4. Decryption

**Client-side operation using collected fragments:**

- Data consumers collect threshold number of fragments (e.g., 26 of 50)
- Local reconstruction combines fragments to recover the original data
- The decryption process is entirely client-side, maintaining end-to-end security

### Protocol Layer

The protocol layer manages the ongoing operation of the TACo network through several interconnected components:

#### Porter Service

**Protocol abstraction and gateway:**

- Provides an interface between applications and the TACo node network
- Simplifies protocol integration for developers
- Can be self-hosted or used as a public service (similar to Infura for Ethereum)

#### TACo Nodes Operations

**Core infrastructure management:**

- **Registration & Staking**: Economic security layer for node participation
- **DKG Rituals Participation**: Nodes participate in key generation triggered by Cohort Admin
- **Verification & Operations**: Condition validation and partial decryption services

#### Governance & Economics

**Network coordination and incentives:**

- **Protocol Governance**: Parameter management through the Coordinator Contract
- **Fee Collection**: Payment processing and distribution mechanisms
- **Staking & Slashing**: Economic incentives to ensure node compliance and availability
- **Cohort Authority Management**: Authority transfer and control mechanisms

#### Ritual Management System

**DKG administrative framework:**

- **RitualID Management**: Unique identifiers for specific DKG rituals
- **Ritual Coordination**: Coordinating node participation in DKG processes
- **Key Management**: Tracking public keys and threshold parameters
- **Nodes Monitoring**: Health checks using dummy DKG rituals

#### Blockchain Integration

**On-chain components:**

- **Coordinator Contract**: Manages protocol coordination
- **Staking Contract**: Handles node staking and security deposits
- **Fee Management**: Processes payment and revenue distribution

### Application Integration Layer

This layer bridges applications with the TACo protocol through developer-friendly interfaces:

#### Client SDK (taco-web)

**Application integration library:**

- Primary toolkit for developers to integrate TACo functionality
- Provides encrypt/decrypt APIs and cross-platform support
- Streamlines implementation of complex cryptographic operations

#### Authentication

**Web3 signature processing:**

- Requesting, collecting, and validating Web3 signatures
- Identity verification before processing decryption requests
- Secure authentication flow management

#### Access Control Framework

**Define decryption conditions:**

- Enables programmable access policies on a per-ciphertext basis
- Extensive condition types for flexible policy definition:
  - **TimeCondition**: Time-based access using blockchain timestamps
  - **RpcCondition**: Using RPC calls to Ethereum API
  - **ContractCondition**: On-chain state and contract function calls
  - **JsonApiCondition**: External API data sources
  - **CompoundCondition**: Logical combinations (AND/OR/NOT)
  - **Additional Conditions**: Extensible for custom verification logic

### Applications Layer

**End-user implementations:**

- **Web Apps**: Browser-based applications leveraging TACo
- **Mobile Apps**: Native or hybrid mobile applications

## Actors & Use Cases

The TACo ecosystem involves four primary actors interacting with the protocol in distinct ways:

### Key Participants

- **Adopting Developer (Cohort Authority)**: Application developers who adopt TACo, configure network parameters, and manage node cohorts
- **Data Producer**: Entities that encrypt data using TACo and define access conditions
- **Data Consumer**: Users who request and decrypt data when they meet the specified conditions
- **Node Operator**: Infrastructure providers who participate in DKG and provide decryption services

### Core Use Cases

#### Distributed Key Generation (DKG)

- **Primary Actor**: Adopting Developer
- **Supporting Actor**: Node Operator
- **Description**: Initial setup process where a cohort generates public encryption keys and distributes private fragments
- **Important Note**: DKG happens only once at network setup, not periodically

#### Cohort Management

- **Primary Actor**: Adopting Developer
- **Description**: Activities related to managing node cohorts, including economic staking and parameter configuration

#### Data Encryption with Conditional Access

- **Primary Actor**: Data Producer
- **Description**: Process of encrypting data locally using the public key and specifying precise access conditions

#### Conditional Data Decryption

- **Primary Actor**: Data Consumer
- **Supporting Actor**: Node Operator
- **Description**: Process involving condition verification, fragment provision, and client-side decryption

For detailed descriptions of these use cases and actor interactions, see the [UML Use Case Diagram](./uml-usecase-diagram.md).

## System Interconnections

### Layer Interactions

The TACo protocol's effectiveness comes from the coordinated interactions between its layers:

- **Applications → SDK**: End-user applications interact with the protocol through the Client SDK
- **SDK → Porter → Nodes**: The SDK communicates with nodes either directly or through Porter gateway services
- **Nodes ↔ Blockchain**: Node operations are coordinated and verified through on-chain contracts
- **Cryptographic ↔ Node Operations**: Nodes participate in DKG and later provide decryption fragments

### Data Flow Directions

- **Top-down** protocol flow: From applications through integration layer to protocol services
- **Top-down** cryptographic flow: From DKG through encryption, fragment generation, to decryption

This interconnected architecture ensures that the TACo protocol maintains security and privacy without requiring trust in any central authority, while providing developers with flexible tools for implementing sophisticated access control systems.

For a detailed explanation of information and operational flows, refer to the [Protocol Flow](./protocol-flow.md) document.
