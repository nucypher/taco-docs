# Action Control

{% hint style="warning" %}
TACo Action Control is currently in alpha and available only on the `DEVNET` environment.
{% endhint %}

TACo Action Control enables decentralized, conditional signing of data using threshold cryptography. It allows applications to request signatures from a distributed network of nodes, where signatures are only generated when predefined conditions are met.

## Problems Addressed

### 1. Centralized Signing Authority

**Problem**: Traditional signing systems rely on a single key holder, creating security vulnerabilities and single points of failure.

**Solution**: TACo distributes signing authority across multiple nodes using threshold cryptography. No single entity can unilaterally generate signatures, reducing the risk of non-availability, key compromise,  or malicious signing.&#x20;

### 2. Unconditional Signing Powers

**Problem**: Once an entity is granted signing authority, they can sign any data at any time without restrictions.

**Solution**: TACo Action Control strictly follows conditional signing, where signatures are only generated when predefined conditions are met, each and every time. This enables fine-grained, dynamic control over what can be signed and under what circumstances.

### 3. Bot/Agent Authorization

**Problem**: Automated systems (bots, trading agents) need signing capabilities but giving them unrestricted private keys is very risky.

**Solution**: TACo allows delegating limited signing authority to automated systems with specific conditions, enabling secure bot/agent operations without exposing master keys.
