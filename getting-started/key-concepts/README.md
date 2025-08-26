# How TACo Works

{% hint style="warning" %}
All third-party services – digital, Web3 and beyond – impose a trust burden on the end-user. TACo's decentralized architecture reduces this trust burden to a universal minima. Sensitive cryptographic operations – key generation, decryption fragment management, data signing – are each disassembled and distributed across a large group of independently operated nodes for individual execution. \
\
To further understand TACo's trust model and read the _Mainnet_ _Trust Disclosure_, head to [Trust Assumptions](../../for-product-leads/trust-assumptions/).&#x20;
{% endhint %}

## Overview

TACo is a decentralized network of nodes that provide a cryptographic control layer for applications. Whether you’re building a privacy-preserving data app or automating secure workflows with digital signatures, TACo provides a modular framework to embed conditional trust directly into your stack.

TACo provides two core primitives, both powered by threshold cryptography and programmable conditions:

* [🔐 Threshold Decryption](../../for-developers/access-control/): Securely share encrypted data that can only be decrypted when access control policies / conditions are satisfied. [Use cases](../../for-product-leads/access-control/use-cases/) are centred around controlled data sharing, private messaging, and zero-trust collaboration, such as seed-phrase recovery, digital rights management, trust-minimized communication channels for channels for journalists, archivists and whistleblowers
* [✍️ Threshold Signing](../../for-developers/action-control/): Authorize transactions or attestations with digital signatures only when the configured signing policies / conditions are met. Useful for smart contract wallet approvals, delegated authority, or automated, policy-bound transaction signing.

{% hint style="warning" %}
TACo Threshold Signing is currently in alpha and available only on the `DEVNET` environment.
{% endhint %}

While the cryptographic operations differ, both products follow the same high-level structure and lifecycle.

### **1. Node Cohort Formation**

TACo organizes a random subset of independent nodes from the network into identifiable groups known as **cohorts.** Each cohort is:

* Randomly assembled to reduce collusion risks
* Assigned a unique identifier
* Responsible for executing operations on behalf of users or applications
* Backed by a cryptographic set up process:
  * Decryption cohorts run a Distributed Key Generation (DKG) protocol to create a shared encryption key
  * Signing cohorts each register with a shared multisig contract and prepare to produce ECDSA signatures
  * Configured with a predefined threshold number of nodes that must respond to perform a successful operation; no single node can act alone.

### **2. Conditionality**

Before performing any action, each node in the cohort must independently verify that a submitted request satisfies configured [conditions](../../for-developers/taco-sdk/references/conditions/). These conditions may be associated with encrypted data (threshold decryption) or configured by the cohort authority (signing).&#x20;

A range of programmable condition types can be defined such as:&#x20;

* EVM-based\
  &#xNAN;_&#x65;.g. Does the requester own a given NFT?_
* RPC-driven\
  &#xNAN;_&#x65;.g. Does the requester have at least X amount of a given token in their wallet?_
* Time-based\
  &#xNAN;_&#x65;.g. Has a predefined period elapsed, after which requests will be ignored?_

Conditions are composable and can be combined in any logical sequence or decision tree. For more on condition logic, check out the [conditions](../../for-developers/taco-sdk/references/conditions/ "mention") section.&#x20;

These ensure that operations are always policy-bound and decentralized.

### **3. Threshold Operation**

Once a valid request is received and conditions are satisfied, a threshold of `t` cohort nodes must independently return a valid response. Once a threshold of these responses is received, the responses are combined into a final result.

This threshold design ensures both **fault tolerance** and **security**, protecting against node failures or malicious behavior.

## **Network parameterization**

In future versions, adopting developers (`cohortAuthority`) will have the option to tweak certain network-level parameters, which affect the collusion-resistance, redundancy, latency and costs of using TACo. These parameters mostly pertain to each cohort of nodes tasked with cryptographic management and condition verification:&#x20;

* The number of shares _n_ i.e. the size of the cohort&#x20;
* The frequency and/or business logic for replacing members of the cohort
* The 'hand-chosen' node address that will always feature in the cohort
* The sampling mechanism for selecting cohort members

This optionality can also be surfaced for end-users – for example, in the form of 'packages' that combine network-level parameters. End-users might choose between a few discrete options, based on their trust, risk and cost preferences.

## **Key Features**

* **Threshold-based coordination**: Independent network of nodes with no single point of failure. All operations require consensus from a decentralized node cohort
* **Trust-minimized control**: Enforce flexible, condition-based policies (e.g., wallet ownership, OAuth claims, scoped automated transactions, custom logic)
* **Unified policy engine**: The same conditions can be used with decryption and signing

