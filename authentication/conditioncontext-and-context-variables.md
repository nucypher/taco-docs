# Condition Context

A `ConditionContext` is a container for dynamic values to be used in conditions at decryption time.  A `ConditionContext` contains `ContextVariables` and their corresponding values.

## Context Variables

Context variables can be recognized by the `:` prefix. They act as placeholders within conditions to be specified at the encryption time, whose value is provided at decryption time.&#x20;

Context variables are broken up into two groups:

*   _**Reserved context variables**_**:** these have special functionality within `taco`.\
    \
    For now, there are only two such reserved context variables:

    * [`:userAddress`](conditioncontext-and-context-variables.md#useraddress)
    * [`:userAddressExternalEIP4361`](conditioncontext-and-context-variables.md#useraddressexternaleip4361)

    which require the use of [Authentication Providers](./) to provide verifiable proof of values specified by the _data consumer_. Applications should be cognizant of which reserved context variable they use based on their needs.
* _**Custom context variables**_**:**  these are application-specific, simple key-value pairs where the _data consumer_ can directly specify the values without any verification.

### `:userAddress`

Whenever the `:userAddress` context variable is present in a decryption condition, the _data consumer_ must use the `EIP4361AuthProvider`. This authentication provider will prompt the user to sign a  `EIP-4361` ([Sign-in With Ethereum](https://docs.login.xyz/general-information/siwe-overview/eip-4361)) message to authenticate the _data consumer's_ wallet address at decryption time. This signature is provided to nodes to prove wallet address ownership when evaluating the decryption condition.

```typescript
import { conditions } from '@nucypher/taco';
import { EIP4361AuthProvider, USER_ADDRESS_PARAM_DEFAULT } from '@nucypher/taco-auth';

const conditionContext = conditions.context.ConditionContext.fromMessageKit(messageKit);
  
const web3Provider = new ethers.providers.Web3Provider(window.ethereum);
  
const authProvider = new EIP4361AuthProvider(
  web3Provider,
  web3Provider.getSigner(),
);

// set auth provider for ":userAddress"
conditionContext.addAuthProvider(USER_ADDRESS_PARAM_DEFAULT, authProvider);
```

{% hint style="info" %}
To negate the need for repeated wallet signatures for every decryption request by the same _data consumer_, the corresponding proof that is generated is cached until an expiry is triggered, after which the _data consumer_ will be prompted again.&#x20;
{% endhint %}

### `:userAddressExternalEIP4361`

The `:userAddressExternalEIP4361` context variable in conditions requires the use of the `SingleSignOnEIP4361AuthProvider` for decryption.&#x20;

The `SingleSignOnEIP4361AuthProvider` integrates _data consumer_ wallet authentication into a broader Sign-in With Ethereum (SIWE) identity management system already used by an application, allowing users to authenticate once with the application and re-use that authentication with TACo.

Therefore, the existing application-specific SIWE sign-in message and signature can be reused with TACo and provide a seamless user experience during TACo decryption without the need to sign multiple messages.

```typescript
import { conditions } from '@nucypher/taco';
import { 
  SingleSignOnEIP4361AuthProvider,
  USER_ADDRESS_PARAM_EXTERNAL_EIP4361,
} from '@nucypher/taco-auth';

const conditionContext = conditions.context.ConditionContext.fromMessageKit(messageKit);
  
const {messageStr, signature} = ...;  // existing SIWE information from application  

const authProvider = SingleSignOnEIP4361AuthProvider.fromExistingSiweInfo(
  messageStr,
  signature,
);

// set auth provider for ":userAddressExternalEIP4361"
conditionContext.addAuthProvider(USER_ADDRESS_PARAM_EXTERNAL_EIP4361, authProvider);
```

{% hint style="warning" %}
TACo requires that Sign-In With Ethereum (SIWE) messages be issued within the last 2 hours based on the "Issued At" timestamp. For single sign-on usage, the application should refresh the user's cached SIWE login accordingly.
{% endhint %}

## Illustrative Example

Let's take a look at this `ContractConditon` example:

```typescript
import { conditions, domains, encrypt } from '@nucypher/taco';

const ownsNFTRaw = new conditions.contract.ContractCondition({
  method: 'balanceOf',
  parameters: [':userAddress'], // <- A context variable
  standardContractType: 'ERC721',
  contractAddress: '0x1e988ba4692e52Bc50b375bcC8585b95c48AaD77',
  chain: 1,
  returnValueTest: {
    comparator: '>',
    value: ':selectedBalance', // <- Another context variable
  },
});

const message = ...  // message to encrypt

const messageKit = await encrypt(
  web3Provider,
  domains.MAINNET,
  message,
  ownsNFTRaw,
  ritualId,
  web3Provider.getSigner(),
);
```

In this example, we can see two different context variables

* `:userAddress` - A reserved context variable
* `:selectedBalance` - A custom context variable

To replace the `:userAddress` context variable with an actual wallet address during decryption, TACo needs to be provided with an `AuthProvider` for the user to sign an `EIP4361` ([Sign-in With Ethereum](https://docs.login.xyz/general-information/siwe-overview/eip-4361)) message to confirm wallet ownership at the decryption time.

Additionally, the `:selectedBalance` custom context variable has to be provided to the `decrypt` function by the _data consumer_.

Both context variables need to be provided to a `ConditionContext` which is then used by the `decrypt` function.

```typescript
import { decrypt, conditions } from '@nucypher/taco';
import { EIP4361AuthProvider, USER_ADDRESS_PARAM_DEFAULT } from '@nucypher/taco-auth';

const web3Provider = new ethers.providers.Web3Provider(window.ethereum);

const conditionContext =
  conditions.context.ConditionContext.fromMessageKit(messageKit);
  
// satisfy ":userAddress"
const authProvider = new EIP4361AuthProvider(
  web3Provider,
  web3Provider.getSigner(),
);
conditionContext.addAuthProvider(USER_ADDRESS_PARAM_DEFAULT, authProvider);

// satisfy ":selectedBalance"; simple key-value pair
const customParameters: Record<string, conditions.CustomContextParam> = {
  ':selectedBalance': 2,
};
conditionContext.addCustomContextParameterValues(customParameters);

const decryptedMessage = await decrypt(
  web3Provider,
  domains.MAINNET,
  messageKit,
  conditionContext,
);
```

With those context parameters, the condition will be evaluated by nodes at decryption time to be:

```typescript
{
  method: 'balanceOf',
  parameters: ['0x...'], // The address of decryptor's wallet (verified via signature)
  standardContractType: 'ERC721',
  contractAddress: '0x1e988ba4692e52Bc50b375bcC8585b95c48AaD77',
  chain: 1,
  returnValueTest: {
    comparator: '>',
    value: 2, // Concrete value to be used in comparison
  },
}
```

{% hint style="info" %}
This is a contrived example. \
\
For **custom context variables** specifically, time should be taken to think through the use case since the _data consumer_ provides these and can be set to any value. In this case, the `selectedBalance`  value can simply be set to `-1` by the requester, which would grant them access without their owning any NFT.\
\
Custom context variables, such as providing a Merkle tree root as a parameter to a contract function, are appropriate, but such an example would be overly complex.
{% endhint %}

## Checking for required context variables

If your application utilizes many different conditions each with different context variables, the required context variables for decryption can be queried.&#x20;

The `requestedContextParameters` property of the `ConditionContext` object can be used to identify the necessary context variables for decryption. By querying this property, the application can understand the context variables required for the relevant condition.

In this way, the application doesn't need prior knowledge of the condition used but can dynamically determine the context variables needed based on the specific condition being handled.

```typescript
// create condition context
const conditionContext = conditions.context.ConditionContext.fromMessageKit(messageKit);

// illustrative optional example of checking what context parameters are required
if (
  conditionContext.requestedContextParameters.has(USER_ADDRESS_PARAM_DEFAULT)
) {
  // add authentication for ":userAddress" in condition
  const authProvider = new EIP4361AuthProvider(
    provider,
    provider.getSigner()
  );
  conditionContext.addAuthProvider(USER_ADDRESS_PARAM_DEFAULT, authProvider);
}
if (
  conditionContext.requestedContextParameters.has(":selectedBalance")
) {  
  // satisfy ":selectedBalance"; simple key-value pair
  const customParameters: Record<string, conditions.CustomContextParam> = {
    ':selectedBalance': 2,
  };
  conditionContext.addCustomContextParameterValues(customParameters);
}
// other context variable checks ...

const decryptedMessage = await decrypt(
  web3Provider,
  domains.MAINNET,
  messageKit,
  conditionContext,
);
```

## Learn more&#x20;

* [references.md](../references.md "mention")
