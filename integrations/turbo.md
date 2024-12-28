---
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Turbo

This guide explains how to integrate TACo with [Turbo](https://docs.ardrive.io/docs/turbo/what-is-turbo.html) – a bundler SDK which facilitates uploading, retrieving, indexing, and paying for data hosted on [Arweave](https://search.brave.com/search?q=arweave+docs\&source=desktop). Developers who combine TACo and Turbo can offer their end-users permanent data storage that is _private by default_, wherein all sensitive payloads are encrypted, with fine-grained access conditions, prior to being uploaded. Given that Arweave offers access to data in perpetuity, it is even more critical that non-public data will only be decryptable by legitimate and qualifying consumers.&#x20;

## Turbo overview

Turbo is an OSS kit for the permaweb – helping developers upload and pay for Arweave data storage. It supports ETH payments and Ethereum-based identities, a level of interoperability that makes integration with TACo straightforward. Turbo offers optimistic data caching and indexing, and performant transaction throughput (860/sec).&#x20;

## Use case ideas

* **Evidence.** Empower researchers and journalists to store critical evidence and analysis on the permaweb, without making all the findings public prematurely. This removes the burden of trusting centralized hosting, which could surveil or deny service to users, and means sensitive evidence can be selectively shared with reviewers and publishers.&#x20;
* **Health records.** Don't rely on HIPAA or the moral compass of medical data custodians. Build health applications that encrypt highly sensitive patient data, such that the patient has full sovereignty over who has access to said data, and under what conditions. This encrypted data can be stored via Turbo, so patients have access to their valuable records forever.&#x20;
* **Private NFTs.** The status quo of artistic NFT asset 'ownership' is little more than holding a symbolic receipt, with the actual asset hosted by centralized gate-keepers. Combine TACo, Turbo and Arweave to offer end-users the ability to purchase the _decryption rights_ to a movie, track, in-game asset, or piece of art – without trusting either the access control or storage layers in order to play, watch or view their assets.&#x20;

***

## Integration steps&#x20;

### 1. Install TACo & Turbo&#x20;

```typescript
yarn add @nucypher/taco
yarn add @ardrive/turbo-sdk
yarn add arbundles
```

### 2. Define access conditions & encrypt the data&#x20;

Next, initialize the `taco-web` [library](https://github.com/nucypher/taco-web). This provides, among other functionality, the facility to create access conditions and embed them in the encrypted data payload. The conditions and ciphertext are inextricable, which ensures that only a threshold of TACo nodes can collectively enforce access.&#x20;

In this example, the _data_ _producer_ is composing a _time-based_ condition for acces&#x73;_._ This requires reading the block height of the Arweave chain, and means _data consumers_ may only access the data once a certain block height has been reached. For more on condition types, check out the [Access Control](../conditions/) section.\
\
We encrypt the message using the `blockHeight` condition, specifying the devnet `domain` and `ritualID`, and a standard web3 provider/signer.&#x20;

{% hint style="warning" %}
This guide utilizes the parameters `ritualId = 27` & `domains.DEVNET`. These refer to an open DKG public key and hacker-facing bleeding-edge testnet respectively, which  are **not decentralized** and unsuitable for real-world sensitive data. For more information, see the [Trust Assumptions](../trust-assumptions/) section.&#x20;
{% endhint %}

<pre class="language-typescript"><code class="lang-typescript">import { encrypt, conditions, domains, initialize, toHexString } from '@nucypher/taco';
<strong>import { ethers } from "ethers";
</strong>
// We have to initialize the TACo library first
await initialize();

const web3Provider = new ethers.providers.Web3Provider(window.ethereum);

const blockHeight = new conditions.base.jsonApi.JsonApiCondition({
  endpoint: 'https://arweave.net/info',
  query: '$.height',
  returnValueTest: {
    comparator: '>',
    value: 1556508,
  },
});
const ritualId = 27

const message = "this will be here forever";

const messageKit = await encrypt(
  web3Provider,
  domains.DEVNET,
  message,
  blockHeight,
  ritualId,
  web3Provider.getSigner() 
);
const encryptedMessageHex = toHexString(messageKit.toBytes());
</code></pre>

### 3. Connect to Turbo & store the data&#x20;

&#x20;Next, connect to a Turbo signer.&#x20;

{% hint style="info" %}
In this example, the data payload size is lower than 100kb, so it is currently free to upload to Arweave via the Turbo SDK.&#x20;
{% endhint %}

```typescript
import { TurboFactory, TurboSigner } from '@ardrive/turbo-sdk/web';
import { InjectedEthereumSigner } from 'arbundles';

const signer = new InjectedEthereumSigner(provider);
await signer.setPublicKey();
const turbo = TurboFactory.authenticated({
  signer: signer as unknown as TurboSigner,
});

const dataItem = createData(encryptedMessageHex, signer);
await dataItem.sign(signer);
const response = await turbo.uploadSignedDataItem({
  dataItemStreamFactory: () => dataItem.getRaw(),
  dataItemSizeFactory: () => dataItem.getRaw().byteLength,
});
console.log({ response });
const encryptedMessageId = response.id;
```

### 4. Retrieve & decrypt the data

Next, use the `encryptedMessageId` to find and retrieve the encrypted payload. This can be done via any Arweave gateway.

TACo's decryption logic involves (1) a decryption request sent to the TACo nodes enforcing access control, (2) for decryption fragments to be retrieved and assembled locally, and (3) finally for the payload to be decrypted into plaintext. These steps are all contained in the `decrypt()` function below.&#x20;

```typescript
import { conditions, decrypt, ThresholdMessageKit } from '@nucypher/taco';

const response = await fetch(`https://arweave.net/${encryptedMessageId}`);
const dataJson = await response.text();
const encryptedMessage = ThresholdMessageKit.fromBytes(
  Buffer.from(JSON.parse(dataJson), 'hex'),
);

const decryptedMessage = await decrypt(
  web3Provider,
  domains.TESTNET,
  encryptedMessage,
);

console.log(decryptedMessage);
```

***

## Using Turbo & TACo in production&#x20;

As noted, the parameters specified in this guide are for testing and hacking only.

For TACo, a funded Mainnet `ritualID` is required – this connects the encrypt/decrypt API to a cohort of independently operated nodes and corresponds to a DKG public key generated by independent parties. A dedicated `ritualID` for Turbo + TACo projects will be sponsored soon. Watch for updates here or in the Discord [#taco](https://discord.com/channels/866378471868727316/870383642751430666) channel.&#x20;
