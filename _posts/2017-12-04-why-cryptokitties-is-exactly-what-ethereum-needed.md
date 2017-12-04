---
layout: post
title:  "Why Cryptokitties is Exactly What Ethereum Needed"
date:   2017-12-04 19:57:00 +0800
categories: [ethereum, cryptocurrency, dapp]
comments: true
---

[Cryptokitties](https://www.cryptokitties.co) came out less than a week ago and it has already taken the Ethereum-world by storm. If you don't know what it is, it's basically a simple game on the [Ethereum](https://www.ethereum.org) blockchain that let's you collect, breed, buy and sell cats. What makes it more interesting is that each cat has a set of "cattributes" that determine its fur pattern, colors, length of its whiskers or shape of their hair and much more. It also gives them a hidden set of genes, which can - with the right cat-breeding-partner - result in new and possibly unique combinations.

![image-title-here](/img/posts/cryptokitties.png)

Breeding cats costs a small fee and takes a certain amount of time, which increases the more often a cat gives birth or sires a kitten, and also depends on their generation. Earlier generations generally have lower cooldowns. The genes combined with a cat's generation, cooldown and looks give it value as a collectible or a breeding cat, to produce more valuable offspring.

When I started "playing" Cryptokitties, cats could be bought for quite cheap but the prices have since then exploded (similar to Bitcoin or Ethereum themselves, actually). The "generation 0" cats that are added to the game in 15-minute intervals by the developers get bought out instantly and currently go for the ETH-equivalent of several thousand USD. The most valuable and unique "Genesis" cat, the one that started it all, sold for the equivalent of nearly 120000 USD at the time of writing.

I don't want to talk too much about the money-side of this. **What I find more interesting is how this simple game is really showing off what Ethereum can and can't do, and how it exposes more people to Blockchain technology.**

It does so by making something that is normally hard to understand (blockchain, proof of work, mining, smart contracts, decentralized applications, gas, etc.) accessible and attractive to the masses, who have probably never heard or cared about Ethereum before, by wrapping it up in a cute little cat-package.

**It gamifies managing your first Ethereum wallet, sending transactions and interacting with smart contracts**, to a point where non-technical people in my social circles are now happily changing gas prices or gas limit in their [metamask](https://metamask.io) popups in order to get their cat auctions processed by the miners. Something that would have been hard to explain to new-comers before now suddenly makes sense to them.

**It also shows off some of the strengths of the Ethereum blockchain** (and by extension similar or future blockchains), by demonstrating the type of decentralized apps that are possible to build. As a sidenote, these DApps will fix something that has bugged me in web development for a long time: user authentication. Something you had to implement in nearly every web project to varying headache-inducing degrees - although some things like Devise makes this easier, some things in the Javascript world make it harder than necessary - is now no longer needed at all. When you interact with the Cryptokitties DApp you identify using public-key cryptography. Thanks to your wallets private key (for example when using metamask), we can now get rid of user registration, login and "forgot your password"-emails. You can still link up your account with a nickname and email if the DApp wants to use these, but the next time you visit the app's site you will still be "logged in" as long as you use the same metamask account.

Of course users have to be educated how to safely manage this form of identity and the assets (kittens!) that are tied to it. If they lose access to their wallet and don't have a secure backup of their private key or mnemonic phrase it's impossible to help them to get it back.

On the other hand, once most people understand this, Blockchain tech like this enables them to manage their digital assets safely and permanently. Unlike other online games where you can get banned or risk that the company running the game shuts down the servers, here you will still be the owner of those cats (depending how the contracts are implemented). This means they have real collector value and this tech can be used for any other asset in the future.

**However it has also demonstrated an already known weakness of Ethereum:** Since the start of the game the amount of pending transactions on the blockchain has increased dramatically. This simple, little cat DApp is slowing down the entire network. Solutions for this have already been proposed some time ago and are actively in the works, but this has given some urgency to the matter and it will be interesting to see how it can be solved before this causes serious issues for Ethereum as a whole.
