# Testnets

{% hint style="warning" %}
Testnets exist to help developers familiarize themselves with the taco-web API, and prototype an integration before deploying on TACo Mainnet. \
\
**Testnets should not be utilized in production with any expectation of a trust-minimized integration**.&#x20;

Hence, the trust assumptions are strictly worse than the Mainnet version, particularly with respect to the node array managing decryption fragments and validating condition fulfillment. Testnet nodes are are operated primarily by members of the NuCypher core developer team, and are not subject to a cryptoeconomic protocol nor required to stake any collateral.
{% endhint %}

## Testnet domains

To run TACo on testnet, it needs to be configured to use one of the two available domains:

```typescript
import { domains } from '@nucypher/taco';

domains.DEVNET  // "lynx" network
domains.TESTNET // "tapir" network
```

* `DEVNET` domain, or `lynx`, is a bleeding-edge developer network that supports upcoming `taco` release.
* `TESTNET` domain, or `tapir`, is a stable testnet environment that supports the current `taco` release.

We encourage you to use the `TESTNET` domain for developing `taco` based apps, and to use `DEVNET` to test compatibility with the upcoming `taco` release and new, experimental features.

{% hint style="warning" %}
Both `DEVNET` and `TESTNET` domains are unsuitable for use in a production setting. Testnet domains have no trust minimization or stability guarantees, which makes them unfit for production or real-world data payloads. Learn more about this in the trust assumptions [section](../../for-product-leads/trust-assumptions/).
{% endhint %}

## Testnet configuration

<table><thead><tr><th width="121">Domain</th><th width="96">Network</th><th width="132">L1 (chain id)</th><th width="126">L2 (chain id)</th><th width="134" data-type="number">Open ritualID</th><th>Cohort</th></tr></thead><tbody><tr><td>DEVNET</td><td><a href="https://lynx-3.nucypher.network:9151/status">lynx</a></td><td>Sepolia<br>(11155111)</td><td>Amoy (80002)</td><td>27</td><td>2-of-3</td></tr><tr><td>TESTNET</td><td><a href="https://tapir-3.nucypher.network:9151/status">tapir</a></td><td>Sepolia<br>(11155111)</td><td>Amoy (80002)</td><td>6</td><td>3-of-5</td></tr></tbody></table>

{% hint style="info" %}
**Open ritualID** refers to a DKG cohort & public key with no restrictions on encryptors – i.e. any device or address can use the public key to encrypt data. See the [encryptor allowlist](../references/encryptor-allowlist.md) section to learn more.
{% endhint %}

{% hint style="info" %}
It should be noted that the blockchains used as L1 and L2 in the various TACo domains (DEVNET, TESTNET, MAINNET) do not determine the blockchains supported by the TACo conditions. For example, the TESTNET and MAINNET domains can be used to define conditions on the Ethereum mainnet.
{% endhint %}

## Contracts

The source code for contracts used on testnet can be found in [`nucypher/nucypher-contracts`](https://github.com/nucypher/nucypher-contracts) repository.&#x20;

Contract addresses for testnets can be found in their respective contract registries:

* [`lynx.json`](https://github.com/nucypher/nucypher-contracts/blob/main/deployment/artifacts/lynx.json)
* [`tapir.json`](https://github.com/nucypher/nucypher-contracts/blob/main/deployment/artifacts/tapir.json)
