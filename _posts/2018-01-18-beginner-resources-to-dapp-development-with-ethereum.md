---
layout: post
title: "Beginner Resources for DApp Development with Ethereum"
date: 2018-01-17 21:13:00 +0800
categories: [ethereum, dapp, solidity]
comments: true
---

I've found that what's missing for developers coming to **Ethereum** are really good educational resources that cover everything from start to finish. In this post I will try to outline what I think is a good process of getting started developing **DApps** as well as link to a number of sites, articles and tools that I found helpful in the past so you hopefully won't have to spend as much time searching and can just focus on learning.

<!--more-->

I'm going to assume you already know (roughly) what Ethereum is, if not I suggest checking out the [official website](https://www.ethereum.org) and [whitepaper](https://github.com/ethereum/wiki/wiki/White-Paper), although you don't need to fully understand the latter in order to develop on the Ethereum platform.

On a sidenote, one thing that really helped me to understand **Proof of Work**, a core concept of current **Blockchain** technology, was building a simple Blockchain implementation from scratch. You can follow [this article](https://hackernoon.com/learn-blockchains-by-building-one-117428612f46) to do that, although I suggest implementing it yourself (maybe even in a different language) instead of just copy-pasting the code. 

After that generic introduction to Blockchain and Ethereum a great resource for learning **Solidity**, the main language used for **Smart Contracts on Ethereum**, is [CryptoZombies](https://cryptozombies.io). It will take you through learning the language and writing your first Smart Contracts to develop a simple game, and is expanding with new lessons roughly every two weeks. For supporting resources you can refer to the [Solidity docs](https://solidity.readthedocs.io).

If you want to try out Solidity in your browser, you can use [Remix](https://remix.ethereum.org) or [EthFiddle](https://ethfiddle.com).

Another important piece in the puzzle which you'd probably want to learn right after going through the Solidity resources or CryptoZombies course is [web3.js](https://github.com/ethereum/web3.js/) - **Ethereum's JavaScript API**. This is used to communicate with a local Ethereum node from your browser and interact with your Smart Contracts. At this point you should also install [MetaMask](https://metamask.io), a Chrome plugin that provides wallet functionality and lets you interact with your DApp's frontend through its web3.js-integration.

When it comes to actually developing your DApp, the [Truffle framework](http://truffleframework.com) is used by most DApps to handle common issues such as migrating your contracts and testing. When you set it up I suggest first going through their tutorial [Ethereum Pet Shop](http://truffleframework.com/tutorials/pet-shop). They also provide several [Truffle Boxes](truffle boxes ethereum) to quickly get started with the most common frontend libraries.

Once you have a basic DApp ready, even if it's just a tutorial app, you will want to understand how to use testnets to test and deploy those apps. During development you will likely use [restrpc](https://github.com/ethereumjs/testrpc) and later on switch to [geth](https://github.com/ethereum/go-ethereum/wiki/geth) in order to connect to the official testnets Rinkeby or Ropsten, both of which let you deploy your contracts and test them without using any real Ether. You could also use [Truffle's Ganache app](https://github.com/trufflesuite/ganache) which they use in their tutorial for local testing, although I've found testrpc to still be better suited for the time being.

For building a real DApp I also strongly recommend [OpenZeppelin](https://openzeppelin.org), a set of libraries to make Smart Contract development more secure and avoid common mistakes.

Or if you are interested in creating your own ERC20 token I suggest going through the [official docs](https://www.ethereum.org/token) as well as articles such as [this](https://steemit.com/ethereum/@maxnachamkin/how-to-create-your-own-ethereum-token-in-an-hour-erc20-verified) or [this](https://medium.com/@Alt_Street/create-your-own-ethereum-token-bfa6302084da).

Hopefully this helps a little bit to understand how all the pieces fit together. If you have any questions or suggestions let me know in the comments.
