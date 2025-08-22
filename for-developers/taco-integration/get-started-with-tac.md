# TACo Domains

{% hint style="warning" %}
Testnets exist to help developers familiarize themselves with the taco-web API, and prototype an integration before deploying on TACo Mainnet. \
\
**Testnets should not be utilized in production with any expectation of a trust-minimized integration**.&#x20;

Hence, the trust assumptions are strictly worse than the Mainnet version, particularly with respect to the node array managing decryption fragments and validating condition fulfillment. Testnet nodes are are operated primarily by members of the NuCypher core developer team, and are not subject to a cryptoeconomic protocol nor required to stake any collateral.
{% endhint %}

## Domain Overview

TACo supports three domains for different use cases:

| Description                             | TACo Domain | Network Type | Production | Chain ID | Available Rituals                                                                                                                                                                                                              |
| --------------------------------------- | ----------- | ------------ | ---------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Bleeding-edge developer network         | lynx        | DEVNET       | ❌ No      | 80002    | **Ritual 27** (Cohort 2-of-3)<br/>🟢 **Open** - No encryptor restrictions<br/>_Any address can encrypt data_                                                                                                                   |
| Stable testnet for current TACo release | tapir       | TESTNET      | ❌ No      | 80002    | **Ritual 6** (Cohort 4-of-6)<br/>🟢 **Open** - No encryptor restrictions<br/>_Any address can encrypt data_                                                                                                                    |
| Production network                      | mainnet     | MAINNET      | ✅ Yes     | 137      | **No open rituals freely available**<br/>🔒 **Custom setup required**: Every dapp needs to have a custom ritual setup<br/>_Contact TACo team for ritual creation. You can reach on [Discord](http://discord.gg/buildwithtaco)_ |

{% hint style="info" %}
**Open ritualID** refers to a DKG cohort & public key with no restrictions on encryptors – i.e. any device or address can use the public key to encrypt data. See the [encryptor allowlist](../references/encryptor-allowlist.md) section to learn more.
{% endhint %}

{% hint style="info" %}
It should be noted that the blockchains used as L1 and L2 in the various TACo domains (DEVNET, TESTNET, MAINNET) do not determine the blockchains supported by the TACo conditions. For example, the TESTNET and MAINNET domains can be used to define conditions on the Ethereum mainnet.
{% endhint %}

To configure TACo for different domains:

```typescript
import { domains } from "@nucypher/taco";

domains.DEVNET; // "lynx" domain
domains.TESTNET; // "tapir" domain
domains.MAINNET; // "mainnet" domain
```

## Domain Details

### DEVNET Domain (lynx)

**Bleeding-edge developer network**

- **Network Type:** DEVNET
- **Production:** ❌ No
- **Chain:** Polygon Amoy (80002) - Your provider needs to be connected to this chain.
- **L1 Chain:** Sepolia (11155111) - For you, this is just a metadata field.
- **Status:** Testnet (Development) - May have breaking changes

**Connectivity:**

- RPC URL: Use an RPC provider that is connected to Polygon Amoy (80002).
- Monitoring Portal: [lynx status](https://lynx-3.nucypher.network:9151/status)

**Rituals:**

- **Ritual 27:** 🟢 Open | Cohort: 2-of-3 | No encryptor restrictions

**Usage Example:**

```typescript
import { encrypt, decrypt, domains } from "@nucypher/taco";

// Encrypt data
const messageKit = await encrypt(
  provider,
  domains.DEVNET, // "lynx"
  message,
  condition,
  27, // ritualId
  signer
);

// Decrypt data
const decryptedMessage = await decrypt(
  provider,
  domains.DEVNET, // "lynx"
  messageKit,
  signer
);
```

### TESTNET Domain (tapir)

**Stable testnet for current TACo release** _(Recommended for development)_

- **Network Type:** TESTNET
- **Production:** ❌ No
- **Chain:** Polygon Amoy (80002) - Your provider needs to be connected to this chain.
- **L1 Chain:** Sepolia (11155111) - For you, this is just a metadata field.
- **Status:** Testnet (Stable) - Recommended for development and testing

**Connectivity:**

- RPC URL: Use an RPC provider that is connected to Polygon Amoy (80002).
- Monitoring Portal: [tapir status](https://tapir-2.nucypher.network:9151/status)

**Rituals:**

- **Ritual 6:** 🟢 Open | Cohort: 4-of-6 | No encryptor restrictions

**Usage Example:**

```typescript
import { encrypt, decrypt, domains } from "@nucypher/taco";

// Encrypt data
const messageKit = await encrypt(
  provider,
  domains.TESTNET, // "tapir"
  message,
  condition,
  6, // ritualId
  signer
);

// Decrypt data
const decryptedMessage = await decrypt(
  provider,
  domains.TESTNET, // "tapir"
  messageKit,
  signer
);
```

### MAINNET Domain (mainnet)

**Production network**

- **Network Type:** MAINNET
- **Production:** ✅ Yes
- **Chain:** Polygon (137) - Your provider needs to be connected to this chain.
- **L1 Chain:** Ethereum (1) - For you, this is just a metadata field.
- **Status:** Production - Requires custom ritual setup and payment

**Connectivity:**

- RPC URL: Use an RPC provider that is connected to Polygon (137).
- Monitoring Portal: Contact TACo team for production monitoring Portal

**Rituals:**

- No open free rituals - Every dapp needs to have a custom ritual setup - Contact TACo team for custom ritual setup

**Usage Example:**

```typescript
import { encrypt, decrypt, domains } from "@nucypher/taco";

// Encrypt data
const messageKit = await encrypt(
  provider,
  domains.MAINNET, // "mainnet"
  message,
  condition,
  YOUR_RITUAL_ID, // Your DApp Custom ritual ID
  signer
);

// Decrypt data
const decryptedMessage = await decrypt(
  provider,
  domains.MAINNET, // "mainnet"
  messageKit,
  signer
);
```

{% hint style="warning" %}
Both `DEVNET` and `TESTNET` domains are unsuitable for use in a production setting. Testnet domains have no trust minimization or stability guarantees, which makes them unfit for production or real-world data payloads. Learn more about this in the trust assumptions [section](../../for-product-leads/trust-assumptions/).
{% endhint %}

## Contracts

You do not need to check the contracts to integrate TACo into your DApp. The magic happens under the hood. However, if you are curious, you can take a look at them.

The source code for contracts used on the various TACo domains can be found in [`nucypher/nucypher-contracts`](https://github.com/nucypher/nucypher-contracts) repository.

And the contract addresses can be found in their respective contract registries:

- [`lynx.json`](https://github.com/nucypher/nucypher-contracts/blob/main/deployment/artifacts/lynx.json)
- [`tapir.json`](https://github.com/nucypher/nucypher-contracts/blob/main/deployment/artifacts/tapir.json)
- [`mainnet.json`](https://github.com/nucypher/nucypher-contracts/blob/main/deployment/artifacts/mainnet.json)
