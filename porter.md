# Porter

## Overview

Porter can be described as the _“Infura for TACo”_. Porter is a web-based service that performs TACo-based protocol operations for applications.

Its goal is to simplify and abstract the complexities surrounding the TACo protocol to negate the need for applications to interact with it via a Python client. Porter introduces the TACo protocol to cross-platform functionality, including web and mobile applications.

<figure><img src=".gitbook/assets/porter_diagram (2).png" alt=""><figcaption></figcaption></figure>

Any publicly available Porter can be used to interface with the Threshold Network, or some application developers opt to [run their own](porter.md#running-a-porter-instance).

## Public Porter Instances

Public Porter instances are operated by centralized entities and have different security properties than user-operated instances. If you're interested in running your own Porter, see  [#running-a-porter-instance](porter.md#running-a-porter-instance "mention")

To use the default Porter URIs in `taco`, run:

```typescript
import { domains, getPorterUri } from '@nucypher/taco';

const porterUri = getPorterUri(domains.MAINNET);  // mainnet
// OR
const devPorterUri = getPorterUri(domains.DEV);  // lynx testnet
// OR
const testnetPorterUri = getPorterUri(domains.TESTNET);  // tapir testnet
```
