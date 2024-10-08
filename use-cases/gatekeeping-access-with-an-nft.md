# Gatekeeping access with an NFT

We provide two predefined conditions for gate access using NFTs: `ERC721Ownership`, which tests for the ownership of the specific NFT from a series, and `ERC721Balance`, which tests for the balance of any NFT token in a series.

```typescript
import { conditions } from '@nucypher/taco';

// The decrypter must own a specific NFT
const ownsNFT = new conditions.predefined.erc721.ERC721Ownership({
  contractAddress: '0x1e988ba4692e52Bc50b375bcC8585b95c48AaD77',
  parameters: [3591], // Id of a specific NFT
  chain: 1,
});

// The decrypter must own at least two of NFT from this series
const hasAtLeastTwoNFTs = new conditions.predefined.erc721.ERC721Balance({
  contractAddress: '0x1e988ba4692e52Bc50b375bcC8585b95c48AaD77',
  chain: 1,
  returnValueTest: {
    comparator: '>=',
    value: 2
  },
});
```

Under the hood, those predefined conditions are defined using `ContractCondition`

```typescript
import { conditions } from '@nucypher/taco';

const ownsNFTRaw = new conditions.base.contract.ContractCondition({
  // Values provided by the `predefined.ERC721Balance`
  method: 'balanceOf',
  parameters: [':userAddress'],
  standardContractType: 'ERC721',
  // Values that we always have to provide
  contractAddress: '0x1e988ba4692e52Bc50b375bcC8585b95c48AaD77',
  chain: 1,
  returnValueTest: {
    comparator: '>',
    value: 0,
  },
});
```

### Learn more&#x20;

* [ContractCondition](../conditions/contractcondition/)
* [references.md](../references.md "mention")

