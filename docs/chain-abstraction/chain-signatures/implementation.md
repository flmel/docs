---
id: implementation
title: Implementing Chain Signatures
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import {CodeTabs, Language, Github} from "@site/src/components/codetabs"

Chain signatures enable NEAR accounts, including smart contracts, to sign and execute transactions across many blockchain protocols.

This unlocks the next level of blockchain interoperability by giving ownership of diverse assets, cross-chain accounts, and data to a single NEAR account.

:::info

This guide will take you through a step by step process for creating a Chain Signature.

⭐️ For complete examples of a NEAR account performing transactions in other chains:

- [CLI script](https://github.com/mattlockyer/mpc-script)
- [web-app example](https://github.com/near-examples/near-multichain)
- [component example](https://test.near.social/bot.testnet/widget/chainsig-sign-eth-tx)

:::

---

## Create a Chain Signature

There are five steps to create a Chain Signature:

1. [Deriving the Foreign Address](#1-deriving-the-foreign-address) - Construct the address that will be controlled on the target blockchain
2. [Creating a Transaction](#2-creating-the-transaction) - Create the transaction or message to be signed
3. [Requesting a Signature](#3-requesting-the-signature) - Call the NEAR `v1.signer` contract requesting it to sign the transaction
4. [Formatting the Signature](#4-formatting-the-signature) - Reconstruct the signature from the MPC service's response
5. [Relaying the Signed Transaction](#5-relaying-the-signature) - Send the signed transaction to the destination chain for execution

![chain-signatures](/docs/assets/welcome-pages/chain-signatures-overview.png)
_Diagram of a chain signature in NEAR_

:::info MPC Contracts

If you want to try things out, these are the smart contracts available on `testnet`:

- `v1.signer-prod.testnet`: [MPC signer](https://github.com/near/mpc/tree/v0.2.0/contract) contract, latest release, made up of 8 MPC nodes

:::

:::info MPC mainnet contracts

- `v1.signer`: [MPC signer](https://github.com/near/mpc/tree/v0.2.0/contract) contract, latest release, made up of 8 MPC nodes
:::

---

## 1. Deriving the Foreign Address

Chain Signatures use [`derivation paths`](../chain-signatures.md#derivation-paths-one-account-multiple-chains) to represent accounts on the target blockchain. The external address to be controlled can be deterministically derived from:

- The NEAR address (e.g., `example.near`, `example.testnet`, etc.)
- A derivation path (a string such as `ethereum-1`, `ethereum-2`, etc.)
- The MPC service's public key (see the tip below for the MPC service public keys)

We provide code to derive the address, as it's a complex process that involves multiple steps of hashing and encoding:

<Tabs groupId="code-tabs">
  <TabItem value="Ξ EVM">
    <Github language="js"
      url="https://github.com/near-examples/near-multichain/blob/main/src/components/EVM/EVM.jsx" start="89" end="92" />

</TabItem>

<TabItem value="₿ Bitcoin">
    <Github language="js"
      url="https://github.com/near-examples/near-multichain/blob/main/src/components/Bitcoin.jsx" start="43" end="46" />

</TabItem>

</Tabs>

We recommend hardcoding the derivation paths in your application to ensure the signature request is made to the correct account

:::tip

Here you can find MPC service public keys:

- **v1.signer-prod.testnet** ([testnet](https://testnet.nearblocks.io/address/v1.signer-prod.testnet)): `secp256k1:4NfTiv3UsGahebgTaHyD9vF8KYKMBnfd6kh94mK6xv8fGBiJB8TBtFMP5WWXz6B89Ac1fbpzPwAvoyQebemHFwx3`

- **v1.signer** ([mainnet](https://nearblocks.io/address/v1.signer)): `secp256k1:3tFRbMqmoa6AAALMrEFAYCEoHcqKxeW38YptwowBVBtXK1vo36HDbUWuR6EZmoK4JcH6HDkNMGGqP1ouV7VZUWya`

:::

:::info

The same NEAR account and path will always produce the same address on the target blockchain.

- `example.near` + `ethereum-1` = `0x1b48b83a308ea4beb845db088180dc3389f8aa3b`
- `example.near` + `ethereum-2` = `0x99c5d3025dc736541f2d97c3ef3c90de4d221315`

:::

---

## 2. Creating the Transaction

Constructing the transaction to be signed (transaction, message, data, etc.) varies depending on the target blockchain, but generally it's the hash of the message or transaction to be signed.


<CodeTabs>

  <Language value="Ξ EVM" language="js">
  <!-- In Ethereum, constructing the transaction is simple since you only need to specify the address of the receiver, and any necessary data for the transaction. -->
      <Github language="js"
      url="https://github.com/near-examples/near-multichain/blob/main/src/components/EVM/FunctionCall.jsx"
      start="30" end="39"
      fname="FunctionCall.jsx"/>
       <Github language="js"
      url="https://github.com/near-examples/near-multichain/blob/main/src/components/EVM/Transfer.jsx"
      start="16" end="23"
      fname="Transfer.jsx" />
  </Language>
 
      
 <Language value="₿ Bitcoin" language="js">

<!-- In bitcoin, you construct a new transaction by using all the Unspent Transaction Outputs (UTXOs) of the account as input, and then specify the output address and amount you want to send. -->
        <Github language="js"
      url="https://github.com/near-examples/near-multichain/blob/main/src/components/Bitcoin.jsx"
      start="62" end="67"
      fname="Bitcoin.jsx"
       />
  </Language>
</CodeTabs>

:::tip
If you're a Rust developer, you can use the [Omni Transaction](https://github.com/near/omni-transaction-rs) Rust library to build transactions easily for different blockchains (like Bitcoin and Ethereum) inside NEAR contracts.
:::

---

## 3. Requesting the Signature

Once the transaction is created and ready to be signed, a signature request is made by calling `sign` on the [MPC smart contract](https://github.com/near/mpc-recovery/blob/f31e39f710f2fb76706e7bb638a13cf1fa1dbf26/contract/src/lib.rs#L298).

The method requires two parameters:

  1. The `transaction` to be signed for the target blockchain
  2. The derivation `path` for the account we want to use to sign the transaction

<Tabs groupId="code-tabs">
  <TabItem value="Ξ EVM">
    <Github language="js"
      url="https://github.com/near-examples/near-multichain/blob/main/src/components/EVM/EVM.jsx"
      start="110" end="125" />

</TabItem>

  <TabItem value="₿ Bitcoin">
    <Github language="js"
      url="https://github.com/near-examples/near-multichain/blob/main/src/components/Bitcoin.jsx"
      start="74" end="97" />

For bitcoin, all UTXOs are signed independently and then combined into a single transaction.

</TabItem>

</Tabs>

<details>

  <summary> Deposit amount </summary>

  In this example, we attach a deposit of 0.05 $NEAR for the signature request. The transaction may fail if the network is congested since the deposit required by the MPC service scales linearly with the number of pending requests, from 1 yoctoNEAR to a maximum of 0.65 $NEAR. Any unused deposit will be refunded and if the signature fails, the user will be refunded the full deposit.

  As an alternative, the MPC contract provides an [`experimental_signature_deposit()`](https://github.com/near/mpc/blob/develop/API.md#experimantal_signature_deposit) method to check the current deposit required.
  Keep in mind that this could provide an unreliable value, since the amount will likely change between the time of the check and the time of the request.

</details>

:::info

The contract will take some time to respond, as the `sign` method [yields execution](/blog/yield-resume), waiting for the MPC service to sign the transaction.

:::

---

## 4. Formatting the Signature

The MPC contract will not return the signature of the transaction itself, but the elements needed to rebuild the signature matching the target blockchain's format.

This allows the contract to generalize the signing process for multiple blockchains.

<Tabs groupId="code-tabs">
  <TabItem value="Ξ EVM">
    <Github language="js"
      url="https://github.com/near-examples/near-multichain/blob/main/src/components/EVM/EVM.jsx"
      start="126" end="132" />

In Ethereum, the signature is formatted by concatenating the `r`, `s`, and `v` values returned by the contract.

</TabItem>

<TabItem value="₿ Bitcoin">
        <Github language="js"
      url="https://github.com/near-examples/near-multichain/blob/main/src/components/Bitcoin.jsx"
      start="98" end="104" />

In Bitcoin, the signature is formatted by concatenating the `r` and `s` values returned by the contract.

</TabItem>

</Tabs>

---

## 5. Relaying the Signature

Once we have reconstructed the signature, we can relay it to the corresponding network. This will once again vary depending on the target blockchain.

<Tabs groupId="code-tabs">
  <TabItem value="Ξ EVM">
      <Github language="js"
      url="https://github.com/near-examples/near-multichain/blob/main/src/components/EVM/EVM.jsx"
      start="150" end="150" />

</TabItem>

<TabItem value="₿ Bitcoin">
    <Github language="js"
      url="https://github.com/near-examples/near-multichain/blob/main/src/components/Bitcoin.jsx"
      start="124" end="124" />

</TabItem>

</Tabs>

:::info
⭐️ For a deep dive into the concepts of Chain Signatures see [What are Chain Signatures?](../chain-signatures.md)

⭐️ For complete examples of a NEAR account performing Eth transactions:

- [web-app example](https://github.com/near-examples/near-multichain)
- [component example](https://test.near.social/bot.testnet/widget/chainsig-sign-eth-tx)

:::
