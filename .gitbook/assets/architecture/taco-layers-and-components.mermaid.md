# TACo Protocol Layers & Components Diagram (Mermaid file)

RELATIONSHIPS:

1.  DRAW.IO FILE: Source of truth for visual representation of the TACo architecture
2.  SVG FILE: Direct visual export from the draw.io file for documentation display
3.  THIS FILE (MERMAID): Simplified LLM-friendly diagram based on the draw.io file

PURPOSE:
This Mermaid diagram provides a simplified version of the TACo architecture that is optimized for LLM processing and understanding. While it contains the same essential components and relationships as the draw.io diagram, it has been structured to be more easily parsed and understood by language models.

**NOTE: Update this file whenever significant changes are made to the draw.io file.**

```mermaid
graph TD
 subgraph app-integration["Applications"]
        web-apps["Web Apps"]
        mobile-apps["Mobile Apps"]
  end

 subgraph taco-system["TACo Protocol Layers & Components"]

 subgraph application-domain["Application Integration Layer"]

    subgraph client-sdk["Client SDK (taco-web) (Application Integration Library)"]
            subgraph authentication["Authentication"]
                    web3-signatures-desc["Request, Collect & Cash Web3 Signatures"]
            end

            subgraph define-decryption-conditions["Define Decryption Conditions (Data Authorization)"]
                    time-condition["TimeCondition"]
                    rpc-condition["RpcCondition"]
                    contract-condition["ContractCondition"]
                    json-condition["JsonApiCondition"]
                    compound-condition["CompoundCondition (AND/OR/NOT)"]
                    more-conditions["(AND MORE)"]
            end
    end
  end
 subgraph node-infrastructure["TACo Nodes Operations"]
        node-participation["Registration & Staking (Economic Security Layer)"]
        cohort-organization["DKG Rituals Participation (Triggered by Cohort Admin)"]
        verification-operations["Verification & Operations (Condition Validation & Partial Decryption)"]
  end
 subgraph governance-economics["Governance & Economics"]
        protocol-governance["Protocol Governance (Parameter Management & Coordinator Contract)"]
        fee-collection["Fee Collection (Payment Processing & Distribution)"]
        staking-slashing["Staking & Slashing (Economic Incentives)"]
        cohort-authority-mgmt["Cohort Authority Management (Authority Transfer & Control)"]
  end
 subgraph ritual-management["Ritual Management System"]
        ritual-id["RitualID Management (Unique Identifiers for DKG rituals)"]
        ritual-coordination["Ritual Coordination (Coordinating DKG participation)"]
        key-management["Key Management (Tracking public keys & thresholds)"]
        node-monitoring["Nodes Monitoring (Using dummy DKG rituals)"]
  end
 subgraph blockchain-integration["Blockchain Integration"]
        coordinator-contract["Coordinator Contract"]
        staking-contract["Staking Contract"]
        fee-contract["Fee Management"]
  end
porter-service["Porter Service (Protocol Abstraction & Gateway)"]
 subgraph protocol-domain["Protocol Layer"]
        node-infrastructure
        governance-economics
        ritual-management
        blockchain-integration
  end
 subgraph crypto-layer["Cryptographic Layer"]
        dkg["Distributed Key Generation (DKG) (Cohort Generates Public Key & Private Fragments)"]
        encryption["Encryption (Client-side Operation by data providers)"]
        threshold-schemes["Generate decrypted data fragment (Nodes Generate Data Fragments, if Conditions were fulfilled)"]
        decryption["Decryption (Client-side Operation using the collected data fragments)"]
  end

  end
    app-integration <---> application-domain
    protocol-domain --- crypto-layer --- application-domain
    application-domain <---> porter-service <---> protocol-domain
    dkg --> encryption --> threshold-schemes --> decryption
```
