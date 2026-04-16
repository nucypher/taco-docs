# Building Conditions with an LLM

TACo conditions are JSON. That makes them an excellent target for LLM-assisted authoring: describe the access policy you want in plain English, hand the LLM the right context, and iterate until the validator is happy.

This page is the recommended workflow.

## The context you give the LLM

Paste these four things into your LLM of choice (Claude, ChatGPT, Cursor, etc.) at the start of a new conversation:

1. **The full condition schema** — the canonical, machine-readable definition of every condition type, every field, every allowed value. Two formats are auto-generated from the same TypeScript source:
   - [`condition-schemas.md`](https://github.com/nucypher/taco-web/blob/signing-epic/packages/taco/schema-docs/condition-schemas.md) — human-readable reference for prose-style LLMs.
   - [`condition-schema.json`](https://raw.githubusercontent.com/nucypher/taco-web/signing-epic/packages/taco/schema-docs/condition-schema.json) — standard [JSON Schema](https://json-schema.org/) document. Use this with structured-output LLM APIs and editor `$schema` references (see [JSON Schema integration](#json-schema-integration) below).

   Link these directly — do not vendor copies that will drift.
2. **The cookbook** — [JSON examples covering every condition type](cookbook.md). Examples teach an LLM patterns far faster than prose.
3. **The deep-dive** — the [Discord tipping bot walkthrough](discord-tipping-bot-deep-dive.md). One realistic, complex, fully-annotated condition is worth a thousand toy examples.
4. **The validator script** — [`validate-conditions.ts`](validating-conditions.md). Tell the LLM it can run this and iterate on the output.

## A prompt template

```
You are helping me author a TACo condition in JSON.

Reference material (please read before writing any condition):
- Schema (JSON Schema, machine-readable source of truth):
  https://raw.githubusercontent.com/nucypher/taco-web/signing-epic/packages/taco/schema-docs/condition-schema.json
- Schema (prose version, same source):
  https://github.com/nucypher/taco-web/blob/signing-epic/packages/taco/schema-docs/condition-schemas.md
- Cookbook of examples: https://docs.taco.build/for-developers/taco-sdk/references/conditions/cookbook
- Annotated complex example: https://docs.taco.build/for-developers/taco-sdk/references/conditions/discord-tipping-bot-deep-dive

Rules:
- Output a single JSON object (no prose around it) when I ask for a condition.
- Every field must exist in the schema. Do not invent fields.
- Context variables start with ":" and match /^:[a-zA-Z_][a-zA-Z0-9_]*$/.
- CompoundCondition: max 5 operands.
- MultiConditions (CompoundCondition, IfThenElseCondition, SequentialCondition)
  share a combined nesting depth limit of 4.
- SequentialCondition: 2–20 variables.
- After each condition you produce, I will run validate-conditions.ts and
  paste the output back. Fix any validation errors and try again.

What I want the condition to enforce:
<describe your access policy in plain English>
```

## The iteration loop

1. LLM produces a condition.
2. Save it as `conditions.json`.
3. Run `npx tsx validate-conditions.ts` ([source](validating-conditions.md)).
4. If invalid, paste the error back to the LLM. If valid, test it end-to-end against a local testnet (see [Quickstart](../../../access-control/quickstart-testnet.md)) or your app.

This loop usually converges in 1–3 rounds even for complex conditions.

## Tips that materially improve LLM output

- **Be explicit about the chain ID.** "Base mainnet" is ambiguous to a model that has not read your config; say `"chain": 8453`.
- **Name the data source.** "Check an NFT balance" is vague. "Call `balanceOf(:userAddress)` on contract `0xabc...` on Polygon (137) and require result `> 0`" is unambiguous.
- **Specify the comparator.** Models default to `==` even when you mean `>=`.
- **For sequential conditions**, list variables in dependency order and remind the model that later variables can reference earlier ones with `:varName`.
- **For ABI validation**, give the model the function signature (e.g. `transfer(address,uint256)`) — it cannot guess parameter order reliably.
- **When using `signing-attribute` / `signing-abi-attribute`**, mention which signing object format you are using (UserOperation, plain transaction, custom struct) so the model picks the right `attributeName`.

## JSON Schema integration

The auto-generated [`condition-schema.json`](https://raw.githubusercontent.com/nucypher/taco-web/signing-epic/packages/taco/schema-docs/condition-schema.json) is the highest-leverage piece of tooling available for condition authoring. It works in three places without any TACo dependency:

### Editors

Add `$schema` to the top of any `conditions.json` and your editor (VS Code, Cursor, JetBrains) will validate it inline as you type, with autocomplete for every field:

```json
{
  "$schema": "https://raw.githubusercontent.com/nucypher/taco-web/signing-epic/packages/taco/schema-docs/condition-schema.json",
  "version": "1.0.0",
  "condition": {
    "conditionType": "time",
    "chain": 137,
    "method": "blocktime",
    "returnValueTest": { "comparator": ">", "value": 1735689600 }
  }
}
```

### LLM structured output

Both Anthropic and OpenAI's APIs accept a JSON Schema directly to constrain model output. Example with the Anthropic SDK:

```ts
import Anthropic from '@anthropic-ai/sdk';

const conditionSchema = await fetch(
  'https://raw.githubusercontent.com/nucypher/taco-web/signing-epic/packages/taco/schema-docs/condition-schema.json'
).then(r => r.json());

const result = await new Anthropic().messages.create({
  model: 'claude-opus-4-6',
  max_tokens: 2048,
  tools: [{
    name: 'emit_condition',
    description: 'Emit a TACo condition matching the requested policy.',
    input_schema: conditionSchema,
  }],
  tool_choice: { type: 'tool', name: 'emit_condition' },
  messages: [{
    role: 'user',
    content: 'Build a condition that allows decryption only if the requester holds at least one NFT from collection 0xabc on Ethereum mainnet.',
  }],
});
```

The model is now structurally constrained — it cannot return invalid shapes.

### Standalone validation (any language)

You no longer need `@nucypher/taco` installed to validate. Any standard JSON Schema validator works:

```bash
pnpm dlx ajv-cli validate \
  -s https://raw.githubusercontent.com/nucypher/taco-web/signing-epic/packages/taco/schema-docs/condition-schema.json \
  -d conditions.json --strict=false
```

The Python equivalent uses [`jsonschema`](https://python-jsonschema.readthedocs.io/), the Go equivalent uses [`gojsonschema`](https://github.com/xeipuuv/gojsonschema), etc.

The `validate-conditions.ts` script ([page](validating-conditions.md)) is still useful when you want runtime semantics (e.g. catching nesting-depth errors that JSON Schema cannot express), but for shape validation alone the JSON Schema is enough.
