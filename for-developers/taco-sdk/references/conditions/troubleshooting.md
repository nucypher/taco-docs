# Troubleshooting Conditions

The TACo SDK validates conditions with [Zod](https://zod.dev/), and Zod's error messages are precise but terse. This page maps the errors you will actually see to what they mean and how to fix them.

The fastest workflow: run the [validator script](validating-conditions.md), copy the error, look it up here (or paste it into an LLM with the [building-with-llms guide](building-with-llms.md)).

## Schema-shape errors

### `Invalid literal value, expected "and"` (or `"or"`, `"not"`)

You wrote `"operator": "AND"` or `"operator": "&&"`. Operators are lowercase strings. Allowed: `"and"`, `"or"`, `"not"`.

### `Invalid literal value, expected "compound"` (or any other `conditionType`)

`conditionType` is wrong or missing. Each condition type has an exact required string — see the [schema reference](https://github.com/nucypher/taco-web/blob/signing-epic/packages/taco/schema-docs/condition-schemas.md). Common typos: `"contractCondition"` (should be `"contract"`), `"jsonApi"` (should be `"json-api"`), `"ifThenElse"` (should be `"if-then-else"`), `"signingAttribute"` (should be `"signing-attribute"`).

### `Required` on a field you thought was optional

The schema reference marks required fields with `(*)`. Common surprises:

- `ContractCondition.parameters` is **required** (use `[]` if the function takes none)
- `ContractCondition.contractAddress` is **required**
- `JsonApiCondition.endpoint` is **required**
- `EcdsaCondition.verifyingKey` and `curve` are **required**
- `TimeCondition.chain` is **required** (`method` defaults to `"blocktime"`)

### `Array must contain at least 2 element(s)` on a SequentialCondition

`SequentialCondition.conditionVariables` requires 2–20 entries. If you only need one variable, you do not need a SequentialCondition — just use the inner condition directly.

### `Array must contain at most 5 element(s)` on a CompoundCondition

`CompoundCondition.operands` is capped at 5. To express more, embed a `SequentialCondition` (or another `CompoundCondition`, but only one level deep — see next).

### `Compound condition nesting depth exceeded`

A `CompoundCondition` can contain a sub-`CompoundCondition`, but **that sub-compound cannot itself contain a compound**. Maximum nesting is 2 levels. To go deeper, replace the inner compound with a `SequentialCondition`.

### `String must match pattern /^:[a-zA-Z_][a-zA-Z0-9_]*$/`

A context variable name is malformed. They must start with `:`, followed by a letter or underscore, then letters/digits/underscores. No dots, dashes, spaces, or leading digits. See [Context Variables](context-variables.md).

### `Invalid url` on `JsonApiCondition.endpoint`

`endpoint` must be a literal HTTPS URL. You cannot pass a context variable here. If you need a dynamic endpoint, you must commit to a specific service at encryption time.

### `Invalid hex string` on `verifyingKey`, `contractAddress`, etc.

Hex strings must contain only `0-9a-fA-F`. `verifyingKey` (Ed25519/SECP256k1 raw bytes) does **not** include a `0x` prefix; addresses **do**. Check the schema for which is which.

## Logical errors that pass validation but fail at runtime

These do not produce a Zod error — they fail when the network actually evaluates the condition. Symptoms: decryption requests are rejected by Porter or by individual nodes.

### "Decryption denied" but the condition looks right

- **Wrong chain ID.** A condition that calls Base USDC with `"chain": 1` (Ethereum) will hit a contract that does not exist there. Sanity-check every chain ID.
- **Stale block tag.** `RpcCondition` defaults its block tag to `"latest"`, but you can pass `"finalized"` or `"safe"`. If you used `"earliest"`, you are reading genesis state.
- **Comparator inverted.** `<` vs `>=` typos are surprisingly common. Re-read the test out loud: "the value must be greater than or equal to X."

### "Cannot decode result" from a `ContractCondition`

The function exists but the `outputs` array in your `functionAbi` does not match what the contract actually returns. Double-check against the source contract or a block explorer's verified ABI.

### `JsonCondition` test always fails

Your JSONPath probably does not match. JSONPath is picky:

- Filter expressions need parens: `[?(@.name == "amount")]`, not `[@.name == "amount"]`.
- Array element access uses `[0]`, not `.0`.
- Test your path against the actual JSON in [jsonpath.com](https://jsonpath.com/) before deploying.

### `SigningObjectAbiAttributeCondition` rejects valid calldata

- Function signature must be **exact**, including parameter types — `transfer(address,uint256)` not `transfer(address, uint256)` (no spaces) and not `transfer(address,uint)` (`uint` is invalid; use `uint256`).
- For tuple parameters, use `(type1,type2,...)`. Example from the [Discord tipping bot deep-dive](discord-tipping-bot-deep-dive.md): `execute((address,uint256,bytes))`.
- `subIndices` walk into nested structures one level at a time. For a tuple `(address, uint256, bytes)`, `subIndices: [2]` selects the `bytes` field.

### `SequentialCondition` variable not found

You wrote `:varname` but the bound `varName` was `varName` (no leading colon, case-sensitive). The binding uses the bare name; the reference uses `:` + name.

## When all else fails

1. Run [`validate-conditions.ts`](validating-conditions.md) and read the full error in `validation-error.txt`.
2. Compare against the closest example in the [cookbook](cookbook.md).
3. Paste the error + your condition + the [schema reference URL](https://github.com/nucypher/taco-web/blob/signing-epic/packages/taco/schema-docs/condition-schemas.md) into an LLM. See [Building Conditions with an LLM](building-with-llms.md).
4. Test end-to-end on the [TACo Playground](https://playground.taco.build/).
