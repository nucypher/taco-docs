# Encryptor Allowlist

## Introduction

The _Encryptor Allowlist_ is a simple access control mechanism that only allows specific data producers, or encryptors, to access a given DKG ritual, cohort of TACo nodes, and associated public key. In practice, it means that developers can limit who can use TACo to encrypt the data using the persistent public key.

The entity that can authorize encryptors is called the `authority`. Each ritual has an `authority` which corresponds to the address of the wallet that initiated the ritual.

## Allowlist&#x20;

In TACo's genesis era, the process of initializing the ritual and managing the `allowlist` is simplified to the following steps:

1. User shares their `authority` address with the testnet operator.
2. The testnet operator initiates the ritual for the user, marking them as the `authority` for that ritual.
3. Now, the user can manage their encryptor allowlist using their `authority`. So the user adds a new encrypter to the allowlist.
4. The user can now use the encryptor in `taco` to encrypt their data.

## Allowlist on testnet

In the early era, it is possible to use one of the premade rituals on the testnet without any extra steps outlined in the setup above. You can simply configure `taco` to use one of those rituals and use any encryptor (wallet address) to perform the encryption.

The current ritual configurations are located in the [Testnet integration ](../../get-started-with-tac.md#testnet-configuration)section.&#x20;
