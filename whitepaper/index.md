# Confidence Coin whitepaper
*v1.0 last updated 1/18/2023*

Before reading, please review the [Flash consensus](/Flash-Consensus-algorithm/), and read about [DTP](/).

At a very high level, the prime goal of the system is to allow DTPs to perform update transactions with millions of users. And will enable each Wallet to opt-in and out from DTP groups. We will have a few vast and small blocks in the system. 

## System nodes
 - The system has three nodes: Wallets, DTPs, and Validators.
 - Wallets will opt in and opt out from DTPs
 - DTPs will make aggregated transactions with potentially millions of wallets involved.
 - Validators will validate and create blocks.

## Consensus
Blocks are enumerated with block IDs. Each Validator creates a block with the same ID value. It then shares it with other validators.
Once a Validator receives a block with ID N from all the validators, it creates a block with ID N+1 and shares it.
The time between two block ids is called a block cycle.

During the bootstrap process, all the Validators agree on a list of Validators and the current balance of all the wallets in Confidence Coin.
Only the Validators from the Validators list are considered for voting.

### Embedded voting system
The voting system is embedded in blocks. It saves time on communication and increases system security.

Each block contains zero or many topics with one or many items. When Validators create block id N+1, they vote for all the topics of blocks with id N. 
A topic is approved if it receives 2/3 votes of all the Validators.

Topics include transactions, opting requests, scoring, and more. 

