# TACo Protocol Layers & Components Diagram (Mermaid file)

RELATIONSHIPS:

1.  DRAW.IO FILE: Source of truth for visual representation of the TACo architecture
2.  SVG FILE: Direct visual export from the draw.io file for documentation display
3.  THIS FILE (MERMAID): Simplified LLM-friendly diagram based on the draw.io file

PURPOSE:
This Mermaid diagram provides a simplified version of the TACo architecture that is optimized for LLM processing and understanding. While it contains the same essential components and relationships as the draw.io file, it has been structured to be more easily parsed and understood by language models.

**LAST UPDATED:** 2025-06-02

**NOTE: Update this file whenever significant changes are made to the draw.io file.**

```mermaid
graph TD
%% TACo Architecture Diagram - Version 3.0
%% Updated: 2025-06-02
%% Enhanced for better processing and aligned with draw.io changes

%% data-category: external-systems
 subgraph web-applications["Web3 & Web2 Applications"]
        web-apps["Web Apps"]
        mobile-apps["Mobile Apps"]
  end

%% data-architecture-version: 3.0
 subgraph taco-system["TACo Protocol Layers & Components"]

%% data-layer: application-integration
 subgraph application-domain["Application Integration Layer"]

%% data-component: client-sdk
    subgraph client-sdk["Client SDK (taco-web) (Application Integration Library)"]
%% data-feature: authentication
            subgraph authentication-component["Authentication"]
                    web3-signatures-desc["Request, Collect & Cash Web3 Signatures, SIWE messages and/or OAuth"]
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
            subgraph collect-decryption-params["Collect Decryption's Context Params"]
                    context-params-desc["Decryptors provide the required params values along with the ciphertext"]
            end

%% data-primitive: encryption
            encryption["Encryption (Client-side Operation by data providers)"]
%% data-primitive: decryption
            decryption["Decryption (Client-side Operation using the collected data fragments)"]
    end
  end

%% data-layer: protocol
 subgraph protocol-domain["Protocol Layer"]

%% data-component: taco-node-client
    subgraph taco-node-client["TACo Node Client"]
            node-participation["Registration & Staking (Economic Security Layer)"]
            dkg-participation["DKG Rituals Participation (Triggered by Cohort Admin)"]
            verification-operations["Verification & Operations (Condition Validation & Partial Decryption)"]

%% data-primitive: dkg
            dkg-primitive["Distributed Key Generation (DKG) (Cohort Generates Public Key & Private Fragments)"]
%% data-primitive: threshold
            threshold-schemes-primitive["Generate Decryption Material (Each Node Generates its Decryption Share, if Conditions were Fulfilled)"]
    end

%% data-component: staking-governance
    subgraph staking-governance["Staking & Governance"]
            protocol-governance["Protocol Governance (Parameter Management & Coordinator Contract)"]
            staking-slashing["Staking & Slashing (Economic Incentives)"]
            node-monitoring["Nodes Monitoring (Using dummy DKG rituals)"]
            slashing-contract["Slashing Contract"]
            staking-contract["Staking Contract"]
    end

%% data-component: protocol-coordination
    subgraph protocol-coordination["Protocol Coordination"]
            ritual-coordination["Ritual Coordination (Coordinating DKG participation)"]
            ritual-id["RitualID Management (Unique Identifiers for DKG rituals)"]
            coordinator-contract["Coordinator Contract"]
    end

%% data-component: fees-subscription
    subgraph fees-subscription["Fees & Subscription"]
            fee-collection["Fee Collection (Payment Processing)"]
            fee-distribution["Fee Distribution (Payment Distribution)"]
            fee-contract["Fee Distribution Contract"]
            subscription-contract["Subscription Contract"]
    end
  end

%% data-component: gateway
  porter-service["Porter Service (Protocol Abstraction & Gateway)"]

end

%% data-category: external-systems
subgraph web3-external-system["Web3"]
        blockchain["Blockchain"]
        ipfs["IPFS"]

end
%% data-flow: web3-integration
taco-system <---> web3-external-system


%% data-flow: gateway-connections
application-domain <---> porter-service <---> protocol-domain

%% data-flow: application-connections
web-applications <---> application-domain

```
