# Confidence Coin whitepaper
*v1.0 last updated 2/15/2023([changelog](changelog/))*

{% include header.md %}

# Introduction
Confidence Coin is a decentralized cryptocurrency that utilizes a unique combination of [Flash consensus](/Flash-Consensus-algorithm/) and [Decentralized Trusted Party (DTP)](/dtp/) technology to provide fast and secure transactions with enhanced privacy features. It is designed to be the fastest cryptocurrency on the market, with the ability to process over a million transactions in a single block.
One of the main features of Confidence Coin is its DTP technology, which provides a level of privacy that is not available with most other cryptocurrencies. By aggregating transactions and only revealing the final balance of wallets, the details of who sent coins to whom are not recorded on the ledger. In addition, the Flash consensus mechanism ensures that transactions are confirmed quickly and securely.
In this whitepaper, we will explore the key features and benefits of Confidence Coin, including its fast transaction speeds, enhanced privacy features, and stable and self-sustaining ecosystem. We will also delve into the technical aspects of the Flash consensus mechanism and DTP technology, and provide an overview of the Confidence Coin incentive program. By the end of this paper, readers will have a comprehensive understanding of how Confidence Coin works and its potential to revolutionize the cryptocurrency industry.

# Flash Consensus
Flash Consensus is a unique consensus mechanism that powers Confidence Coin. It is designed to be a fast, efficient, and secure consensus mechanism that can process a high volume of transactions. Flash Consensus operates in a three-step process that validators run in a loop. This three-step process enables validators to achieve consensus efficiently.

The first step of Flash Consensus is Collect Blocks. Validators wait for each validator in the system to produce a block with the same ID, which is known as block N. The second step is Create a Snapshot. Validators examine the voting section of blocks N for transactions included in blocks N-1 and create a corresponding Snapshot (Snap N-1). The final step is Vote and Create a Block. Validators use Snap N-1 to cast votes on transactions in blocks N and create the next block (block N+1).
The Flash Consensus mechanism allows for the creation of new Snapshots with each block cycle, ensuring a complete system record is always maintained. The simplicity of this process makes it efficient and allows for fast processing of a high volume of transactions. The use of Flash Consensus in Confidence Coin ensures that the blockchain is secure and can process transactions in a timely and efficient manner.

