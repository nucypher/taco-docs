---
description: >-
  The TACo SDK allows you to use threshold encryption & decryption in your
  application.
---

# Quickstart (Testnet)

In just a few minutes you will able to:

* **Define decryption conditions** – these are predefined rules or criteria that must be fulfilled before the encrypted data can be decrypted.
* **Encrypt data & assign decryption conditions** – when you encrypt data, you not only secure it but also tie the decryption process to the conditions you defined.
* **Threshold-decrypt data** – once the decryption conditions are met and validated by a threshold of TACo nodes, decryption can occur.

### 1. Installation

Install `taco` , `taco-auth`, and `ethers` with your favorite package manager:

<pre class="language-bash"><code class="lang-bash"><strong>$ npm install @nucypher/taco @nucypher/taco-auth ethers@5.7.2
</strong></code></pre>

### 2. Configuration

To run the code examples below, you will need the `ritualId` encryption parameter. **In production**, your wallet address (encryptor) will also have to be allow-listed for this specific ritual. Please reach out to us [here](https://discord.com/channels/866378471868727316/870383642751430666) to receive a `ritualId` and allow-list access.\
\
Additionally, we have [publicly available testnet rituals](for-developers/taco-integration/get-started-with-tac.md#testnet-configuration) for use when developing your apps.

### 3. Define decryption condition and encrypt data

With `ritualId` and [a web3 provider from `ethers`](https://docs.ethers.org/v5/api/providers/#providers-getDefaultProvider), we can `taco.encrypt` our data.&#x20;

In this example, we will use our [`tapir` testnet](get-started-with-tac.md#testnet-configuration), where you can freely use `ritualId = 6`; A read-only connection to Polygon Amoy is required due to DKG Coordination contracts being stored there. The `signerProvider` is required to [authenticate](for-developers/references/authentication/) the Encryptor.

<pre class="language-typescript"><code class="lang-typescript">import { initialize, encrypt, conditions, domains } from '@nucypher/taco';
import { ethers } from "ethers";

// We have to initialize the TACo library first
await initialize();

// Define decryption condition
const ownsNFT = new conditions.predefined.erc721.ERC721Ownership({
  contractAddress: '0x1e988ba4692e52Bc50b375bcC8585b95c48AaD77',
  parameters: [3591],
  chain: 11155111,  // sepolia
});

const signerProvider = new ethers.providers.Web3Provider(window.ethereum);
const polygonProvider = new ethers.providers.JsonRpcProvider("https://polygon-amoy.drpc.org");

const message = "my secret message";
const ritualId = 6

// encrypt data
const messageKit = await encrypt(
<strong>  polygonProvider,
</strong><strong>  domains.TESTNET,
</strong>  message,
  ownsNFT,
  ritualId,
  signerProvider.getSigner() 
);
</code></pre>

### 4. Decrypt the data

Now we just have to pass the `messageKit` to the intended _data consumer_:

<pre class="language-typescript"><code class="lang-typescript"><strong>import { conditions, decrypt, domains, initialize,  } from '@nucypher/taco';
</strong>import { EIP4361AuthProvider, USER_ADDRESS_PARAM_DEFAULT } from '@nucypher/taco-auth';
import { ethers } from "ethers";

// We have to initialize the TACo library first
await initialize();

const web3Provider = new ethers.providers.Web3Provider(window.ethereum); 

const conditionContext =
  conditions.context.ConditionContext.fromMessageKit(messageKit);
  
// auth provider when condition contains ":userAddress" context variable
// the decryptor user must provide a signature to prove ownership of their wallet address
const authProvider = new EIP4361AuthProvider(
  web3Provider,
  web3Provider.getSigner(),
);
conditionContext.addAuthProvider(USER_ADDRESS_PARAM_DEFAULT, authProvider);

const decryptedMessage = await decrypt(
  web3Provider,
  domains.TESTNET,
  messageKit,
  conditionContext,
);
</code></pre>

Since `ownsNFT` condition refers to an NFT owned by the _data consumer_, `decrypt` call will prompt the recipient to sign a message and prove the ownership of the caller's wallet.

### Next steps

Learn more about using TACo in a sandboxed environment in the [Testnets](for-developers/taco-integration/get-started-with-tac.md) section.

### Example applications

The following samples showcase integrations with React-based web apps, and serve as an 'end-to-end' reference for creating conditions-based encryption & decryption:

* [`taco-web/demos`](https://github.com/nucypher/taco-web/tree/main/demos)
* [`taco-web/examples/taco`](https://github.com/nucypher/taco-web/tree/main/examples/taco)
