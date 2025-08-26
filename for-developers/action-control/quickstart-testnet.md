# Quickstart (Testnet)

## Quickstart Guide (Testnet)

The TACo SDK allows you to implement conditional signing for ERC-4337 UserOperations. In just a few minutes, you'll be able to:

* **Define signing conditions** – Specify rules that must be fulfilled before UserOperations can be signed
* **Request signatures** – Get signatures from the decentralized network when conditions are met
* **Execute signed UserOperations** – Use the aggregated signature to execute transactions via Account Abstraction

### 1. Installation

Install `@nucypher/taco`, `@nucypher/taco-auth`, and `ethers` with your favorite package manager:

```bash
npm install @nucypher/taco @nucypher/taco-auth ethers@5.7.2
```

### 2. Configuration

To run the code examples below, you will need a `cohortId` signing parameter. For development and testing, we provide publicly available testnet cohorts:

| Network | Domain    | Cohort ID | Chain ID | Description           |
| ------- | --------- | --------- | -------- | --------------------- |
| Sepolia | `TESTNET` | 1         | 11155111 | Public testnet cohort |
| Lynx    | `DEVNET`  | 1         | 11155111 | Development cohort    |

For production use, please [contact us](https://docs.taco.build/contact) to have a dedicated cohort configured.

### 3. Define Signing Conditions (Admin Only)

Only the cohort admin can set conditions. The testnet cohort comes pre-configured with a time-based condition that allows signing after a certain timestamp.

Admins can set conditions on the contract easily:

```typescript
import { conditions, domains, setSigningCohortConditions } from '@nucypher/taco';
import { ethers } from 'ethers';

// Connect with admin wallet (only admin can set conditions)
const signerProvider = new ethers.providers.Web3Provider(window.ethereum);

// Example: Define a time-based condition
const timeCondition = new conditions.base.time.TimeCondition({
  returnValueTest: {
    comparator: '>',
    value: Math.floor(Date.now() / 1000), // Current timestamp in seconds
  },
});

// Set conditions for your cohort (requires admin privileges)
await setSigningCohortConditions(
  signerProvider,
  domains.TESTNET,
  timeCondition,
  cohortId, // your cohort ID
  chainId,
  signerProvider.getSigner()
);
```

### 4. Request Signatures

Once conditions are set, you can request signatures for UserOperations that meet those conditions:

```typescript
import { conditions, domains, signUserOp, initialize } from '@nucypher/taco';
import { UserOperation } from '@nucypher/shared';
import { ethers } from 'ethers';

// Initialize TACo library (required once per application)
await initialize();

// Set up provider
const ethProvider = new ethers.providers.JsonRpcProvider('https://ethereum-sepolia-rpc.publicnode.com');

const userOp: UserOperation = {
  sender: '0x...',
  nonce: 0,
  callData: '0x...',
  callGasLimit: 50000,
  // ... other UserOperation fields
};

const conditionContext = await conditions.context.ConditionContext.forSigningCohort(
  ethProvider,
  domains.TESTNET,
  1, // cohortId
  11155111 // chainId
);

const aaVersion = '0.8.0';

const signResult = await signUserOp(
  ethProvider,
  domains.TESTNET,
  1, // cohortId
  11155111, // chainId
  userOp,
  aaVersion,
  conditionContext,
);

console.log('Message Hash:', signResult.messageHash);
console.log('Aggregated Signature:', signResult.aggregatedSignature);
```

### 5. Execute with Signature

The aggregated signature can now be used to execute the UserOperation through an ERC-4337 bundler:

```typescript
// Add the signature to your UserOperation
userOp.signature = signResult.aggregatedSignature;

// Submit to your ERC-4337 bundler
// The bundler will validate the signature against the multisig contract
const bundlerResponse = await bundlerClient.sendUserOperation(userOp);

// The multisig contract's isValidSig method is called automatically
// by the EntryPoint contract during UserOperation validation
```

The signature validation happens automatically when the bundler submits the UserOperation to the EntryPoint contract. The EntryPoint calls the account's `validateUserOp` function, which in turn verifies the signature using the multisig contract's `isValidSig` method.

### Next Steps

* **Advanced Conditions**: See the [Signing Object Conditions Guide](../taco-sdk/references/conditions/signing-object-conditions.md) for more complex condition examples
* **Production Access**: [Contact us](https://docs.taco.build/contact) for a dedicated production cohort
* **Examples**: Check out [example implementations](https://github.com/nucypher/taco-web/tree/main/examples)

For additional support, join our [Discord community](https://discord.gg/taco)
