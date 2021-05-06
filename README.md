# Anchor-earn

Anchor-earn is a client SDK for building applications that can interact with the earn functionality of Anchor Protocol from within JavaScript runtimes. 


> **NOTE**
This SDK only supports the earn functionalities of anchor protocol and cannot be used for other functionalities like bond or borrow.

## Table of Contents <!-- omit in toc -->
- [Getting Started](#getting-started)
    - [Requirements](#requirements)
    - [Installation](#installation) 
    - [Dependencies](#dependencies)
    - [Test](#test)
- [Usage](#usage)
- [Fund Account with UST](#fund-account-with-ust)
- [Examples](#examples)
    - [Executor](#executor)
    - [Querier](#querier)
- [CustomSigner](#customsigner)
- [Logabble](#loggable)
- [License](#license)

## Getting Started
A walk through of the steps to get started with the Anchor-earn SDK alongside with a few use case examples are provided below.

## Requirements

- Node.js 12+
- NPM

## Installation
Anchor-earn is available as a package on NPM and it is independent from other Terra and Anchor SDKs.\
To add to your JavaScript project's `package.json` as a dependency using preferred package manager: 
```bash
npm install -S @anchor-protocol/anchor-earn
```
or
```bash
yarn add @anchor-protocol/anchor-earn
```

## Dependencies
Anchor earn uses only Terra.js as a dependency. To get set up with the required dependencies, run:
```shell
# debug
yarn install
```
## Test
Anchor earn provides extensive tests for data classes and functions. To run them, after the steps in [Dependencies](#dependencies):
```shell
# debug
yarn test
```
## Usage

### `Account` object
Anchor-earn provides a facility to create a wallet on the Terra blockchain.\
This functionality is accessible through the `Account` object.
```ts
const account = new Account();
```  
> **NOTE** It is crucial to store or write down your account information before doing any interactions with the SDK. A user can have access to this info by printing the account.
```ts
console.log(account.toData());
```

```
      Account {
        accAddress: 'terra15kwnsu3a539l8l6pcs6yspzas7urrtsgs4w5v4',
        publicKey: 'terrapub1addwnpepq2wc706a537ct954wfxxxwe8yhrqpuwxs2ejykya9jadwk0jj3ud5935v95',
        accessToken: 'TERRA_m2rIfcnwpIZXlxrdjpcSj7VOZHoRj8Sc1Wv8C9F09vY=',
        MnemonicKey: 'weird rent soft alien write globe october wish arena cream agree toe gain chunk club clip green night hobby keep void garden help diagram'
      }

```
`accessToken` is essential for later usage.

### `Wallet` and `MnemonicKey` object
`Wallet` and `MnemonicKey` object are borrowed from Terra.js. Users have access to them in Anchor earn without dependency on Terra.js.

In case users have a previous account on the Terra chain, they can use their private key and MnemonicKey to recover their keys.
 ```ts
import { Wallet, MnemonicKey } from '@anchor-protocol/anchor-earn';

    const account = new MnemonicKey({
      mnemonic:
        '...',
    });
```

Additional usage of `Wallet` object is that it can be used for [customSigner](#customsigner). An example is provided in  [customSigner](#customsigner) section.
### `AnchorEarn` object
Anchor-earn provides facilities for two main use cases: 

- execute: Signs the message and broadcasts it using Terra.js
- query: Runs a series of smart contract and chain queries through LCD

Both of these functions are accessible through the `AnchorEarn` object. 

To create the `AnchorEarn` object.
```ts
    const anchorEarn = new AnchorEarn({
      chain: CHAINS.TERRA,
      network: NETWORKS.TESTNET,
      accessToken: account.accessToken,
    });
```
The above example uses the `Account` object for instantiating `anchor-earn`.

For the case that a user has a previous account on the Terra chain, the user can recover their key using `MnemonicKey` and use the following code to instantiate `AnchorEarn`.
 ```ts
   import { MnemonicKey } from '@anchor-protocol/anchor-earn';
    const account = new MnemonicKey({
      mnemonic:
        '...',
    });

    const anchorEarn = new AnchorEarn({
      chain: CHAINS.TERRA,
      network: NETWORKS.TESTNET,
      privateKey: account.privateKey,
    });
```
## Fund Account with UST
For Terra testnet (tequila-0004), users can top up their balance with UST using [faucet](https://faucet.terra.money/).

## Examples
As mentioned above, `AnchorEarn` helps execute messages and query the state of the market and account. The following examples show how to use the object.

## Executor

`AnchorEarn` executor has three functionalities:
- deposit: deposit funds in the Anchor protocol
- withdraw: withdraw previously deposited funds
- send: transfer `UST` and `AUST` to other accounts

The following code snippets show how to use the `AnchorEarn` object.

> **NOTE**: Currently, Anchor-earn supports the deposit of the`UST` currency only.

### Deposit 
To deposit funds in the Anchor Protocol, use the following example:
```ts
    const deposit = await anchorEarn.earn.deposit({
      amount: '...', // amount in natural decimal e.g. 100.5. The amount will be handled in macro.
      currency: DENOMS.UST,
    });
```

### Withdraw
To withdraw funds from the protocol, use the following example:
```ts
    const deposit = await anchorEarn.earn.withdraw({
      amount: '...', // amount in natural decimal e.g. 100.5. The amount will be handled in macro.
      currency: DENOMS.UST,
    });
```

### Send
To send `UST` and `AUST` to other accounts, use the following example: 
<br/>
<sub>(For this functionality, the `AUST` denom is also supported.) </sub>
```ts
 const sendUst = await anchorEarn.earn.send(DENOMS.UST, {
      recipient: 'terra1....',
      amount: '...', // amount in natural decimal e.g. 100.5. The amount will be handled in macro.
    });
```
## Querier
`AnchorEarn` querier facilitates both querying smart contracts and the chain. There are two queries provided by the `AnchorEarn` object:
- balance: query user balance and user deposit based on currency
- market: return the state of the specified currency's market

If a user wishes to use only the queries, there is no need to instantiate the object as explained [here](#anchorearn-object);
instead, they can provide the address for queries as demonstrated by the following examples: 
### Balance
To get the current state of an account, use the following example: 
```ts
 const anchorEarn = new AnchorEarn({
      chain: CHAINS.TERRA,
      network: NETWORKS.TESTNET,
    });

const userBalance = await anchorEarn.earn.balance({
      currencies: [DENOMS.UST],
      address: 'terra1...'
    });
```
### Market
To get the current state of the market, use the example below:
```ts
    const market = await anchorEarn.earn.market({
      currencies: [DENOMS.UST],
    });
```
## CustomSigner
Anchor-earn also provides users with the functionality to sign transactions and leave the signed transaction to the SDK to perform the broadcasting.
 
 `CustomSigner` is a callback function with which the users can sign `deposit`, `withdraw`, and `send` transactions.
  
The following code snippet specifies an example of the `CustomSigner` usage:

> **Note**: The address must be specified. 
```ts
const deposit = await anchorEarn.earn.deposit({
      amount: '0.01',
      currency: DENOMS.UST,
      customSigner: async (tx: Msg[]) => {
        const account = new MnemonicKey({
          mnemonic:
            '...',
        });

        const wallet = new Wallet(
          new LCDClient({
            URL: 'https://tequila-lcd.terra.dev',
            chainID: 'tequila-0004',
          }),
          account,
        );

        return await wallet.createAndSignTx({
          msgs: tx,
          gasAdjustment: 2,
          gasPrices: { uusd: 0.15 },
        });
      },
      address: 'terra1us9cs88cxhcqclusvs4lxw0pfesc8y6f44hr3u',
    });
```
## Loggable 
For seeing the progress of the transaction on the chain, `loggable` is provided. The following code shows how to use it:
```ts
    const deposit = await anchorEarn.earn.deposit({
      amount: '...',
      currency: DENOMS.UST,
      log: (data) => {
        console.log(data);
      }
    });
```

## License
This software is licensed under the Apache 2.0 license. Read more about it [here](./LICENSE).

© 2021 Anchor Protocol