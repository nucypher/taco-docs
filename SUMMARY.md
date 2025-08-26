# Table of contents

## Getting Started

* [Introduction to TACo](README.md)
* [How TACo Works](getting-started/key-concepts/README.md)
  * [Access Control](getting-started/key-concepts/access-control.md)
  * [Action Control](getting-started/key-concepts/action-control.md)

## For Developers

* [Testnets](for-developers/get-started-with-tac.md)
* [Access Control](for-developers/access-control/README.md)
  * [Quickstart (Testnet)](for-developers/access-control/quickstart-testnet.md)
  * [Integrate Into Apps](for-developers/access-control/taco-integration/README.md)
    * [Encryptor Allowlist](for-developers/access-control/taco-integration/encryptor-allowlist.md)
  * [Ecosystem Integrations](for-developers/access-control/integrations/README.md)
    * [OrbisDB](for-developers/access-control/integrations/orbisdb.md)
    * [Waku](for-developers/access-control/integrations/waku.md)
    * [Waku + Codex](for-developers/access-control/integrations/waku-+-codex.md)
    * [Irys](for-developers/access-control/integrations/irys.md)
    * [ComposeDB](for-developers/access-control/integrations/composedb.md)
    * [Turbo](for-developers/access-control/integrations/turbo.md)
* [Action Control](for-developers/action-control/README.md)
  * [Quickstart (Testnet)](for-developers/action-control/quickstart-testnet.md)
* [TACo SDK](for-developers/taco-sdk/README.md)
  * [Programmable Conditions](for-developers/taco-sdk/references/README.md)
    * [Authentication](for-developers/taco-sdk/references/authentication/README.md)
      * [Condition Context](for-developers/taco-sdk/references/authentication/conditioncontext-and-context-variables.md)
    * [Conditions](for-developers/taco-sdk/references/conditions/README.md)
      * [TimeCondition](for-developers/taco-sdk/references/conditions/timecondition.md)
      * [RpcCondition](for-developers/taco-sdk/references/conditions/rpccondition.md)
      * [ContractCondition](for-developers/taco-sdk/references/conditions/contractcondition/README.md)
        * [Use custom contract calls](for-developers/taco-sdk/references/conditions/contractcondition/use-custom-contract-calls.md)
        * [Implement revocation via smart contract](for-developers/taco-sdk/references/conditions/contractcondition/implementing-revocation-via-smart-contract.md)
      * [JSON Endpoint Conditions](for-developers/taco-sdk/references/conditions/json-endpoint-conditions/README.md)
        * [JsonApiCondition](for-developers/taco-sdk/references/conditions/json-endpoint-conditions/jsonapicondition.md)
        * [JsonRpcCondition](for-developers/taco-sdk/references/conditions/json-endpoint-conditions/jsonrpccondition.md)
      * [JWTCondition](for-developers/taco-sdk/references/conditions/jwtcondition.md)
      * [Signing Object Conditions](for-developers/taco-sdk/references/conditions/signing-object-conditions.md)
      * [Logical Conditions](for-developers/taco-sdk/references/conditions/logical-conditions/README.md)
        * [CompoundCondition](for-developers/taco-sdk/references/conditions/logical-conditions/condition-set.md)
        * [IfThenElseCondition](for-developers/taco-sdk/references/conditions/logical-conditions/ifthenelsecondition.md)
        * [SequentialCondition](for-developers/taco-sdk/references/conditions/logical-conditions/sequentialcondition.md)
  * [WIP / Feature Requests](for-developers/taco-sdk/wip-feature-requests/README.md)
    * [Any (Major) EVM Chain Condition Support](for-developers/taco-sdk/wip-feature-requests/any-major-evm-chain-condition-support.md)

***

* [Migration to Mainnet](migration-to-mainnet/README.md)
  * [Mainnet Access](migration-to-mainnet/mainnet-taco-beta-program.md)
  * [Mainnet Deployment](migration-to-mainnet/deploying-to-mainnet.md)

## For Product Leads

* [Value Propositions](for-product-leads/value-propositions.md)
* [Access Control](for-product-leads/access-control/README.md)
  * [Use cases](for-product-leads/access-control/use-cases/README.md)
    * [Seed phrase recovery & transfer](for-product-leads/access-control/use-cases/seed-phrase-recovery-and-transfer.md)
    * [Digital Rights Management for on-chain assets](for-product-leads/access-control/use-cases/digital-rights-management-for-on-chain-assets.md)
    * [Trustless channels for journalists, archivists & whistleblowers](for-product-leads/access-control/use-cases/trustless-channels-for-journalists-archivists-and-whistleblowers.md)
    * [Crowdsourcing real-world data with trustless contribution](for-product-leads/access-control/use-cases/crowdsourcing-real-world-data-with-trustless-contribution.md)
* [Action Control](for-product-leads/action-control/README.md)
  * [Use Cases](for-product-leads/action-control/use-cases.md)
  * [Limitations](for-product-leads/action-control/limitations.md)
* [Trust Assumptions](for-product-leads/trust-assumptions/README.md)
  * [Mainnet Trust Disclosure (Provider Answers)](for-product-leads/trust-assumptions/mainnet-trust-disclosure-provider-answers.md)
  * [Mainnet Trust Model Foundation](for-product-leads/trust-assumptions/mainnet-trust-model-foundation.md)
  * [Trust levers & parameter packages](for-product-leads/trust-assumptions/trust-levers-and-parameter-packages.md)

***

* [Mainnet Fees](mainnet-fees.md)

## Reference

* [Contract Addresses](reference/contract-addresses.md)
* [Architecture](reference/architecture/README.md)
  * [Protocol Architecture](reference/architecture/protocol-architecture.md)
  * [Protocol Flow](reference/architecture/protocol-flow.md)
  * [UML Use Case Diagram](reference/architecture/uml-usecase-diagram.md)
  * [Porter](reference/architecture/porter.md)
* [Github](https://github.com/nucypher/taco-web)
* [TACo Playground](https://playground.taco.build/)
* [TACo Scan](https://tacoscan.com/)

## For Node Operators

* [Getting Set Up](for-node-operators/getting-set-up/README.md)
  * [Minimum System Requirements](for-node-operators/getting-set-up/minimum-system-requirements.md)
  * [Run a TACo Node with Docker](for-node-operators/getting-set-up/run-a-taco-node-with-docker.md)
* [Operations](for-node-operators/operations/README.md)
  * [TACo Node Management](for-node-operators/operations/taco-node-management.md)
  * [TACo Node Recovery](for-node-operators/operations/taco-node-recovery.md)
  * [Stake Authorization](for-node-operators/operations/stake-authorization.md)
* [Duties, Compensation & Penalties](for-node-operators/duties-compensation-and-penalties.md)
