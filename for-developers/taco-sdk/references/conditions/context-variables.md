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

The SDK recognises a small set of reserved or default context-variable names. Their behaviour differs — read the table carefully before assuming a value is "automatic".

| Variable | Kind | Available in | Description |
| :--- | :--- | :--- | :--- |
| `:signingConditionObject` | **Auto-injected** (node side) | `SigningObjectAttributeCondition`, `SigningObjectAbiAttributeCondition` | The full object (e.g. UserOperation) being submitted for threshold signing. You do not need to add this to the condition context — the node injects it at evaluation time. |
| `:nullAddress` | **Auto-injected** (node side) | All conditions | The zero address (`0x0000…0000`). Useful as a sentinel in allowlists or when a field genuinely means "no address". The node injects the value; you cannot override it. |
| `:userAddress` | **Reserved** | All conditions | The Ethereum address of the requester. Its value can only be provided by an `AuthProvider` at request time — it is never set by user code directly. |
| `:message` | **Default name** | `EcdsaCondition` | Default for `EcdsaCondition.message`. Not automatically populated — the decrypter must supply a value at request time. |
| `:signature` | **Default name** | `EcdsaCondition` | Default for `EcdsaCondition.signature`. Not automatically populated — the decrypter must supply a value at request time. |
| `:jwtToken` | **Default name** | `JwtCondition` | Default for `JwtCondition.jwtToken`. Not automatically populated — the decrypter must supply a value at request time. |

> The source of truth for this table is the `AUTOMATICALLY_INJECTED_CONTEXT_PARAMS` and `RESERVED_CONTEXT_PARAMS` arrays in [`taco-web/packages/taco/src/conditions/context/context.ts`](https://github.com/nucypher/taco-web/blob/signing-epic/packages/taco/src/conditions/context/context.ts).

### Auto-injected vs. reserved vs. default

- **Auto-injected** means the node populates the value at evaluation time. Setting these manually via `ConditionContext.addCustomContextVariableValues` will throw. `:signingConditionObject` and `:nullAddress` are the only auto-injected variables.
- **Reserved** means the name is fixed and the value can only come from a specific source. `:userAddress` is reserved and can only be supplied by an `AuthProvider` — attempting to set it as a custom parameter will throw.
- **Default name** means the condition *defaults* to that variable name, but you can override it. For example, if a single compound condition contains two `JwtCondition`s that must validate two different tokens, give each its own variable:

```json
{
  "conditionType": "compound",
  "operator": "and",
  "operands": [
    {
      "conditionType": "jwt",
      "jwtToken": ":idpToken",
      "publicKey": "…",
      "expectedIssuer": "https://idp.example.com/"
    },
    {
      "conditionType": "jwt",
      "jwtToken": ":partnerToken",
      "publicKey": "…",
      "expectedIssuer": "https://partner.example.com/"
    }
  ]
}
```

The same applies to `EcdsaCondition.message` and `EcdsaCondition.signature` when verifying multiple signatures in one condition.

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
