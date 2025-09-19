# Encryptor Allowlist

## Introduction

The _Encryptor Allowlist_ is a simple access control mechanism that only allows specific data producers, or encryptors, to access a given DKG ritual, cohort of TACo nodes, and associated public key. In practice, it means that developers can limit who can use TACo to produce valid encryptions for the public key.

The entity that can authorize encryptors is called the `authority`. Each ritual has an `authority` which corresponds to the address of the wallet that initiated the ritual.

Note that encryptor authorization is enforced at decryption time, which means that ciphertexts produced by unauthorized encryptors will not be decryptable and TACo client will return an error.

The restriction is not enforced at encryption time for several reasons:

* Due to the inherent nature of public key cryptography, anyone can use a public key to encrypt data and generate a ciphertext.
* There's valid scenarios where encryption may happen before authorization.

For these reasons, no error will occur at encryption time when using a wallet that is not authorized (i.e. it's not in the _Encryptor Allowlist_), and authorization enforced is deferred until decryption.

## Managing the Allowlist on Testnet

On [**testnet**](../../get-started-with-tac.md) it is possible to use one of the premade rituals without any extra steps since there is no allowlist implemented on them. You can simply configure `taco` to use one of those rituals and use any encryptor (wallet address) to perform the encryption.

If you require to test allowlist management flows, don't hesitate [to reach out to us](https://discord.com/invite/buildwithtaco).

## Managing the Allowlist on Mainnet

If you wish to use TACo in production for your application, you will need to establish your own ritual. Please, [contact us](https://discord.com/invite/buildwithtaco) for further assistance.

The _Encryptor Allowlist_ is managed through the _AccessController_ contract associated with the ritual.

Only the ritual authority wallet can manage the _Encryptor Allowlist_ of the _ritual_. This `authority` was set during the Ritual initialization.

1. The contract address of the ritual's AccessController can be found calling the `getAccessController()` function on the [Coordinator contract](../../../reference/contract-addresses.md#contracts-on-polygon-mainnet-l2), specifying the `ritual ID`.
2. The ritual's `authority` wallet can call the `authorize()` function on the `AccessController` contract, specifying the `ritual ID` and a list of addresses to be included in the allowlist.
3. From now on, these encryptor wallets can be used for encrypting in this ritual.

Likewise, the `deauthorize()` function of the AccessController can be called by the authority to remove encryptor addresses from the allowlist.
