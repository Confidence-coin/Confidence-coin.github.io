# Confidence Coin - Tech spec
{% include header.md %}

This document will serve as a guide to building Confidence Coin, which comprises three main applications:
 - Confidence Coin CORE: This is the Kotlin-based validator app responsible for creating new blocks and snapshots.
 - Confidence Coin Bridge: This is a CORE GO helper app designed to facilitate calls to IPFS and report back to CORE.
 - Confidence Coin Wallet: This is the client wallet app, with the initial version available as an Android native app.
For a more detailed understanding of how Confidence Coin works, please refer to the whitepaper available at [whitepaper](/whitepaper/).

```
+-------------+        +-------------+       +-------------+  
|             |        |             |       |             |  
|   CORE #1   |--------|   CORE #2   |-------|   CORE #3   |  
|             |        |             |       |             |  
|  +-------+  |        |  +-------+  |       |  +-------+  |  
|  |Bridge |  |        |  |Bridge |  |       |  |Bridge |  |  
|  +-------+  |        |  +-------+  |       |  +-------+  |  
|             |        |             |       |             |  
+------+------+        +------+------|       +------+------+  
       |                      |  
       |                      |  
       |                      |  
       |                      |  
       |                      |  
       |                      |  
+------+------|       +------+------+  
|             |       |             |  
|   Wallet #1 |       |   Wallet #2 |  
|             |       |             |  
+------+------|       +------+------+                   
```
# CORE
Core works with two data types, `BlockData` and `SnapData` while executing the main loop of:
 - Collect blocks - Waiting to collect blocks via the Bridge
 - Create a snapshot - Look at the voting section of the collected blocks and the current Snapshot and apply it to the blocks from the previous block cycle to create the next Snapshot.
 - Create a new block - Look at the collected blocks and the new Snapshot to create a new block with all the pending transactions and vote for the transactions on the collected blocks.

## Block Data structure
-   `baseSnap` - A string that represents the CID (Content identifier) of the Snapshot data that was used to create this block - `(Snap blockId - 1)`.
-   `blockId` - A running block id integer.
-   `walletId` - The validator public key generated with  Curve25519. It is also used to create the `signature` property below that signs this block. It is also the wallet address of this validator to receive block rewards. Note that it's ok for the validator to opt-in a DTP with this wallet address.
-   `blockCreationTime` - A long that represents the time of block creation. Snapshot creation time is the median of the `blockCreationTime`
-   `topics` - An array of `Topic` objects. Transactions are a topic here.
-   `votes` - An array of `Vote` objects. Votes on topics; Only 
-   `timeouts` - An array of strings that represents the validators who didn't send a block in the expected time.
-   `signature` - A string that represents the validator signature of all the fields but the cId using the `blockId` public key.
-   `cId` - A string that represents the block CID (Content identifier).

## Snap data structure
 - `snapId` - The snapshot id, also referred to as the block id.
 - `creationTime` - Creation time is calculated as the media creation time of the blocks used to create this Snapshot
 - `snapCid` -  snapshot CID (Content identifier)
 - `validators` - an array of `ValidatorData` ordered by the validator wallet address. It contains HTTP API and scores for validators
 - `dTps` - an array of DTPs ordered by the DTP wallet address
 - `holders` - an array of holder wallets.
 - `transactionFees` - a long of transaction fees.

Coins in Confidence Coin can range up to 1,000,000,000,000 and have up to 6 decimal digits. They are represented in a long multiplied by 1,000,000. Ex: 6.5 coins are represented with 6,500,000


## DTP data structure
 - `lastSnapId` - The snap id used in the update transaction. This is used to prevent Validators from submitting outdated DTP transactions. 
 - `timeToNextUpdate` - The time to the next update calculate based on this snapshot creation time.
 - `updateTime` - How often should DTP perform update transactions
 - `walletAddress` - DTP wallet address. The DTP wallet address must be a member of this DTP. It cannot be assigned to another DTP.
 - `pendingMembers` - a list of wallets that are pending to become members of this DTP. They will become such after the next transaction update of their current DTP unless they choose to move into another DTP by then
 - `members` is an array of DTP members ordered by member wallet id. In addition, a member contains the Wallet balance. During the update transaction, members are referred by their index in this array to save on space.

