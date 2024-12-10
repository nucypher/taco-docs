# Waku

This guide explains how to leverage [Waku](https://docs.waku.org/) and TACo in combination, providing developers with a general-purpose, conditions-based, end-to-end encrypted and censorship-resistance messaging layer. This integration enables a variety of high-performance decentralized applications, including the location-sharing app [Arc](https://github.com/nucypher/Arc), a reference prototype and repo accompanying this tutorial.&#x20;

This tutorial helps set up a minimum viable integration, which includes setting up relevant services, encrypting and sending messages with embedded, NFT-based access conditions, and receiving and decrypting said messages.&#x20;

## Waku overview

Waku's protocols provide decentralized message transport and routing, efficiently supporting in-browser light clients, and offering protocol-level metadata privacy. Ephemeral, real-time messaging with minimal persistence ensures quick and secure communication within adopting applications. Like TACo, Waku does not make compromises with respect to permissionlessness, trust-minimization and decentralization. For more on Waku's JavaScript SDK, check out their [documen](https://docs.waku.org/guides/js-waku/)[tation](https://docs.waku.org/guides/js-waku/).

***

## Integration steps&#x20;

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

Note that _content topics_ are metadata strings embedded into outgoing messages that facilitate protocol-level features like selectively processing incoming messages. These strings can be thought of as 'channels', which determine, among other things, the path of messages through the network. Learn more about content topics [here](https://docs.waku.org/learn/concepts/content-topics/).&#x20;

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
const ritualId = 0 // Replace with your actual ritual ID
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
