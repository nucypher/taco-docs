# Context Variables Cheatsheet

A **context variable** is a placeholder used inside a condition. Its value is supplied at decryption (or signing) time, not at encryption time. That is what makes a single encrypted ciphertext usable across many requesters and many runtime states.

This page is a cheatsheet. For a longer narrative, see [Condition Context](../authentication/conditioncontext-and-context-variables.md).

## Naming rules

- Always start with `:` — e.g. `:userAddress`
- After the `:`, the name must match `/^[a-zA-Z_][a-zA-Z0-9_]*$/`
- Case-sensitive
- No dots, dashes, or spaces. `:user-address`, `:user.address`, `:1stParam` are all invalid.

If you forget the leading `:`, the SDK will treat your value as a literal string and your condition will silently match nothing (or worse, match something).

## Built-in context variables

These are automatically populated by the SDK or by the network. You do **not** need to set them yourself.

| Variable | Set by | Available in | Description |
| :--- | :--- | :--- | :--- |
| `:userAddress` | Decrypter's wallet | All conditions | The Ethereum address of the requester. Replaced after the requester signs an authentication message. |
| `:message` | Decrypter | `EcdsaCondition` | The message that was signed. Default for `EcdsaCondition.message`. |
| `:signature` | Decrypter | `EcdsaCondition` | The signature being verified. Default for `EcdsaCondition.signature`. |
| `:jwtToken` | Decrypter | `JwtCondition` | The JWT being validated. Default for `JwtCondition.jwtToken`. |
| `:signingConditionObject` | Signing flow | `SigningObjectAttributeCondition`, `SigningObjectAbiAttributeCondition` | The full object (e.g. UserOperation) being submitted for threshold signing. |

## Custom context variables

You can define any name you like, as long as it follows the [naming rules](#naming-rules). The decrypter is responsible for supplying its value at decryption time.

```json
{
  "conditionType": "json",
  "data": ":discordPayload",
  "query": "$.member.user.id",
  "returnValueTest": { "comparator": ">", "value": 0 }
}
```

Here `:discordPayload` is a custom variable. The bot supplies it as part of the decryption request.

### Variables produced by SequentialCondition

`SequentialCondition` lets each step bind a `varName`. That name becomes a context variable usable by subsequent steps:

```json
{
  "conditionType": "sequential",
  "conditionVariables": [
    {
      "varName": "balance",
      "condition": { "...": "fetches an ERC20 balance" }
    },
    {
      "varName": "validate",
      "condition": {
        "conditionType": "context-variable",
        "contextVariable": ":balance",
        "returnValueTest": { "comparator": ">=", "value": 1000 }
      }
    }
  ]
}
```

`varName` is a **plain string** (no leading `:`), but you reference it later **with** the `:` prefix.

## Where context variables can appear

| Field type | Accepts context variable? |
| :--- | :--- |
| `returnValueTest.value` | ✅ |
| `ContractCondition.parameters[*]` | ✅ |
| `JsonCondition.data` | ✅ (required — must be a context variable) |
| `JsonCondition.query` | ✅ (or a JSONPath) |
| `JsonApiCondition.endpoint` | ❌ (must be a literal HTTPS URL) |
| `JsonApiCondition.authorizationToken` | ✅ |
| `EcdsaCondition.message` / `signature` | ✅ |
| `JwtCondition.jwtToken` | ✅ |
| `chain`, `method`, `comparator` | ❌ — must be literal |
| ABI function signature keys (`"transfer(address,uint256)"`) | ❌ — must be literal |
| ABI parameter `value` inside `returnValueTest` | ✅ |

## Common gotcha: JSONPath vs context variable

`JsonCondition.query` accepts **either** a JSONPath expression (`$.member.user.id`) **or** a context variable (`:somePath`). The schema disambiguates by the leading character — `$` means JSONPath, `:` means context variable. Anything else is a validation error.
