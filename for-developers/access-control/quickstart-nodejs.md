---
description: >-
  Encrypt and decrypt data with TACo in a Node.js environment — no browser or
  wallet extension required.
---

# Quickstart — Node.js (Testnet)

This guide walks through a complete encrypt → decrypt flow in Node.js. Unlike the browser quickstart, there's no MetaMask or `window.ethereum` — we use private keys and `JsonRpcProvider` directly.

{% hint style="info" %}
A full working example lives in the taco-web repo: [`examples/taco/nodejs`](https://github.com/nucypher/taco-web/tree/main/examples/taco/nodejs)
{% endhint %}

{% stepper %}
{% step %}
## Install dependencies

```bash
npm install @nucypher/taco @nucypher/taco-auth ethers@5.7.2
```

{% hint style="warning" %}
TACo currently requires **ethers v5**. The API is not compatible with ethers v6. If your project uses ethers v6, you must use `ethers@5.7.2` for TACo integration.
{% endhint %}
{% endstep %}

{% step %}
## Configure provider, domain, and ritual ID

You'll need three things:

| Parameter | Value | Notes |
|-----------|-------|-------|
| **RPC URL** | A Polygon Amoy RPC endpoint | This reads DKG coordination contracts — it's **not** your application's chain. Any public Amoy RPC works. |
| **Ritual ID** | `6` (for TAPIR testnet) | See the [testnets](../get-started-with-tac.md#threshold-decryption) page for other environments. |
| **Private keys** | Two test wallets | One for the encryptor, one for the consumer. Any Ethereum wallets — no testnet funds required. |

```typescript
import { ethers } from 'ethers';
import { domains } from '@nucypher/taco';

// This provider reads DKG coordination contracts on Polygon Amoy.
// It is NOT your application's RPC — it's infrastructure.
const provider = new ethers.providers.JsonRpcProvider(
  'https://polygon-amoy.drpc.org'
);

const domain = domains.TESTNET;  // "tapir"
const ritualId = 6;
```
{% endstep %}

{% step %}
## Encrypt data

```typescript
import { conditions, encrypt, initialize } from '@nucypher/taco';
import { ethers } from 'ethers';

await initialize();

// Any condition works — here's a simple balance check
const hasBalance = new conditions.base.rpc.RpcCondition({
  chain: 80002,  // Polygon Amoy — the chain where this condition is evaluated (can be any supported chain)
  method: 'eth_getBalance',
  parameters: [':userAddress', 'latest'],
  returnValueTest: {
    comparator: '>=',
    value: 0,
  },
});

const encryptorSigner = new ethers.Wallet('<ENCRYPTOR_PRIVATE_KEY>');

const messageKit = await encrypt(
  provider,
  domain,
  'my secret message',
  hasBalance,
  ritualId,
  encryptorSigner,
);

// Serialize for storage or transmission
const encryptedBytes = messageKit.toBytes();
```

{% hint style="warning" %}
**Do not JSON-serialize `ThresholdMessageKit` directly** — it uses a custom binary format. Always use `.toBytes()` for serialization and `ThresholdMessageKit.fromBytes()` for deserialization. If you need to store it in JSON, encode the bytes as base64 or hex first.
{% endhint %}
{% endstep %}

{% step %}
## Decrypt data

```typescript
import {
  ThresholdMessageKit,
  conditions,
  decrypt,
  initialize,
} from '@nucypher/taco';
import {
  EIP4361AuthProvider,
  USER_ADDRESS_PARAM_DEFAULT,
} from '@nucypher/taco-auth';
import { ethers } from 'ethers';

await initialize();

// Deserialize
const messageKit = ThresholdMessageKit.fromBytes(encryptedBytes);

const consumerSigner = new ethers.Wallet('<CONSUMER_PRIVATE_KEY>');

// Build condition context
const conditionContext =
  conditions.context.ConditionContext.fromMessageKit(messageKit);

// In Node.js, EIP4361AuthProvider needs explicit SIWE parameters
// (in the browser, these come from the page origin)
const authProvider = new EIP4361AuthProvider(
  provider,
  consumerSigner,
  { domain: 'localhost', uri: 'http://localhost:3000' },
);
conditionContext.addAuthProvider(USER_ADDRESS_PARAM_DEFAULT, authProvider);

const decryptedBytes = await decrypt(
  provider,
  domain,
  messageKit,
  conditionContext,
);

const decryptedMessage = new TextDecoder().decode(decryptedBytes);
console.log(decryptedMessage); // "my secret message"
```
{% endstep %}
{% endstepper %}

## Key differences from browser usage

{% tabs %}
{% tab title="Provider" %}
**Browser:**
```typescript
const provider = new ethers.providers.Web3Provider(window.ethereum);
```

**Node.js:**
```typescript
const provider = new ethers.providers.JsonRpcProvider('https://polygon-amoy.drpc.org');
```
{% endtab %}

{% tab title="Signer" %}
**Browser:**
```typescript
const signer = provider.getSigner(); // MetaMask popup
```

**Node.js:**
```typescript
const signer = new ethers.Wallet('<PRIVATE_KEY>');
```
{% endtab %}

{% tab title="SIWE Auth" %}
**Browser:** Domain and URI inferred from page origin.
```typescript
const authProvider = new EIP4361AuthProvider(provider, signer);
```

**Node.js:** Must pass `{ domain, uri }` explicitly.
```typescript
const authProvider = new EIP4361AuthProvider(provider, signer, {
  domain: 'localhost',
  uri: 'http://localhost:3000',
});
```
{% endtab %}

{% tab title="Serialization" %}
**Browser:** `messageKit` often stays in memory.

**Node.js:** Use binary serialization for storage or transmission.
```typescript
const bytes = messageKit.toBytes();
const restored = ThresholdMessageKit.fromBytes(bytes);
```
{% endtab %}
{% endtabs %}

## Next steps

* Browse all [condition types](../taco-sdk/references/conditions/) to define more complex access rules
* See the [full Node.js example](https://github.com/nucypher/taco-web/tree/main/examples/taco/nodejs) in the taco-web repo
* Review [testnets](../get-started-with-tac.md) for ritual IDs and network configuration
