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

### 1. Installation

```bash
npm install @nucypher/taco @nucypher/taco-auth ethers@5.7.2
```

{% hint style="warning" %}
TACo currently requires **ethers v5**. If you're using ethers v6, you'll need to install v5 alongside it or use the v5 compatibility layer.
{% endhint %}

### 2. Configuration

You'll need three things:

| Parameter | Value | Notes |
|-----------|-------|-------|
| **RPC URL** | A Polygon Amoy RPC endpoint | This reads DKG coordination contracts — it's **not** your application's chain. Any public Amoy RPC works (e.g. `https://polygon-amoy.drpc.org`). |
| **Ritual ID** | `6` (for TAPIR testnet) | See the [testnets](../get-started-with-tac.md#threshold-decryption) page for other environments. |
| **Private keys** | Two test wallets | One for the encryptor, one for the consumer. These can be any Ethereum wallets — no testnet funds required. |

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

### 3. Encrypt

```typescript
import { conditions, encrypt, initialize, toBytes } from '@nucypher/taco';

await initialize();

// Any condition works — here's a simple balance check
const hasBalance = new conditions.base.rpc.RpcCondition({
  chain: 80002,  // Polygon Amoy (condition evaluation chain, not the DKG chain)
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

### 4. Decrypt

```typescript
import {
  ThresholdMessageKit,
  conditions,
  decrypt,
  domains,
  fromBytes,
  initialize,
} from '@nucypher/taco';
import {
  EIP4361AuthProvider,
  USER_ADDRESS_PARAM_DEFAULT,
} from '@nucypher/taco-auth';

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

const decryptedMessage = fromBytes(decryptedBytes);
console.log(decryptedMessage); // "my secret message"
```

### Key differences from browser usage

| Concern | Browser | Node.js |
|---------|---------|---------|
| **Provider** | `new ethers.providers.Web3Provider(window.ethereum)` | `new ethers.providers.JsonRpcProvider(rpcUrl)` |
| **Signer** | `provider.getSigner()` (MetaMask popup) | `new ethers.Wallet(privateKey)` |
| **SIWE auth** | Domain/URI inferred from page | Must pass `{ domain, uri }` explicitly |
| **Serialization** | Often stays in memory | Use `messageKit.toBytes()` / `ThresholdMessageKit.fromBytes()` |

### Next steps

* Browse all [condition types](../taco-sdk/references/conditions/) to define more complex access rules
* See the [full Node.js example](https://github.com/nucypher/taco-web/tree/main/examples/taco/nodejs) in the taco-web repo
* Review [testnets](../get-started-with-tac.md) for ritual IDs and network configuration
