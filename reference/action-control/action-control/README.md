# Action Control

{% hint style="warning" %}
TACo Threshold Signing is currently in alpha and available only on the `DEVNET` environment.
{% endhint %}

TACo Action Control enables decentralized, conditional signing of data using threshold cryptography. It allows applications to request signatures from a distributed network of nodes, where signatures are only generated when predefined conditions are met.

## Problems Addressed

### 1. Centralized Signing Authority

**Problem**: Traditional signing systems rely on a single key holder, creating security vulnerabilities and single points of failure.

**Solution**: TACo distributes signing authority across multiple nodes using threshold cryptography. No single entity can unilaterally generate signatures, reducing the risk of non-availability, key compromise, or malicious signing.

### 2. Unconditional Signing Powers

**Problem**: Once an entity is granted signing authority, they can sign any data at any time without restrictions. Providing unrestricted access to private keys to automated system such as bots or trading agents poses significant security risks.

**Solution**: TACo allows delegating limited signing authority to automated systems and strictly enforces conditional signing, where signatures are produced only when predefined conditions are independently verified by multiple signers, each and every time. This enables fine-grained, dynamic control over what can be signed and under what circumstances, without exposing master keys.

## TACo Signing vs. Alternatives

* **Custodial Signing APIs.** Centralized services (e.g., Fireblocks, Coinbase Custody) can be used to manage signing keys on behalf of users or applications. However these present with a &#x73;_&#x69;ngle point of failure and trust,_ where if the provider is compromised or misbehaves, the signing authority is at risk. There is typically no native support for programmable access control.
* **Threshold MPC Services.** Proprietary threshold signing solutions using multi-party computation (MPC) are often provided as SaaS. Such services are opaque and vendor-locked. The policy or signing condition logic is often inaccessible or non-verifiable, and reliance on a single provider reduces decentralization and auditability.
* **Basic Multisig Wallets.** On-chain wallets (e.g., Gnosis Safe) similarly require M-of-N approvals to execute transactions. However, practically, they are manual and rigid, requiring individual wallet owners to coordinate signatures manually. It also lacks support for automated, dynamic policies based on real-time data or off-chain claims.
* **Single-Key Wallets.** Traditional wallets or smart contract accounts can be secured by a single private key. In such cases, there is no fault tolerance and if the key is lost or compromised, the signing authority is irrecoverably broken. Additionally, there is no native support for automated/conditional logic or multi-party governance.
