# Building Conditions with an LLM

TACo conditions are JSON. That makes them an excellent target for LLM-assisted authoring: describe the access policy you want in plain English, hand the LLM the right context, and iterate until the validator is happy.

This page is the recommended workflow.

## The context you give the LLM

Paste these four things into your LLM of choice (Claude, ChatGPT, Cursor, etc.) at the start of a new conversation:

1. **The full condition schema** — the canonical, machine-readable definition of every condition type, every field, every allowed value:\
   [`condition-schemas.md` on `taco-web`](https://github.com/nucypher/taco-web/blob/signing-epic/packages/taco/schema-docs/condition-schemas.md)\
   This is auto-generated from the TypeScript source, so it is always the source of truth. Link it directly — do not vendor a copy that will drift.
2. **The cookbook** — [JSON examples covering every condition type](cookbook.md). Examples teach an LLM patterns far faster than prose.
3. **The deep-dive** — the [Discord tipping bot walkthrough](discord-tipping-bot-deep-dive.md). One realistic, complex, fully-annotated condition is worth a thousand toy examples.
4. **The validator script** — [`validate-conditions.ts`](validating-conditions.md). Tell the LLM it can run this and iterate on the output.

## A prompt template

```
You are helping me author a TACo condition in JSON.

Reference material (please read before writing any condition):
- Schema (source of truth): https://github.com/nucypher/taco-web/blob/signing-epic/packages/taco/schema-docs/condition-schemas.md
- Cookbook of examples: https://docs.taco.build/for-developers/taco-sdk/references/conditions/cookbook
- Annotated complex example: https://docs.taco.build/for-developers/taco-sdk/references/conditions/discord-tipping-bot-deep-dive

Rules:
- Output a single JSON object (no prose around it) when I ask for a condition.
- Every field must exist in the schema. Do not invent fields.
- Context variables start with ":" and match /^:[a-zA-Z_][a-zA-Z0-9_]*$/.
- CompoundCondition: max 5 operands, max 2 levels of nesting.
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
4. If invalid, paste the error back to the LLM. If valid, test it end-to-end with the [TACo Playground](https://playground.taco.build/) or your app.

This loop usually converges in 1–3 rounds even for complex conditions.

## Tips that materially improve LLM output

- **Be explicit about the chain ID.** "Base mainnet" is ambiguous to a model that has not read your config; say `"chain": 8453`.
- **Name the data source.** "Check an NFT balance" is vague. "Call `balanceOf(:userAddress)` on contract `0xabc...` on Polygon (137) and require result `> 0`" is unambiguous.
- **Specify the comparator.** Models default to `==` even when you mean `>=`.
- **For sequential conditions**, list variables in dependency order and remind the model that later variables can reference earlier ones with `:varName`.
- **For ABI validation**, give the model the function signature (e.g. `transfer(address,uint256)`) — it cannot guess parameter order reliably.
- **When using `signing-attribute` / `signing-abi-attribute`**, mention which signing object format you are using (UserOperation, plain transaction, custom struct) so the model picks the right `attributeName`.

## llms.txt

This documentation site exposes an [`llms.txt`](https://docs.taco.build/llms.txt) at the root, following the [emerging convention](https://llmstxt.org/). Point your LLM agent at it and it will discover the highest-signal pages on its own.
