# Developer guide

This document covers tips and guidelines to help you to understand how Osmosis works and efficiently navigate the codebase of Osmosis Core, the official Golang reference implementation of the Osmosis node software.

::: tip Recommendation
Osmosis Core is built using the [Cosmos SDK]( https://cosmos.network/sdk), which provides a robust framework for constructing blockchains that run atop the [Tendermint](https://tendermint.com/) Consensus Protocol.

It is highly recommended that you review these projects before diving into the Osmosis developer documentation, as they assume familiarity with concepts such as ABCI, Validators, Keepers, Message Handlers, etc.
:::

## How to Use the Docs

As a developer, you will likely find the [Module](./modules/) section the most informative. Each specification starts out with a short description of the module's main function within the architecture of the system, and how it contributes in implementing Osmosis's features.

Beyond the introduction, each module will lay out a more detailed description of its main process and algorithms, alongside any concepts you may need to know. It is recommended that you start here to understand a module, as it is usually cross-referenced with more specific sections deeper in the spec such as specific state variables, message handlers and functions that may be of interest.

The current function documentation is not an exhaustive reference, but has been written to be a reference companion for those needing to work directly with the Osmosis Core codebase or understand it. The important functions in each module have been covered, and many of the trivial ones (such as getters and setters) have been left out for economy. Other places where module logic can be found is within either the message handler, or block transitions (begin-blocker and end-blocker).

At the end, the specification lists out various module parameters alongside their default values with a brief explanation of their purpose, and associated events / tags and errors emitted by the module.


### Inherited modules

Many of the modules in Osmosis Core are inherited from Cosmos SDK and are configured to work with Osmosis through customization in either genesis parameters or by augmenting their functionality with additional code.

## Block Lifecycle

The following processes get executed during each block transition:

### Begin block

1. Distribution

   - Distribute rewards for the previous block

2. Slashing
   - Checking of infraction evidence or downtime of validators for double-signing and downtime penalties.

### Process messages

3. Messages are routed to the modules that are responsible for working them and then processed by the appropriate Message Handlers.

### End block

4. Crisis

   - Check all registered invariants and assert they remain true

5. Governance

   - Get rid of inactive proposals, check active proposals whose voting periods have ended for passes, and run registered proposal handler of the passed proposal.

7. Market

   - [Replenish](modules/spec-market.md#end-block) liquidity pools, **allowing spread fees to decrease**.

8. Treasury

   - At the end of `epoch`, update indicators, burn seigniorage, and recalibrate monetary policy levers (tax-rate, reward-weight) for the next epoch.

9. Staking
   - The new set of active validators is determined from the top 130 Osmo stakers. Validators that lose their spot within the set start the unbonding process.