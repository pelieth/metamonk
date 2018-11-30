This repository contains a modified, working-in-progress MetaMask that supports "pluggable identities".

# Motivation

As MetaMask users, we often want programmable interfaces that a plain wallet account cannot provide. Wallet contracts (like multisig contracts in Ethereum), for example, is the simplest way to storage and transfer ethers in practice. However, when it comes to interacting with other contracts, users may find it much harder to use this kind of contracts, because an account only refers to a key pair in current MetaMask for now.

# Proposal

This fork introduces the concept of "identity", and "pluggable" means that it is an opt-in feature. When the proxy mode is enabled, a user can switch to one of its identities on-the-fly, just like it can switch to different account/network before. It can make transactions to an address, and this MetaMask-fork can transparently transform it to a suitable function call to the proxy contract on behalf of the original user. The proxy contract is expected to use that information to perform the requested transaction afterwards.

# Implementation

MetaMask exposes the wallet address of the selected account at `web3.currentProvider.accounts` after unlocking (in MetaMask < 5). Dapps use this address, the identity of the user, to issue signing requests or transactions. There is an assert in the code that the "from" address of a transaction must match the selected address in MetaMask, to ensure that the user is able to sign that transaction later ([reference](https://github.com/MetaMask/metamask-extension/blob/v4.16.0/app/scripts/controllers/transactions/index.js#L169-L172)). If we consider a proxy contract also an identity of a user, this restriction no longer applies. As a result, we implement this feature as outlined:

1. The user turns on the proxy mode option by toggling it in a dropdown menu
2. The user adds a proxy contract `(nickname, proxy address, function hash)` and switch to this identity in the side menu
3. MetaMask changes the exposed address to the address **of the selected identity** (contract address)
4. If the Dapp wants to initialize a transaction, modify its destination to the address of the proxy contract, and encode parameters according to the type of that proxy contract, supplied by the user
5. A popup is displayed as normal. The only differences are transformed `amount` and `data`
6. The user clicks "sign", signing the transaction with the original account
7. The transaction is sent as before

If (1) the user turns off the proxy mode option, or (2) it selects the non-contract address, no data is transformed, and the workflow should be identical as if this feature has not exist. This switch ensures its backward-compatibility.

The technical notes made while developing [can be found here](https://www.notion.so/qbane/MetaMask-Metamonk-dev-74cb6725e8344b0da581e236c3dbcab1) (on Notion).

# Demo

The MetaMask-fork provides testing builds (Chrome and Firefox only) as zip packages. It is strongly encouraged to use a clean environment to minimize the interferences.

* 4.14.0: https://github.com/andy0130tw/metamask-extension/releases/tag/v4.14.0-metamonk
