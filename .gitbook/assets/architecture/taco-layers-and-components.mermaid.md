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
%% TACo Architecture Diagram - Version 2.0
%% Enhanced for better processing
%% data-category: user-applications
 subgraph app-integration["Applications"]
        web-apps["Web Apps"]
        mobile-apps["Mobile Apps"]
  end

%% data-architecture-version: 2.0
 subgraph taco-system["TACo Protocol Layers & Components"]

%% data-layer: application
 subgraph application-domain["Application Integration Layer"]

%% data-component: client-sdk
    subgraph client-sdk["Client SDK (taco-web) (Application Integration Library)"]
%% data-feature: authentication
            subgraph authentication["Authentication"]
                    web3-signatures-desc["Request, Collect & Cash Web3 Signatures"]
            end

%% data-feature: conditions
            subgraph define-decryption-conditions["Define Decryption Conditions (Data Authorization)"]
                    time-condition["TimeCondition"]
                    rpc-condition["RpcCondition"]
                    contract-condition["ContractCondition"]
                    json-condition["JsonApiCondition"]
                    compound-condition["CompoundCondition (AND/OR/NOT)"]
                    more-conditions["(AND MORE)"]
            end
            
%% data-feature: context-params
            subgraph collect-context-params["Collect Decryption's Context Params"]
                    context-params-desc["Decryptors provide the required params values along with the ciphertext"]
            end
    end
  end
%% data-component: nodes
 subgraph node-infrastructure["TACo Nodes Operations"]
        node-participation["Registration & Staking (Economic Security Layer)"]
        cohort-organization["DKG Rituals Participation (One-time setup triggered by Cohort Admin)"]
        verification-operations["Verification & Operations (Condition Validation & Partial Decryption)"]
  end
%% data-component: governance
 subgraph governance-economics["Governance & Economics"]
        protocol-governance["Protocol Governance (Parameter Management & Coordinator Contract)"]
        fee-collection["Fee Collection (Payment Processing & Distribution)"]
        staking-slashing["Staking & Slashing (Economic Security Layer)"]
        cohort-authority-mgmt["Cohort Authority Management (Economic staking & parameters set by cohortAuthority)"]
  end
%% data-component: ritual-management
 subgraph ritual-management["Ritual Management System"]
        ritual-id["RitualID Management (Unique Identifiers for DKG rituals)"]
        ritual-coordination["Ritual Coordination (Coordinating DKG participation)"]
        key-management["Key Management (Tracking public keys & thresholds)"]
        node-monitoring["Nodes Monitoring (Using dummy DKG rituals)"]
  end
%% data-component: blockchain
 subgraph blockchain-integration["Blockchain Integration"]
        coordinator-contract["Coordinator Contract"]
        staking-contract["Staking Contract"]
        fee-contract["Fee Management"]
  end
%% data-component: gateway
porter-service["Porter Service (Protocol Abstraction & Gateway)"]
%% data-layer: protocol
 subgraph protocol-domain["Protocol Layer"]
        node-infrastructure
        governance-economics
        ritual-management
        blockchain-integration
  end
%% data-layer: cryptography
 subgraph crypto-layer["Cryptographic Layer"]
%% data-operation: key-generation
        dkg["Distributed Key Generation (DKG) (One-time generation of Public Key & Private Fragments)"]
%% data-operation: data-encryption
        encryption["Encryption (Client-side Operation by data owners using public key)"]
%% data-operation: fragment-generation
        threshold-schemes["Generate decrypted data fragment (Nodes provide fragments when conditions are met)"]
%% data-operation: data-decryption
        decryption["Decryption (Client-side Operation using the collected data fragments)"]
  end

  end
%% data-flow: app-integration
    app-integration <---> application-domain
%% data-flow: layer-connections
    protocol-domain --- crypto-layer --- application-domain
%% data-flow: porter-gateway
    application-domain <---> porter-service <---> protocol-domain
%% data-flow: crypto-operations
    dkg --> encryption --> threshold-schemes --> decryption
```