## Network
Confidence Coin uses three networks
 - [IPFS](https://docs.ipfs.tech/concepts/what-is-ipfs/#what-is-ipfs) & [IPNS](https://docs.ipfs.tech/concepts/ipns/#mutability-in-ipfs)
 - [libp2p](https://libp2p.io/) [gossip pubsub](https://github.com/libp2p/specs/tree/master/pubsub).
 - HTTP

Validators publish new blocks over IPFS and announce the [Content Identifier (CID)](https://docs.ipfs.tech/concepts/content-addressing/) over pubsub. The CID must be signed with [Curve25519](https://en.wikipedia.org/wiki/Curve25519). Only hashes signed by validators from the Validators list are considered valid. Other messages are dropped. This is to prevent DDOS attacks.

Validator Id is Curve25519 public key. 
 - It is also the Validator Wallet address for collecting block rewards. 
 - The IPNS record name is generated from it
 - It is used for signing a block, and the CID is sent over pubsub

After each block cycle, validators generate a snapshot of the entire blockchain and publish it over their IPNS. The sequence(version number) must be the block number. The IPNS must link to a directory that has a new file for each snapshot. Also, there is no requirement to have the entire blockchain there.

Note that the snapshots directory and the files inside are not duplicated times the number of the Validators. There is a consensus about the content of those snapshots. All the Validators produce the same files that result in the same CIDs.

It serves multiple purposes.
 1. A snapshot is the transaction acknowledgment that it is committed.
 1. It reduces the amount of storage Validators need to operate. They don't need the entire blockchain. The latest ten snapshots are enough.
 1. It allows efficient encoding. After building a snapshot, each Wallet receives a short id. This id can be used in DTP update transactions to reduce the block size. 

## Blockchain snapshot structure
It contains block time T, a new line, and then the below items, separated with a new line. Each item is a headless TSV list. The list terminates with a new line.
1. Balance List - A balance of each user in Confidence Coin ordered by wallet id. 
   1. wallet id 
   1. balance
1. Validators List - A validators list ordered by Validator id
   1. Validator id(As described above)
   1. Address - IPv4/v6 or a domain address. It is used for sending transactions.
1. DTPs list
   1. DTP-id - the wallet id of the DTP
   1. update time - the update time of this DTP in seconds
   1. time to next update - time to next update as of the block creation time in seconds
   1. Wallets - comma-separated list of Wallets inside that DTP group, including the DTP Wallet itself, so this list can never be empty. Wallet ids are based on their order in the Balance List above, starting from 0
   1. Pending opt-out wallets - comma-separated list of Wallets inside that DTP group that will opt out after the next update transaction
   1. Pending opt-in wallets - comma-separated list of Wallets outside that DTP group that will opt in after the next update transaction in their DTP

## Topics
Wallets and DTPs create topic items. They use the HTTP protocol to contact Validators. The list of the HTTP remotes is obtained from the Validators list in the snapshot that is hosted on IPFS by all the Validators and some other Wallets and DTPs. Those are the most popular files in Confidence Coin.

HTML requests use JSON API. The response is a success or error code.
In this documentation, I placed the HTML API in a table. The topic structure inlines all the columns and delimiters them with a tab.

For example, the below documentation:
|param|description|
|-|-|
|param 1| param1 description|
|param 2| param2 description|
|param 3| param3 description|

Translates into the below JSON request
```JSON
{
	"param1" : "data",
	"param2" : "data",
	"param3" : "data"
}
```

And the below line in topics
```
param1	param2	param3
```

### Opting requests
Wallets can opt in and off to and from any DTP. 

|param|description|
|-|-|
|wallet address|Curve25519 public key|
|DTP id|DTP wallet address|
|signature| Curve25519 signature|

### Color Scoring
There is a T seconds timeout waiting to get blocks from other validators.
 - If a block was announced and downloaded(from IPFS) within 0.5T, it gets a GREEN score, and the times will be recorded.
 - If it took less than T seconds to get an announcement and less than T seconds to get it from IPFS, then it scores YELLOW, and the times will be recorded.
 - Otherwise, it gets RED, and the attempt to get the block should halt. -1 will be recorded in that case for time.
The top 1% of the total validators who scored RED by over 2/3 VALIDATORS are removed from the Validator list. They are ordered by the hash of the block id and the validator id. This is to create a random order.
After removing Validators, T is doubled.

In case no RED was removed, and there is less than 10% YELLOW, the time is reduced to have 10% YELLOW and up to 1% RED(not inclusive) by 2/3 votes, based on the last block cycle. For this calculation purpose, a RED with a low score will be calculated as T.

The goal of this algorithm is to remove slow Validators and always operate at maximum network capacity. Aka not running into timeouts.
This makes the block time inconsistent. It will improve as technology improves.

|param|description|
|-|-|
|pubsub score| the time in integer seconds, round down it took to announce the block|
|IPFS score| the time  in integer seconds, round down it took to download the block from IPNS|

Note that topics are already sectioned by Validators, so there is no need to include a validator id. Also, there is no HTTP API for scoring. This is a particular internal action.

### Register DTP
A wallet can convert itself to DTP if it opt-out from other DTPs. 
It must provide an update time in seconds for how often it will sync Wallet balances to the blockchain via the update transaction.

|param|description|
|-|-|
|Wallet address| DTP wallet address|
|update time| the time in seconds for the periodic update|
|signature| Curve25519 signature|

### DTP update transaction
DTP must perform an update transaction within the update time from its conversion to DTP or the last update transaction. If it fails, it loos all the Wallets in the DTP group and converts back to regular Wallet.
DTP must not add or remove coins. The number of coins in the system before and after the update transaction should not change.

During this transaction, DTP can also create new Wallets. Those Wallets will automatically become part of the DTP group.

Only the Wallets ids and the new balance are required when making the transaction. The ids come from the Balance list in the Blockchain snapshot. The snapshot must be fresh from the last ten snapshots. DTP should try to use the latest.

|param|description|
|-|-|
|Block id| Block id for wallets ids mapping|
|new Wallet addresses| Comma-delimited list of new wallet address
|Wallets ids| Comma-delimited list of Wallets ids from the Balance List|
|balance ids| Comma-delimited list of balances corresponding to the Wallets ids and the new Wallets|
|signature| Curve25519 signature of the DTP|

### DTP external transaction
DTP can send coins to other DTPs on behalf of the Wallets. No Wallets signature is needed, and multiple transactions can be aggregated.

|param|description|
|-|-|
|Block id| Block id for wallets ids mapping|
|origin Wallets ids| Comma-delimited list of Wallets ids from the Balance List|
|balance ids| Comma-delimited list of balances to transfer corresponding to the Wallets ids|
|destination Wallets addresses| Comma-delimited list of Wallets addresses, those are no using ids since those could potentially be new Wallets|
|signature| Curve25519 signature of the DTP|

### Opting
When opting for a DTP, the Wallet is automatically removed from other DTPs.
To synchronize the off-chain transactions inside the DTP with opting requests. When the Wallet belongs to a DTP group, its opting requests be delayed until the next DTP update transaction or until it is automatically removed from the DTP because the DTP didn't update on time.

|param|description|
|-|-|
|DTP id| The wallet address of the DTP|
|opting| in/out to opt-in or out|
|signature| Curve25519 signature|

## Block structure
Blocks use JSON API and have the below structure:

 - Block id - A running integer
 - Block creation time - used for DTP update transaction calculations
 - Validators list CID
 - Validator id - the validator id in the Validators list
 - Validator IP - an IPv4/v6 address or a domain name where topics can be submitted
 - Topics[] - object array
    -  Topic name - String
    -  items[] - Objects array based on the name value
 - votes[] - objects array
 - CID - String
 - Validator signature of the fields above

Note that there is no block hash in here. This is because the system is using CID.
