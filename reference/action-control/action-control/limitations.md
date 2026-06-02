# Limitations

TACo Action Control product represents cutting-edge innovation in decentralized conditional signing. As an early-stage technology pushing the boundaries of what's possible with distributed systems and cryptography, there are some current limitations to be aware of as the platform continues to evolve.

We're actively developing new capabilities and optimizations. These limitations represent the current state of the technology, not its final form.

### Current Limitations

#### Transaction Simulation

Conditions are currently evaluated based on transaction parameters rather than simulation outcomes. This means conditions can check values like gas limits, addresses, and amounts, but cannot validate what would happen if the transaction were executed. Forthcoming versions will incorporate simulation-based validation for more sophisticated condition logic.

#### On-Chain Operations

Setting and updating conditions requires on-chain transactions, which incur gas costs. While this ensures transparency and immutability of signing rules, it means condition updates cannot be instant or free.

#### Condition Complexity

While the system supports sophisticated conditional logic through compound conditions, extremely complex nested conditions may impact performance. We recommend keeping condition structures as simple as possible while meeting your security requirements.

### Looking Forward

These limitations are actively being addressed by our development team. We're committed to building the most advanced conditional signing infrastructure available.

If you'd like to know more, or would like to share feedback/ideas, please come and say hello in our [Discord](http://discord.gg/buildwithtaco).
