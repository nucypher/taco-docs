# Any (Major) EVM Chain Condition Support

As the adoption of TACo continues to grow, one challenge that arises is supporting an increasing number of blockchains. Currently, nodes are required to configure a dedicated RPC URL for each blockchain supported for conditions. For example, regardless of the TACo domain (`mainnet`or testnet), nodes must configure an RPC URL for Ethereum (Mainnet/Sepolia). and Polygon (Mainnet/Amoy). Those chains would represent the available chains that can be used for on-chain conditions.

While this configuration ensures reliability, it becomes cumbersome as more blockchains are added to TACo's ecosystem for conditions. Each new blockchain requires nodes to manually set up and manage its RPC URL, increasing the administrative burden over time.

Currently, TACo nodes employ a fallback mechanism to enhance reliability by utilizing a list of public RPC URLs. In this setup, the configured RPC URL is given priority, but if it becomes temporarily unavailable, the node automatically switches to one of the fallback public RPC URLs. For instance, on mainnet, these backup URLs are drawn from the [`mainnet.json` file in the `nucypher/chainlist` repository](https://github.com/nucypher/chainlist/blob/main/mainnet.json). The list of supported chains on `lynx` is available in the [`lynx.json`file in the `nucypher/chainlist`repository](https://github.com/nucypher/chainlist/blob/main/lynx.json).

Rather than requiring nodes to configure specific RPC URLs for each supported blockchain for conditions, we are experimenting with maintaining a curated list of public RPC endpoints for most EVM-compatible chains. This approach allows nodes to utilize these pre-defined public RPC URLs, removing the need for dedicated configurations for every blockchain. Since most chains offer multiple public RPC URLs, the list provides built-in redundancy, ensuring reliability during condition verification.

&#x20;This comprehensive list enables rapid prototyping for dApps across multiple blockchains addressing a growing demand from developers.
