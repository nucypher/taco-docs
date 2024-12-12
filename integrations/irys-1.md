# Turbo

This guide explains how to combine TACo with [Turbo](https://docs.ardrive.io/docs/turbo/what-is-turbo.html) – a bundler SDK which facilitates uploading, retrieving, indexing and paying for data hosted on [Arweave](https://search.brave.com/search?q=arweave+docs\&source=desktop). This integration enables developers to offer their end-users permanent data storage that is private by default – ensuring that sensitive payloads will only be acessible to legitimate and qualifying consumers, and will be accessible in perpetuity.&#x20;

## Turbo overview

WIP

## Use case ideas

* **Governance.** Generate tamper-proof, timestamped records of voting activity, enhancing transparency and reducing trust assumptions.&#x20;
* **Connected Vehicles.** Store sensitive real-time vehicle diagnostics and geolocation data, such that the data stream is instantly available when required (e.g. while driving) but not leaked beyond known and legitimate recipients (e.g. a smart city traffic system).&#x20;
* **Private NFTs.** Move beyond the status quo of symbolic receipts stored on centralized platforms, to a world where one owns the _decryption rights_ to a movie, track, in-game asset, or piece of art – trust-mimimized and in perpetuity.&#x20;

***

## Integration steps&#x20;

### 1. Install TACo & Turbo&#x20;

```typescript
yarn add @nucypher/taco
yarn add @ardrive/turbo-sdk
yarn add arbundles
```

### 2. Define access condition & encrypt the data&#x20;

Next, initialize the `taco-web` [library](https://github.com/nucypher/taco-web). This provides, among other functionality, the facility to create access conditions and embed them in the encrypted data payload. In this example, the basis for decryption is time-based – which requires reading the block height of the Arweave chain. Decryption can only occur once a certain block height has been reached. More on condition types [here](../conditions/).\
\
Encrypt the message using the `blockHeight` condition, specifying the aforementioned devnet `domain` and `ritualID`, and a standard web3 provider/signer.&#x20;

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

{% hint style="warning" %}
This guide utilizes the parameters `ritualId = 27` & `domains.DEVNET`. These refer to an open DKG public key and hacker-facing bleeding-edge testnet respectively, which  are **not decentralized** and unsuitable for real-world sensitive data. For more information, see the [trust assumptions section](../trust-assumptions/).&#x20;
{% endhint %}

### 3. Connect to Turbo & store the data&#x20;

&#x20;Next, connect to a Turbo signer.&#x20;

{% hint style="info" %}
In this example the payload size is lower than 100kb, so it is currently free to upload to Arweave via Turbo.&#x20;
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

Next, use the `encryptedMessageId` to find and retrieve the encrypted payload via any Arweave gateway.

The decryption logic enables a decryption request to be sent to the TACo nodes enforcing access control, for decryption fragments to be retrieved, assembled locally, and finally to decrypt the payload. These steps are contained in the `decrypt()` function below.&#x20;

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
