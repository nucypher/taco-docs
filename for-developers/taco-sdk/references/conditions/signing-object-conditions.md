# Signing Object Conditions

{% hint style="warning" %}
TACo Threshold Signing functionality is currently in alpha and available only on the `DEVNET` environment.
{% endhint %}

TACo supports validating specific fields within signing request objects before allowing a threshold signature. Two condition types are currently supported:

## `SigningObjectAttributeCondition`

Validates a **simple attribute** on the signing object (e.g., a top-level field in the request object).

It is composed of the following properties:

* `attributeName`: The name of the attribute to inspect.
* `returnValueTest`: the test to validate the value extracted from the attribute of the object to sign.

### Example

Ensure the `callGasLimit` value for a `UserOperation` is less than 100,000 units of gas.

```typescript
import { conditions } from '@nucypher/taco';

const gasLimitRestriction = new conditions.base.signing.SigningObjectAttributeCondition({
  attributeName: 'callGasLimit',
  returnValueTest: {
    comparator: '<',
    value: 100000,
  },
});
```

## `SigningObjectAbiAttributeCondition`

Allows validation of ABI-encoded call data submitted for threshold signing (eg. `UserOperation` 's `callData`) using human readable ABI signatures. This is useful when the object to sign represents a function call (e.g. a smart contract interaction) and you want to enforce constraints on the function being called and its arguments.

It is composed of the following properties:

* **`attributeName`**: The name of the field in the signing object containing the ABI-encoded data.
* **`abiValidation`**: See [`AbiCallValidation`](signing-object-conditions.md#abicallvalidation) below.

### AbiCallValidation

Defines which function signatures are allowed and what validations must apply to their parameters.

Key Fields:

* `allowedAbiCalls` : A mapping of human-readable ABI signatures (e.g., `"transfer(address,uint256)"`) to an array of `AbiParameterValidation` rules. At least one function signature entry must be defined.

### AbiParameterValidation

Specifies how to validate an individual parameter within the ABI-encoded call.

#### Key Fields:

* `parameterIndex`: zero-based index of the parameter to validate.
* `indexWithinTuple` _(Optional)_: If the value at `parameterIndex` is a tuple, the index of a value within the tuple, if applicable.
* `returnValueTest` _(Optional)_: the test to validate the value extracted from the `parameterIndex` / `indexWithinTuple` combination.
* `nestedAbiValidation` _(Optional)_: Recursively apply ABI call validation on the decoded `bytes` payload if applicable.

**Constraints:**

* Only one of `returnValueTest` or `nestedAbiValidation` may be set for a given parameter.
* Ensures correct Solidity types and validates that indices fall within expected bounds.

### **Example**

Only allow signing of a `UserOperation` for ERC-4337 if the `callData` field is trying to send an ERC-20 transfer of token at address `0x818E59818647700913Cf83f691dcC1b50Ac4864E` and an amount < `10000000000000000000` wei.&#x20;

_Assume that the initial execution call data of the_ `UserOperation` _for the smart contract wallet uses_ `execute(address,uint256,bytes)`.

```typescript
import { conditions } from '@nucypher/taco';

const callDataRestriction = new conditions.base.signing.SigningObjectAbiAttributeCondition({
  attributeName: 'callData',
  abiValidation: {
    allowedAbiCalls: {
      'execute(address,uint256,bytes)': [
        // check address of ERC-20 contract
        {
          parameterIndex: 0,
          returnValueTest: {
            comparator: '==',
            value: '0x818E59818647700913Cf83f691dcC1b50Ac4864E',
          },
        },
        // check bytes used for call to be made on ERC-20 contract
        {
          parameterIndex: 2,
          nestedAbiValidation: {
            allowedAbiCalls: {
              'transfer(address,uint256)': [
                {
                  parameterIndex: 1,
                  returnValueTest: {
                    comparator: '<',
                    value: 10000000000000000000,
                  },
                },
              ],
            },
          },
        },
      ],
    },
  },
});
```

## Development References

* Client-side:
  * [https://github.com/nucypher/taco-web/pull/670](https://github.com/nucypher/taco-web/pull/670)
* Server-side:
  * [https://github.com/nucypher/nucypher/pull/3606](https://github.com/nucypher/nucypher/pull/3606)
  * [https://github.com/nucypher/nucypher/pull/3609](https://github.com/nucypher/nucypher/pull/3609)
