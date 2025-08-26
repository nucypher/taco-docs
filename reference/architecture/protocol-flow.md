# Protocol Flow

This document details the operational flow of the TACo protocol - how information and operations move through the system. For an overview of the actors involved and their roles, see the [UML Use Case Diagram](uml-usecase-diagram.md).

The TACo protocol operations follow several distinct but interconnected flows:

## Cohort Formation

* Adopting developers (`cohortAuthority`) initiate and manage a one-time cohort formation network setup
* The initialization produces a logical group of nodes with each node maintaining its own private material
* The `cohortAuthority` manages parameters for the cohort
* A minimum of one honest party is required during cohort formation to ensure that private material is not spoofed

## Condition Configuration

* Programmable conditions define who can execute a successful TACo operation (decryption/signing). Conditions are either associated with encrypted payloads (encryption) or configured for a signing cohort.

## TACo Services

* Nodes independently verify that the consumer satisfy the requisite conditions before servicing any request
* For each node that validates the conditions, a response fragment is provided to the consumer
* Once a threshold of nodes have provided their fragments, the consumer can locally combine these fragments to complete the operation

## Cohort Management

* Node participation is secured through economic staking in the TACo Nodes Network
* Cohorts can rotate members according to predefined rules set by the `cohortAuthority`
* The rotation rules can be tailored to balance security, availability, and decentralization needs

For a more detailed explanation of the protocol operations, see [How TACo Works](../readme/key-concepts.md).

## Integration Points

The protocol flow integrates with various TACo components:

* **User applications** interact with the TACo protocol through the Client SDK (taco-web)
* **Programmable Conditions** enables the definition and validation of conditional execution of operations
* **TACo Nodes Network** provides the economic staking mechanism for node operators
* **Coordinator contracts** manage cohort formation on-chain

For details on how these components relate to each other architecturally, see [Protocol Architecture](protocol-architecture.md).

For conceptual explanations of the protocol's design principles, see [How TACo Works](../readme/key-concepts.md).
