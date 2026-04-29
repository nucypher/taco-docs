# Validating Conditions

Before you ship a condition — or before you ask the network to evaluate one — validate it locally. Client-side validation catches the overwhelming majority of mistakes (typos, wrong field names, missing required properties, illegal nesting depth, comparator typos) without ever touching a node.

You have two options. Pick whichever fits your stack — they overlap heavily.

| Approach | When to use |
| :--- | :--- |
| **JSON Schema validator** (any language) | You want zero TACo dependencies, you are validating in CI, or you want inline editor validation via `$schema`. See [Building Conditions with an LLM → JSON Schema integration](building-with-llms.md#json-schema-integration). |
| **`validate-conditions.ts`** (this page) | You are already in TypeScript, or you need the SDK's runtime semantics (e.g. nesting-depth checks that JSON Schema cannot express). |

This page gives you a 40-line TypeScript script you can drop into any project.

## The script

Save this as `validate-conditions.ts`:

```ts
#!/usr/bin/env npx tsx
/**
 * Validates conditions.json against the TACo SDK schemas.
 * Catches client-side validation errors before they hit the network.
 */

import { conditions } from '@nucypher/taco';
import * as fs from 'fs';
import * as path from 'path';

async function main() {
  const conditionsPath = path.join(__dirname, '..', 'conditions.json');
  console.log(`Loading conditions from: ${conditionsPath}`);

  const conditionsJson = fs.readFileSync(conditionsPath, 'utf-8');
  const conditionsObj = JSON.parse(conditionsJson);

  console.log('\nConditions loaded:');
  console.log(JSON.stringify(conditionsObj, null, 2));

  console.log('\n--- Validating with TACo SDK ---\n');

  try {
    const expr = conditions.conditionExpr.ConditionExpression.fromObj(conditionsObj);
    console.log('✅ Conditions are VALID according to TACo SDK!');
    console.log('\nParsed condition type:', expr.condition.conditionType);
  } catch (error) {
    console.error('❌ Conditions are INVALID according to TACo SDK!');
    const errorStr = error instanceof Error
      ? `${error.name}: ${error.message}\n\nStack: ${error.stack}`
      : JSON.stringify(error, null, 2);
    const errorPath = path.join(__dirname, 'validation-error.txt');
    fs.writeFileSync(errorPath, errorStr);
    console.error(`\nFull error written to: ${errorPath}`);
    console.error('\nError message:', error instanceof Error ? error.message : String(error));
    process.exit(1);
  }
}

main().catch(console.error);
```

## Setup

```bash
npm install --save-dev @nucypher/taco tsx
# or
pnpm add -D @nucypher/taco tsx
```

Place your condition in `conditions.json` at the project root (or adjust the path inside the script).

## Run it

```bash
npx tsx scripts/validate-conditions.ts
```

### Valid output

```
Loading conditions from: .../conditions.json

Conditions loaded:
{
  "version": "1.0.0",
  "condition": { ... }
}

--- Validating with TACo SDK ---

✅ Conditions are VALID according to TACo SDK!

Parsed condition type: compound
```

### Invalid output

```
❌ Conditions are INVALID according to TACo SDK!

Full error written to: scripts/validation-error.txt

Error message: [
  {
    "code": "invalid_literal",
    "expected": "and",
    "received": "AND",
    "path": ["condition", "operator"],
    "message": "Invalid literal value, expected \"and\""
  }
]
```

The full Zod error (with stack) is written to `validation-error.txt` so you can paste it into an LLM verbatim — see [Building Conditions with an LLM](building-with-llms.md) and [Troubleshooting](troubleshooting.md).

## What this catches

- Unknown / misspelled `conditionType` values
- Missing required fields (`returnValueTest`, `chain`, `parameters`, …)
- Wrong literal values (`"AND"` instead of `"and"`, `"=="` written as `"="`)
- MultiCondition nesting deeper than the allowed limit (4 levels total across `CompoundCondition`, `IfThenElseCondition`, and `SequentialCondition`)
- Compound condition with more than 5 operands
- Sequential condition with fewer than 2 or more than 20 variables
- Context variable names that violate the `/^:[a-zA-Z_][a-zA-Z0-9_]*$/` pattern
- ABI validation referencing parameter indices that do not exist on the declared function signature
- Invalid hex strings for verifying keys, contract addresses, bytecode hashes

## What this does **not** catch

Client-side validation only checks shape. It cannot tell you:

- Whether the contract at `contractAddress` actually exists or has the function you declared
- Whether the JSON API endpoint will return data in the shape your JSONPath expects
- Whether the user will satisfy the condition at decryption time

For end-to-end testing, run the condition against a local testnet (see [Quickstart](../../../access-control/quickstart-testnet.md)) or your own integration.

## Source

The canonical copy of this script lives at [`nucypher/taco-web/packages/taco/scripts/validate-conditions.ts`](https://github.com/nucypher/taco-web/blob/main/packages/taco/scripts/validate-conditions.ts).
