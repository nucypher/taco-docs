# Waku

This guide explains how to integrate [Waku](https://docs.waku.org/) and TACo, which together offer developers a comprehensive _Web3 privacy development toolkit –_ i.e. a general-purpose, censorship-resistant, end-to-end encrypted messaging layer where the access logic for delivered messages is programmable and conditions-based. This enables a variety of performant decentralized applications – including the location-sharing app [Arc](https://lionfish-app-rtgbk.ondigitalocean.app/), a reference prototype and [repository](https://github.com/nucypher/Arc) that accompanies this tutorial.&#x20;

This guide sets up a minimum viable integration of the two SDKs/protocols, that can be used across use cases and domains. This includes setting up relevant services and dependencies, encrypting/sending messages with NFT-based access conditions, and receiving/decrypting said messages.&#x20;

## Waku overview

Waku's communication protocols provide gas-free message transport and routing – supporting in-browser light clients, and ensuring protocol-level metadata privacy and end-user anonymity. Ephemeral, real-time messaging with minimal persistence brings efficient data/message transmission within adopting applications. Like TACo, Waku's underlying network makes no compromises with respect to permissionlessness, trust-minimization and decentralization.&#x20;

For more on Waku's JavaScript SDK, check out their [documen](https://docs.waku.org/guides/js-waku/)[tation](https://docs.waku.org/guides/js-waku/).

## Use case ideas

* **Crowdsourcing.** Whether it's weather, flight tracking, large geospatical models, traffic data, or even verifying the ingredients used by restaurants, contributors in large-scale crowdsourcing projects are often taken for granted – both in terms of their privacy and their share of the spoils. Leverage Waku's anonymity-preserving data stream transmission, and TACo's payment-conditioned access, to ensure contributors to highly valuable data sets are able to maintain their anonymity while guaranteeing their compensation.&#x20;
* **Medical Wearables.** Don't rely on HIPAA or the moral compass of medical device manufacturers. Waku's the ideal transport layer for devices which stream small data payloads, while TACo enables strict sharing requirements via composable condition logic. Health data applications which integrate Waku & TACo can uniquely offer their end-users total sovereignty over their sensitive data, where they alone control flows of information to their healthcare providers.&#x20;
* **GenAI Inference.** Messages to and from an LLM model/chatbot should be 100% private and censorship-resistant, not harvested by an intermediary or blocked by a central authority. Harness Waku's efficient, ephemeral message transmission, alongside TACo's per-message/ciphertext encryption granularity, to provide a smooth and private experience for users of GenAI interfaces. Both TACo and Waku's decentralized/P2P frameworks ensure messages (and decryption material) can propagate without any single entity being able to unilaterally censor interactions with the model.&#x20;

***

## Example application & repo

[Arc](https://github.com/nucypher/Arc) is an experimental, privacy-preserving, permissionless, decentralized location-sharing application that combines Waku & TACo's capabilites. Arc provides secure, end-to-end encrypted location sharing with fine-grained, on-chain access control, making it a powerful tool for individuals, groups or applications seeking a way to selectively share their location without trusting an intermediary platform.&#x20;

In this version of Arc, users may set on-chain conditions for accessing their real-time location, such as a specific time window for access (based on block time), a minimum token balance, or the ownership of a special-purpose NFT.&#x20;

***

## Integration steps&#x20;

This section walks through a _minimum viable integration_ to get developers started. For more powerful extensions and advanced condition logic, check out the [Access Control ](../conditions/wip-feature-requests/)section.&#x20;

### 1. Required imports

```javascript
// Waku SDK components for decentralized messaging
import { createLightNode, createEncoder, createDecoder, waitForRemotePeer } from '@waku/sdk'

// TACo components for conditional encryption
import { initialize, encrypt, decrypt, conditions, domains, ThresholdMessageKit } from '@nucypher/taco'

// Authentication provider for TACo
import { EIP4361AuthProvider, USER_ADDRESS_PARAM_DEFAULT } from '@nucypher/taco-auth'

// Ethereum interaction library
import { ethers } from 'ethers'
```

### 2. Define a Waku encoder

```javascript
// Define your application's content topic
// Format: /<application-name>/<version>/<content-type>/<encoding>
const CONTENT_TOPIC = '/taco-waku-example/1/messages/proto'
const encoder = createEncoder({ contentTopic: CONTENT_TOPIC })
const decoder = createDecoder(CONTENT_TOPIC)
```

{% hint style="info" %}
Note that _content topics_ are metadata strings embedded into outgoing messages that facilitate protocol-level features like selectively processing incoming messages. These strings can be thought of as 'channels', which determine, among other things, the path of messages through the network. Learn more about content topics [here](https://docs.waku.org/learn/concepts/content-topics/).&#x20;
{% endhint %}

### 3. Set up Web3 provider

Configure the Ethereum provider for on-chain interactions:

```javascript
const web3Provider = new ethers.providers.Web3Provider(window.ethereum)
await web3Provider.send('eth_requestAccounts', [])
```

### 4. Initialize Waku & TACo services&#x20;

```javascript
// Initialize TACo
await initialize()

// Create and start Waku node
const node = await createLightNode({ defaultBootstrap: true })
await node.start()
await waitForRemotePeer(node)
```

### **5. Define access conditions**

Next, define the the conditions that must be satisfied for messages to be decrypted into plaintext. In this example, a qualifying _data consumer_ must hold a ERC721, in a wallet they provably own at the time of decryption request:&#x20;

```javascript
const condition = new conditions.predefined.erc721.ERC721Ownership({
  contractAddress: '0x1e988ba4692e52Bc50b375bcC8585b95c48AaD77',
  parameters: [3591],  // Token ID
  chain: 11155111,     // Sepolia testnet
})
```

### 6. Encrypt and transmit messages

In this step, set up the plaintext message to be encrypted locally, with the conditions for access embedded. Once encrypted, the message can be transmitted across the Waku network:&#x20;

```javascript
const message = 'Hello, encrypted Waku world!'
const ritualId = 6  // This is a testnet-specific ritual ID

// Encrypt message with TACo
const messageKit = await encrypt(
  web3Provider,
  domains.TESTNET,
  message,
  condition,
  ritualId,
  web3Provider.getSigner()
)

// Send via Waku
await node.lightPush.send(encoder, {
  payload: messageKit.toBytes()
})
```

{% hint style="warning" %}
Note that the `ritualID` refers to the group of TACo nodes that will enforce access to the message(s). This example uses `ritualId = 6` and `domains.TESTNET`. These development environments are **not decentralized** and unsuitable for real-world sensitive data.
{% endhint %}

### 7. Receive and decrypt messages

Finally, set up the facility for receiving the encrypted message payload. Once the message is retrieved, the (authenticated) _data consumer_ can present the payload to the TACo network and receive decryption material if qualifying – all via the following decryption logic:&#x20;

```javascript
await node.filter.subscribe([decoder], async (wakuMessage) => {
  if (!wakuMessage.payload) return
  
  try {
    // Convert received bytes to MessageKit
    const receivedMessageKit = ThresholdMessageKit.fromBytes(wakuMessage.payload)
    
    // Set up condition context and authentication
    const conditionContext = conditions.context.ConditionContext.fromMessageKit(receivedMessageKit)
    const authProvider = new EIP4361AuthProvider(web3Provider, web3Provider.getSigner())
    conditionContext.addAuthProvider(USER_ADDRESS_PARAM_DEFAULT, authProvider)
    
    // Decrypt message
    const decryptedMessage = await decrypt(
      web3Provider,
      domains.TESTNET,
      receivedMessageKit,
      conditionContext
    )
    
    console.log('Decrypted message:', decryptedMessage)
  } catch (error) {
    console.error('Error decrypting message:', error)
  }
})
```

### Complete integration example

```javascript
import { createLightNode, createEncoder, createDecoder, waitForRemotePeer } from '@waku/sdk'
import { initialize, encrypt, decrypt, conditions, domains, ThresholdMessageKit } from '@nucypher/taco'
import { EIP4361AuthProvider, USER_ADDRESS_PARAM_DEFAULT } from '@nucypher/taco-auth'
import { ethers } from 'ethers'

const CONTENT_TOPIC = '/taco-waku-example/1/messages/proto'

// Initialize TACo and Waku
await initialize()
const node = await createLightNode({ defaultBootstrap: true })
await node.start()
await waitForRemotePeer(node)

// Setup Web3 provider
const web3Provider = new ethers.providers.Web3Provider(window.ethereum)
await web3Provider.send('eth_requestAccounts', [])

// Define decryption condition
const condition = new conditions.predefined.erc721.ERC721Ownership({
  contractAddress: '0x1e988ba4692e52Bc50b375bcC8585b95c48AaD77',
  parameters: [3591],
  chain: 11155111, // Sepolia testnet
})

// Encrypt and send message
const encoder = createEncoder({ contentTopic: CONTENT_TOPIC })
const message = 'Hello, encrypted Waku world!'
const ritualId = 6 // Replace with your actual ritual ID
const messageKit = await encrypt(
  web3Provider,
  domains.TESTNET,
  message,
  condition,
  ritualId,
  web3Provider.getSigner()
)
await node.lightPush.send(encoder, {
  payload: messageKit.toBytes()
})
console.log('Encrypted message sent over Waku')

// Set up message reception and decryption
const decoder = createDecoder(CONTENT_TOPIC)
await node.filter.subscribe([decoder], async (wakuMessage) => {
  if (!wakuMessage.payload) return
  try {
    const receivedMessageKit = ThresholdMessageKit.fromBytes(wakuMessage.payload)
    const conditionContext = conditions.context.ConditionContext.fromMessageKit(receivedMessageKit)
    const authProvider = new EIP4361AuthProvider(web3Provider, web3Provider.getSigner())
    conditionContext.addAuthProvider(USER_ADDRESS_PARAM_DEFAULT, authProvider)
    const decryptedMessage = await decrypt(
      web3Provider,
      domains.TESTNET,
      receivedMessageKit,
      conditionContext
    )
    console.log('Decrypted message:', decryptedMessage)
  } catch (error) {
    console.error('Error decrypting message:', error)
  }
})
```

***

## Using Waku & TACo in production&#x20;

The parameters specified in this guide are for testing and hacking only. For real-world use cases, the production version of TACo is required – i.e. a funded Mainnet `ritualID` which connects the encrypt/decrypt API to a cohort of independently operated nodes, and corresponds to a DKG public key generated by independent parties.&#x20;

A dedicated `ritualID` for Waku + TACo integrations will be sponsored in early 2025. Watch for updates here or in the[ #taco](https://discord.com/channels/866378471868727316/870383642751430666) channel on the Threshold Network discord server.&#x20;