## Voting system
The way validators vote on blocks is by submitting a Vote object in the block they produce. The vote object contains:
 - `blockCid` - a content identifier of a block. This is the only place in a block where other blocks are referenced.
 - `validator id` - the author of the block
 - `rejectionList` - list of integers where each item is the index of a topic item that is rejected by this validator. Items that are not on this list are approved.

There is one Vote object for each block even if there are no rejection items in order to reference the blocks.

## Establishing Consensus
A consensus is established during the construction of a new snapshot.
1. First, the blocks are tested for common snapshot parent. At least 2/3 of the validators must create a block with the same parent snapshot CID. If the parent snapshot is different from the one constructed by the current validator, it is time to sync and download the right Snapshot. 
2. After agreeing on a parent Snapshot, time to decide what blocks will be part of the next Snapshot. At least 2/3 of the validators must vote on each block that will be part of the next Snapshot.
3. After the blocks of the next Snapshot are established, they need to be normalized(ordering by CID) and examined. This will guarantee that all the validators produce the same Snapshot since the input for everyone is the same.
4. Finally, the Topics are examined. Topics with over 2/3 of the validators' votes are processed into a new Snapshot. 
Note that 2/3 could be 68.6 or 68. In both cases, 69 votes are required for consensus. 

## Block Topics
A topic object contains a list of `ApiObject's and a `topicId`. `ApiObject` is a generic object that contains a byte array and a signature. It is encoded via Java object stream API. One can decode it based on the `topicId`

Some topics originated in the Wallets, like opting requests or an update transaction. Wallets have access to snapshots. Inside the Snapshot, there is information about validators, and each validator exposes an API string. It could be a domain address or just an IP. Wallets use this information to send a topic request over HTTP directly to validators. The validator selection is made at random with some consideration in networking.

### Update Transaction topic
This is the most important topic. DTP sends an update transaction to update the balances of all its members as well as external members(For external members, the balance can only grow).

Due to external members' dependency, updating a balance has an execution order importance. To avoid this restriction, the DTP doesn't provide the final balance but only the balance delta. It makes the order of the execution nonimportant. The requirement is that the total delta for all the wallets is zero since DTP cannot create or destroy coins in the system.

#### Update Transaction Object
 - `snapId` - The update transaction snapId is used to prevent duplications. Only one update transaction is allowed per block cycle
- `walletAddress` - The wallet address of the DTP  
- `newWallets` - An array of strings of new wallets to be created 
- `memberWalletIds` - An array of integers, indexes of the wallet ids in the DTP 
- `externalWallets`  - An array of strings for external wallet addresses.
 - `balanceDeltas`  - A list of longs representing the balance deltas. It corresponds to the new wallets, then the members, and finally, the external wallet addresses. For example, the balance of the third item in `memberWalletIds` is equal to `newWallets` array size + 3.

I put a lot of thought into how to structure the update transaction and how to make it storage efficient for better networking. The reason external wallets don't use the same optimization as member wallets is due to the need to make this transaction Snap independent. Even if the DTP is not up to date with the most recent Snap, the transaction it will create will still be valid.

### Convert to DTP
A wallet that doesn't belong to any DTP can convert into DTP. After it becomes a DTP, its wallet address becomes a member of that DTP. DTP wallet address cannot leave the DTP. 

#### Convert to DTP object
 - `walletAddress` - the wallet address to be converted
 - `updateTimeHours` - how often will the DTP have to execute transaction update

### Opting
Each wallet can opt in or out of a DTP. When opting in, an automated opt-out request is sent to the current DTP. Another way to opt for a DTP is to ask the DTP to create a new wallet for you by only providing it with the public key.

#### Opting object
 - `walletAddress` - the wallet address to opt-in/out
 - `dtpId` - the DTP wallet address

To opt out, the `dtpId` needs to be "out."

### Validator scores
When waiting for blocks, the time to receive each block is recorded. Those times are used to score validators for performance. It impacts the block reward.

#### Scores object
 - `validatorIds` - an array of validator strings. 
 - `scores` - an array of integers of scores in seconds. In case of a timeout, the score is -1. Timeout has an initial value, but after each Snap, it is recalculated based on historical snaps to get better values. The goal is to remove slow validators, but the challenge is not to overdo it. I am still thinking about a good strategy for it.

### Hold
The Confidence Coin insensitive program allows wallets to go on hold for 1-60 months and get block rewards during this time. There are two inner insensitive here:
 - for each extra month holding, the baseline is calculated as the `walletBalance * (1.012 ^ holdingMonths)`
 - the baseline is reduced each day by `baseline/totaldays`

More details about how the Holds are in the reward section.

#### Hold object
 - `walleAddress` - current wallet address
 - `holdingMonths` - for how many months to hold.

### Update system settings
System settings include things like transaction fees and other values that are constant in the system and require voting to be changed. There is no special trigger for it. It requires people to coordinate to fire and vote. By default, validators are programmed to deny all system changes.

#### Settings Object
 - `validatorId` the validator who cast the vote
 - `key` - a key string
 - `value` - a value string

### Add/remove/update validator
Validators should be removed if they timeout to produce a block, underperform or break the rules. In case they break the rules, their stack should be damaged as well.

When a validator is removed due to a timeout or poor performance, it will autojoin back by producing a block without timing out that has less than 5% timeout scores for other validators. Other validators will keep track of it for 48 hours and vote to return the validator if it is able to produce the required block.

Validators who got a timeout score from more than 2/3 of the validators will be removed. Otherwise, the timeout scores are ignored; Validator who got a time score smaller than $\frac{median}{2}$ is removed for poor performance.

It is possible to verify the Snapshots produced by a validator. There are some valid reasons for validators to produce different snapshots(Like timeout from a tiebreaker validator who didn't timeout for anyone else). But even those snapshots must follow the rules if the validator produces an incorrect Snapshot or approves a transaction it shouldn't. There will be a trace left to verify that decision by other validators. In such a case, they can vote to remove this validator and provide a reason with a link to where the mistake/attack could be verified.

When a validator is removed for violating the rules, it is added to the blacklist by other validators and will be rejected from rejoining the network. Unlike any other voting, to remove a validator for violating the rules, 1/10 votes are enough. This is in order to protect the network from bad actors.

 ### Validator object
 - `validatorId` - the validator who cast the vote
 - `validatorData` - Needed for updates and addition of new validators. In case of an update, the `validatorId` in `validatorData`  and the root must be much. 
 - `removeReason` - in case of removal, the reason for the removal
 - `remove data` - in case of removal for rules violation, a CID to where the violation could be seen.

### Reword
Once a day, based on the snap time, during the first block, whose parent is the new day snapshot, a rewarding topic is created. It consists of new coins reward and transactions fees for the day and is divided among three parties:

 - Governments reward 4%
 - Validators reward 48%
 - Holders reward 48%

#### System capacity
There are 1,798,974,000,000 hard-cap coins. It is based on the dollar circulation value in Jan 2020, right before the epidemic started.

The daily reward is calculated using the below function, where X is a day starting from 1.<br>
$f(X)=Max(0, 98563309 - 2700X)$
It will be fully mined in 36,505 days(~100 years).

#### Validators reward
It is calculated based on the validators' performance in the past snapshots.

Validators are scored by:
 - The number of update transactions created.
 - The number of opting requests created.
 - Networking time as it has been reported by other validators(Median value)
 - Number of reported timeouts(Those have a negative score impact).
 - Number of blocks created - this is an alternative way to account for offline time

Only one value is kept in the Snapshot for scores for each validator metric. Each new result contribute a weight of $\frac{1}{10}$, such that the new value is $\frac{9currentValue + newValue}{10}$ 

A score of 0.5 to 1.5 is distributed among the validators based on their metrics above. 

The balance in the validator's wallet is in their stack. The stack is then multiplied by the score and used as a baseline for the validator's reward.

#### Holders reward
Once a user sets a Wallet on hold, it provides the `holdingMonths` 1-60, and two calculations apply:
 - The baseline is calculated as the `walletBalance * (1.012 ^ holdingMonths)`
 - the baseline is reduced for each reward by `baseline/totalDays`

Holders can send more coins to their wallets on hold. When that happens, the baseline calculation uses the original months provided in the hold request and increases the baseline. This is done to not encourage opening multiple wallets. $\frac{x}{totalDays} + \frac{y}{totalDays} = \frac{x+y}{totalDays}$. The reward will be the same. 

The one benefit of using the same wallet, however, is that it doesn't extend the hold time. The original hold time remains.

This method is especially helpful when encoding the holders into the Snapshot. The only required data for each holder is:
 - `startingHoldDate` - when the hold started
 - `monthsDuration` - for how long the hold will last.

The baseline for day $X$ is equal to:
$f(X)=walletBalance(1.012^{holdingMonths}) - \frac{X}{totalDays}$

The day's count starts from 0, such that the reduction is always the next day.

#### Governments reward
Each government reward from a block is calculated by the percentage of coins in wallets that chose that government. In case the government didn't adopt Confidence Coin just yet, the reward will go to the Confidence Coin foundation company. 

#### First block example reward

 - Assuming there are 100 validators with a total 10M stack(after scores multiplication), where the mid validator who scored 1 has a stack of 100K.
 - 1000 holders with 10M hold(after bonuses) where one of them has a 200K hold(after bonuses)
 - and one government that was chosen by half of the wallets

The first block reward is 98,560,609
 - The validators get 47,309,092.32 with the example validator who gets 473,090.9232
 - The holders also get 47,309,092.32 with the example holder who gets 946,181.8464
 - The government gets 1,971,212.18

## Transaction fees
Transaction fees are static in the system and are part of system settings. 

Ethereum makes, on average, 1M transactions a day and charges for it on average 500 ETC, which are $823K as of today(2/21/2023). So assuming we wish to charge 2M wallets(Sender and receiver, ignoring multi-send for simplicity) 823K coins. It's 0.41 coins for each address in the update transaction.

For opting and holding transactions, we can charge a similar amount of 1 coin.

Transaction fees are charged by the transaction maker. DTP pays for the updated transaction.

Transaction fees are accumulated in the Snapshot and released during the daily reward.


# Bridge
The Bridge is a microservice living in the same network as the CORE. They communicate over HTTP.

Bridge performs the next functionality:

## Publish blocks
It receives a file from CORE over a shared file system and publishes it to [IPFS](https://docs.ipfs.tech/concepts/what-is-ipfs/#what-is-ipfs) and announces the CID over  [libp2p](https://libp2p.io/) [gossip pubsub](https://github.com/libp2p/specs/tree/master/pubsub)

## Publish Snaps
It receives a file from CORE over a shared file system and publishes it to [IPNS](https://docs.ipfs.tech/concepts/ipns/#mutability-in-ipfs) where the sequence(version number) is the block id. The IPNS name is the validator public key.

This makes it very easy for Wallets to download Snapshots since they are hosted by all the validators, and all the validators provide a tracker to monitor new releases. 

## Collect blocks
Bridge is subscribed for the pubsub and downloads each new block from CID. It is then saved to the shared file system and notify the CORE. It doesn't have an understanding of what a block cycle is. That is managed by the CORE.

## Download Snapshot
In a rare scenario when the validator produces a bad Snapshot and needs to download a new one. It uses this endpoint. The Bridge downloads the Snapshot by CID.



# Wallet

The wallet is a Kotlin SDK. It will be developed in Kotlin and use IPFS Kotlin API. You can use it both on a desktop as well as on Android.

It will provide the functionality for 
 - Opt-in and out
 - Convert to DTP
 - Make update transactions.
 - Retrieve wallet balance
 - Subscribe for snapshot updates

There will be no functionality for downloading Snapshots and Blocks since that can be achieved via the IPFS SDKs.
