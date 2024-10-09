# Authentication

Some dynamic access conditions require specific information about the decryptor, which needs to be verified/authenticated, such as wallet address or other identity-related information. This verification needs to be done in a way that doesn't allow the decryptor to simply provide _**ANY**_ value. Instead, the decryptor should provide proof that can be verified so that the validity of the value can be confirmed and the value subsequently used for properly evaluating access.

In the case of a wallet address, the decryptor must sign a message with the private key corresponding to the public wallet address. This signature serves as proof of ownership, which nodes can then verify before using the corresponding wallet address for decryption condition evaluation. Otherwise, the decryptor could specify a wallet address they do not own but still satisfy the required condition, e.g. pretend to own `vitalik.eth` to satisfy an ETH balance condition.

`AuthProvider` is an abstraction provided by `@nucypher/taco-auth` that plays a critical role in generating the necessary proof for authenticating information about the decryptor. This proof is then validated as a part of condition evaluation during the decryption process. Instead of directly providing the necessary information (e.g., wallet address), which could be falsified, the decryptor uses an `AuthProvider` to generate the requisite proof.&#x20;

At the moment,  [Sign-In With Ethereum (SIWE)](https://docs.login.xyz/general-information/siwe-overview/eip-4361) is supported, with more authentication protocols expected to be added in the future.

For more information on specific authentication providers and how they can be utilized alongside access conditions, see [Condition Context](conditioncontext-and-context-variables.md).

