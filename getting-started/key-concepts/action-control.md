---
description: Conditions-based Threshold Signing
---

# Action Control

{% hint style="warning" %}
TACo Threshold Signing is currently in alpha and available only on the `DEVNET` environment.
{% endhint %}

_Action Control_ enables the generation of user-controlled, automated, multi-party digital signatures leveraging both signing data-specific and existing conditions available in TACo.

Like access control, action control is executed by a decentralized cohort of nodes, but instead of recovering a plaintext if conditions are satisfied, they collaboratively produce a digital signature. This aggregated signature can be used to authorize account-abstraction blockchain transactions, attest to off-chain data, or approve sensitive workflows – **only if the pre-specified conditions are fulfilled**.&#x20;

## Key concepts

### **Threshold Signing**

TACo's Action Control works similarly to a multisig system: each node in a signing cohort independently produces an ECDSA signature, and a **threshold** number of those signatures are aggregated by simply appending them together.

A threshold of cohort nodes must be online and approve the request by signing the provided payload. The resulting overall signature is an ordered bundle of 65-byte ECDSA signatures.

Each signing cohort is associated with a **multisig smart contract** (for each relevant supported chain) that implements `isValidSignature(hash, signature)` according to [ERC-1271](https://eips.ethereum.org/EIPS/eip-1271). This contract can validate the aggregated signature against the known set of signing addresses for the cohort.

Alternatively, integrators can implement their own verification logic:

* Split the aggregated signature into 65-byte chunks.
* Recover the signer from each chunk.
* Ensure that each recovered address is part of the cohort and that at least the threshold number have signed.

### **Conditionality**

Signing conditions are at the heart of TACo Action Control's utility. They can be used as safety mechanisms/guardrails, to program oracle logic, or to facilitate automation workflows.&#x20;

There

* [Signing Object Conditions](../../for-developers/taco-sdk/references/conditions/signing-object-conditions.md) \
  &#xNAN;_&#x45;xample:_ only sign the UserOperation if the transaction sum is < 0.1 ETH.&#x20;
* [TimeCondition](../../for-developers/taco-sdk/references/conditions/timecondition.md) \
  &#xNAN;_&#x45;xample:_ only sign after a certain timestamp.
* [RpcCondition](../../for-developers/taco-sdk/references/conditions/rpccondition.md) \
  &#xNAN;_&#x45;xample:_ only sign if the requestor address holds a minimum ETH balance.
* [ContractCondition](../../for-developers/taco-sdk/references/conditions/contractcondition/)\
  &#xNAN;_&#x45;xample:_ only sign if the execution wallet holds a special-purpose NFT.&#x20;
* [JSON Endpoint Conditions](../../for-developers/taco-sdk/references/conditions/json-endpoint-conditions/)\
  &#xNAN;_&#x45;xample:_ only sign – enabling an executable discount on event tickets – if the temperature is below freezing, according to a multiple weather APIs.&#x20;
* [JWTCondition](../../for-developers/taco-sdk/references/conditions/jwtcondition.md) \
  &#xNAN;_&#x45;xample:_ only sign a pre-generated JWT if it is scoped to expire within 1 minute.&#x20;

Conditions are defined **per chain**. This means that: you can define different signing conditions for different chains (e.g., Ethereum Mainnet vs. Base). However, a single cohort **cannot** have multiple sets of conditions for the **same chain**. This design ensures that policy enforcement remains deterministic and auditable per environment.&#x20;

## Signing Flow

At it's simplest, TACo Action Control works as follows:

1. Define top-level signing [conditions](../../for-developers/taco-sdk/references/conditions/) for the cohort of TACo nodes. \
   This could include introspection of the object being signed, wallet ownership, contract call results, Web 2.0 responses. Only a `cohortAuthority`  can set top-level conditions.
2. Clients submit signing requests to the Action Control cohort, including:
   * The identifier for the cohort.
   * The chain ID for the signature.
   * The data object to sign e.g. a `UserOperation` from [ERC-4337](https://www.erc4337.io/), attestation data, etc.
3. Nodes in the Action Control cohort independently evaluate the conditions. If they are satisfied, nodes will each produce an ECDSA signature, with a threshold of signatures required.
4. The Action Control API assembles the aggregated ECDSA signatures into a threshold of signatures. This can be returned to the client and verified using the multisig contract associated with the signing cohort. Alternatively the aggregated signature can be split into it's corresponding individual signatures and verified them based on each node signer addresses.

## Use Cases

Action Control introduces a new primitive in the TACo toolkit:

* **Automated authorization** with policy-controlled triggers.
  * Composable with existing signing/decryption conditions.
  * Guardrails built-in **-** custom conditions, limited scope of actions.
* **Decentralized attestations** for identity, credentials, or events.
  * No single point of failure – signing requires consensus among nodes,
* **Verifiable approvals** for smart contracts or external systems.
  * Delegated signing with user-defined constraints.