# InterPlanetary File System (IPFS) and Confidence Coin
[The InterPlanetary File System (IPFS)](https://docs.ipfs.tech/concepts/what-is-ipfs/#what-is-ipfs) is a distributed file system with a global namespace for files. It is ideal for the Confidence Coin network. Confidence Coin will use IPFS to share blocks and snapshots among validators.

Confidence Coin uses a combination of three networks to operate: IPFS and [IPNS](https://docs.ipfs.tech/concepts/ipns/#mutability-in-ipfs), [libp2p](https://libp2p.io/) [gossip pubsub](https://github.com/libp2p/specs/tree/master/pubsub), and HTTP. Validators publish new blocks over IPFS and announce the [Content Identifier (CID)](https://docs.ipfs.tech/concepts/content-addressing/) over pubsub. The CID must be signed with [Curve25519](https://en.wikipedia.org/wiki/Curve25519). Only hashes signed by validators from the Validators list are considered valid, and other messages are dropped to prevent Distributed Denial of Service (DDOS) attacks.

Each validator has a Validator ID, which is their Curve25519 public key. It is then used for signing a block and also serves as their Validator Wallet address for collecting block rewards. The IPNS record name is generated from the Validator ID and the CID is sent over pubsub by the validator when they publish a new block.

After each block cycle, validators generate a snapshot of the entire blockchain and publish it over their IPNS. The sequence (version number) must be the block number. The IPNS must link to a directory that has a new file for each snapshot. Note that there is no requirement to have the entire blockchain there.

The snapshots directory and the files inside are not duplicated times the number of validators. There is a consensus about the content of those snapshots. All validators produce the same files that result in the same CIDs.

Using IPFS and IPNS serves multiple purposes for Confidence Coin. First, a snapshot is the transaction acknowledgment that it is committed. Second, it reduces the amount of storage validators need to operate. They don’t need the entire blockchain, only the last snapshot. Third, it allows efficient encoding. After building a snapshot, each wallet receives a short ID inside a DTP. This ID is used in DTP update transactions to reduce the block size.

Overall, IPFS and IPNS are essential components of the Confidence Coin network that enable the system to function in a fast, secure, and efficient manner.

# DTP Technology
Decentralized Trusted Party (DTP) is a proprietary tool developed for secure and confidential transactions through Confidence Coin blockchain. DTP aggregates multiple transactions and only reveals the final balance. This ensures that there is no record of origin, destination, and transacting balance. All transactions are required to be made via DTP, which operates as a legal company, allowing the government to monitor and prevent fraudulent and illegal activities in cryptocurrency transactions.

The process of using DTP involves a wallet declaring itself as a DTP through the Confidence Coin blockchain, after which users can entrust their wallets to the DTP through the same blockchain. Users can then conduct transactions with one another and with external entities through the DTP's external monetary system. During an update transaction, the DTP has the capability to create new wallets and transfer the users' coins to these wallets if necessary. The DTP performs an update transaction on the blockchain to synchronize the wallet balances with the balances recorded on the blockchain.

To ensure the integrity of the system, certain restrictions must be followed. The total number of coins in the system must remain unchanged both before and after a transaction, and the wallet balances outside of the DTP can only increase. Additionally, DTP must perform an update transaction every x days, as it declared in the blockchain, or risk losing the trust of all wallets. Also, wallets can revoke their trust from the DTP at any time through the blockchain.

DTP not only offers increased privacy and security but also safety. By utilizing DTP, users can rest assured that their transactions are protected from malicious actors and in compliance with legal regulations.

# Holder-Sensitive Mechanism
The Confidence Coin blockchain offers a unique reward system for its holders, which incentivizes them to hold their coins for longer periods and add more coins to their holdings.
To encourage long-term holding, the Confidence Coin blockchain uses a baseline calculation that takes into account the number of coins held and the duration of holding. Each additional month of holding is multiplied by a factor of 1.016, which results in higher rewards for holders who keep their coins on hold for longer periods. This mechanism ensures that holders are incentivized to hold their coins for as long as possible, as the longer they hold, the higher the potential rewards.
To further incentivize holders to add more coins to their holdings, the Confidence Coin blockchain implements a linear penalty mechanism that decreases rewards each day. This mechanism ensures that holders receive zero rewards on the last day of holding, encouraging them to add more coins to their holdings and hold them for longer periods. This mechanism helps to maintain a stable and consistent demand for Confidence Coin and prevent excessive selling and market volatility.
In summary, the Confidence Coin blockchain offers a unique and innovative reward system for its holders, which incentivizes them to hold their coins for longer periods and add more coins to their holdings. This mechanism helps to ensure the stability and growth of the Confidence Coin blockchain network.

# Government Incentive Program
The Confidence Coin blockchain is not only focused on incentivizing its users and developers, but it also has a Government Incentive Program that is designed to promote and encourage government adoption of the platform. This program is designed to promote Confidence Coin blockchain as a government-friendly blockchain that can benefit the country and its people.
The program works by allocating 1% of the block rewards to the Confidence Coin Foundation Wallet. These coins are then used to invest back into the development of the Confidence Coin blockchain, and to further promote its adoption and growth. However, the coins are also utilized as part of the Government Incentive Program.
When users create wallets on the Confidence Coin blockchain, they are required to provide the country that will receive the block reward. When a new block is mined, before the reward is split between validators and holders, it is allocated to the Government Incentive Program. Each government that adopts Confidence Coin will receive the 1% reward instead of the Confidence Coin Foundation.
The reward is then divided between the governments based on the percentage of coins that were transacted in the block. Whenever a wallet balance is increased, it counts toward the country that the wallet chose, thereby promoting the adoption and use of Confidence Coin blockchain in that country.
The Government Incentive Program is an innovative and effective way to encourage government adoption of Confidence Coin blockchain, which can lead to a more transparent and efficient economy. By providing incentives to governments, Confidence Coin is promoting the use of blockchain technology in a way that benefits both the government and its citizens.

# Capacity
Confidence Coin has a hard-cap of 1,798,974,000,000 coins, which was based on the dollar circulation value in January 2020, before the COVID-19 pandemic. The daily reward for mining new coins is calculated using the following function, where X is the day starting from 1:

$f(X)=Max(0,-2,700X + 98,563,309)$

The coin supply will be fully mined in approximately 36,505 days, or roughly 100 years. This ensures that there is a finite supply of Confidence Coins and that the value of each coin may increase over time due to scarcity.

# Use Cases
Confidence Coin is a versatile blockchain platform with many potential use cases. Here are a few examples:
1. Company: Online Retailer Use Case: Customers can make purchases using DTP. The retailer can then use DTP to aggregate transactions and only reveal the final balance, providing greater privacy and security for their customers.
2. Company: Health Insurance Provider Use Case: Customers can submit claims using DTP, which would allow for greater privacy and security of their personal and medical information. The insurance provider could use DTP to aggregate claims and only reveal the final balance, providing greater efficiency and privacy for their customers.
3. Company: Ride-Sharing Service Use Case: Customers can pay for rides using DTP, providing a secure and private way to make payments. The ride-sharing service could use DTP to aggregate payments and only reveal the final balance, providing greater efficiency and privacy for their customers.
4. Company: Music Streaming Service Use Case: Customers can subscribe to the service using DTP, providing a secure and private way to make payments. The streaming service could use DTP to aggregate subscription payments and only reveal the final balance, providing greater efficiency and privacy for their customers.
5. Company: Real Estate Agency Use Case: Customers can make property transactions using DTP, providing greater security and privacy for their transactions. The real estate agency could use DTP to aggregate transactions and only reveal the final balance, providing greater privacy and security for their customers.
Overall, Confidence Coin's fast and secure transaction processing, low fees, and unique incentive system make it a compelling platform for a wide range of use cases.

# Conclusion
In conclusion, Confidence Coin is a revolutionary blockchain platform that offers a unique reward system to its validators and holders. Its Decentralized Trusted Party (DTP) provides a secure and transparent way for businesses and individuals to transact with each other, while its Flash consensus mechanism enables the processing of over a million transactions in a single block. The government incentive program, funded by 1% of the block rewards, aims to promote the adoption of Confidence Coin by giving the reward to the governments based on the percentage of coins transacted in the block. Confidence Coin is still in development, but with its innovative technology and incentives, it has the potential to become one of the most successful blockchain platforms in the world. The platform also utilizes IPFS for file storage, providing a more efficient and decentralized way to store data. With its unique features and strong community support, Confidence Coin is poised to play a major role in shaping the future of blockchain technology.
