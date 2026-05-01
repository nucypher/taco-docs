# Testnet

{% hint style="warning" %}
Testnet exists to help developers familiarize themselves with the taco-web API, and prototype an integration before deploying on TACo Mainnet.\
\
**Testnet should not be utilized in production with any expectation of a trust-minimized integration**.

Hence, the trust assumptions are strictly worse than the Mainnet version, particularly with respect to the nodes managing decryption fragments, signing wallets, and validating condition fulfillment. Testnet nodes are are operated primarily by the TACo team, and are not subject to a crypto-economic protocol nor required to stake any collateral.
{% endhint %}

## Testnet domains

To run TACo on testnet, it needs to be configured to use one of the two available domains:

```typescript
import { domains } from '@nucypher/taco';

domains.DEVNET  // "lynx" network
```

* `DEVNET` domain, or `lynx`, is a bleeding-edge developer network that supports the upcoming `taco` release.

{% hint style="warning" %}
`DEVNET` is unsuitable for use in a production setting since it does not have any trust minimization or stability guarantees, which makes it unfit for production or real-world data payloads. Learn more about this in the trust assumptions [section](../for-product-leads/trust-assumptions/).
{% endhint %}

## Quick reference

Copy-paste these values to get started immediately. These examples use **ethers v5** (`npm install ethers@5.7.2`), which TACo currently requires.

{% tabs %}
{% tab title="Lynx (bleeding-edge)" %}
```typescript
import { domains } from '@nucypher/taco';
import { ethers } from 'ethers';

const domain = domains.DEVNET;
const ritualId = 27;  // Open ritual, no encryptor allowlist needed

// Provider must connect to Polygon Amoy (where DKG contracts live)
const provider = new ethers.providers.JsonRpcProvider(
  'https://polygon-amoy.drpc.org'
);
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
The `provider` configured above reads DKG coordination contracts on the L2 chain (Polygon Amoy for testnet, Polygon for mainnet). It is required by `encrypt()` and can also be used with `decrypt()`. In browser contexts, `decrypt()` typically uses a `Web3Provider` connected to the user's wallet instead. Neither provider determines which chains your conditions can target — conditions can reference any supported EVM chain.
{% endhint %}

## Testnet configuration

### Threshold Decryption

<table><thead><tr><th width="111.73046875">Domain</th><th width="104.45703125">Network</th><th width="97.421875">L1</th><th width="149.21875">L2</th><th width="132.953125" data-type="number">Open Ritual ID</th><th>Cohort</th></tr></thead><tbody><tr><td>DEVNET</td><td><a href="https://lynx-3.nucypher.network:9151/status">lynx</a></td><td>Sepolia<br>(11155111)</td><td>Amoy (80002)</td><td>27</td><td>2-of-3</td></tr></tbody></table>

{% hint style="info" %}
**Open Ritual ID** refers to a DKG cohort & public key with no restrictions on encryptors – i.e. any device or address can use the public key to encrypt data. See the [encryptor allowlist](access-control/taco-integration/encryptor-allowlist.md) section to learn more.
{% endhint %}

{% hint style="info" %}
It should be noted that the blockchains used as L1 and L2 in the various TACo domains (`DEVNET`, `MAINNET`) do not determine the blockchains supported by the TACo conditions. For example, the `DEVNET` can be used to define conditions on Ethereum mainnet.
{% endhint %}

### Threshold Signing

<table><thead><tr><th width="102.328125">Domain</th><th width="110.19921875">Network</th><th width="92.296875">L1</th><th width="150.02734375" data-type="number">Open Cohort ID</th><th width="102.4609375">Cohort</th><th>Supported Chains</th></tr></thead><tbody><tr><td>DEVNET</td><td><a href="https://lynx-3.nucypher.network:9151/status">lynx</a></td><td>Sepolia<br>(11155111)</td><td>1</td><td>2-of-3</td><td><ul><li>84532 (Base)</li><li>11155111 (Sepolia)</li></ul></td></tr></tbody></table>

{% hint style="info" %}
**Open Cohort ID** refers to a signing cohort with a policy condition that always passes and therefore there are no restrictions on what it will sign – i.e. any signing request will be fulfilled.
{% endhint %}

## Contracts

The source code for contracts used on testnet can be found in [`nucypher/nucypher-contracts`](https://github.com/nucypher/nucypher-contracts) repository.

Contract addresses for testnet can be found in their respective contract registries:

* [`lynx.json`](https://github.com/nucypher/nucypher-contracts/blob/main/deployment/artifacts/lynx.json)
