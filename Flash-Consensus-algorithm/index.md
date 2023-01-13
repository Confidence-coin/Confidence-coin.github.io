# Flash Consensus algorithm
*v1.1 last update: 1/12/2023*

Flash offers the **fastest possible** transaction per second time(TPS). Nothing blocks a transaction from being added to the blockchain. When Wallets send transactions, they are instantly added.
 - No waiting for blocks to be shared in the network
 - No waiting for voting
 - No waiting for leader nodes to sync between themself
 - No waiting, period!

The network performance is the only TPS limitation. It's not just fast but mathematically the fastest(TPS) possible consensus solution. 

## Flash nodes
There are two types of nodes in Flash, Validators, and Wallets.

Validators create and validate blocks, while Wallets create new transactions. Both Validators and Wallets share blocks. 

There is no transactions pool in Flash. 
Wallets send transactions to Validators
Validators create new blocks with those transactions and share them with the network.
Done - consensus is reached. Let's see how this magic works.

# The Consensus magic
To reach a consensus, Validators validate all the blocks of each other. They vote on various topics from other blocks in each new block they create. It is structured as follows:
Block Hash
Transactions Topic
Unapproved transaction index as it appears in the original block referenced above. If it's not mentioned, it is considered approved.
New Validators Topic
...
Once 2/3 of the validators approve an item, it is considered final. Even if all the items in a block are rejected, or it has non, it is still added to the blockchain since it contains voting information on other blocks.

Blocks are enumerated. A Validator must vote for the Ns blocks when producing the N+1 block. While waiting for other validators to share their blocks, it collects transactions and other topics and will flush it all in the next block. There is no algorithmic limit on the block size.

There are a few extra conditions here to address connectivity. But at this point, you should be able to see the power of Flash.
Blocks are validated simultaneously. 
Blocks are added simultaneously to the blockchain.
No time is wasted on sharing transactions. Transactions are instantly added to a block that becomes part of the blockchain once it is shared.
## Connectivity
With Flash, the network is taken to the limit. The system operates at maximum network capacity.
There are no timers.
No queues
No acks
No limits! 

The moment a Validator gets all its blocks, it generates a new one, and the cycle repeats. The system is massively parallelized.

Now let's talk about reality. Rules need to be placed for occasions when Validators go offline or leggy. To that end, knowing how many blocks a Validator should expect to get in each cycle is essential.

## Hello Validators List
The network is managing a validators list. In that list, each Validator is scored for performance. The faster the system can produce blocks, the better.
There is a particular topic to vote for the addition and removal of validators from that list. It creates a consensus around how many validators are in each block cycle.

There are three scores for performance: Green (Default), Yellow, and Red.
They are selected based on the time it takes to receive a block from a Verifier, for example: 
Zero to one minute -> Green
One to two minutes -> Yellow
Over two minutes -> Red
Yellow and green are the best. It allows for maximizing the network performance to the limit. 
A Red score is when a block was not received within the time limit. In such a case, a Validator will submit a block and not evaluate the red Validators.
A yellow score will indicate how long it took to receive a block.

During each iteration, The top 1% validators(From the total validators number, rounded up) who got a yellow score by more than 2/3 of validators will be removed. They are ranked based on the total cumulative waiting time in nanoseconds. And in the unlikely case of even, the decision could be made based on the Validator's identity or block hashes.

If some validators gave a Red score, and others gave a Yellow score for the same Validator, the Red will be treated as Yellow.

In case over 2/3 of the Validator scored a Validator as Red, it is removed.

After removing Red validators, the next block score times increase.

If all the validators got a green score, the score time would be reduced. The particular logic for time shrinking depends on the implementation.

The idea is to decide the time dynamically. Such that if the Validators upgrade their gear, the time settings adapt accordingly.

## Join the game - How are new validators added?

Each Validator can nominate new Validators to join. It is put to the vote, and a decision is made.
There also should be a penalty for rejoining the Validators list.

## Block Structure
 - block id - a running number
 - Validator id - It could be a wallet address for rewards or another identifier. All the blocks produced by a validator should have the same validator id.
 - block hashes - this is the voting system described in "The Consensus magic."
   - topics - optional data topics. It could include:
    - transactions
    - new validators
    - some requests(like banning a validator or changing settings)
 - hash of this block - It can also serve as InfoHash in BitTorrent

## Reword system
Flash is built for performance and rewards performance. Every 100 blocks, the top 50% of the validators receive new coins from the system. They are scored by Red/Yellow/Green color votes and the number of block hashes they referred to in their blocks. This is to disincentivize voting Red.

# Attacks

## Double-spending attack
Unlike PoW and many other consensuses, Flash is highly resistant to the double-spend attack since there are no forks in the system. All the blocks are valid and become part of the blockchain. 

But still, let's review the possible cases on Flash.

**Case 1**: A wallet attempts to double-spend a coin in block 19(Each validator produce block 19) after spending it in block 10.

A validator has knowledge about all the blocks up to 19 in the blockchain. It will detect the attack and not even add the transaction.

**Case 2**: A wallet attempts to double-spend a coin by sending two transactions simultaneously to the same Validator.

A Validator will decline the second transaction. It will have no record in the blockchain.

**Case 3**: A wallet attempts to double-spend a coin by sending two transactions simultaneously to different Validators.

Each Validator will approve the transaction and add it to the block it creates. When other Validators receive those blocks, they will randomly approve one transaction and reject the other since they are looking at the full blockchain picture and waiting for all the blocks from all the validators before making a vote. 
One of those transactions may get over 2/3 Validator's votes, but there are not enough votes to approve both of them.

**Case 4**: A wallet attempts to double-spend a coin by sending two transactions simultaneously to different Validators, and 90% of the validators are compromised and willing to help.

Validator votes are part of the blockchain. Not only that the blockchain will record two conflicting blocks, but it will also record who were the validators who allowed it. 
When the genuine validators see that, they will automatically remove the compromised validators from the validators list.

** Network attack
There is no motivation for compromised Validators to perform attacks since there is no way to double-spend coins. 
So the motivation to run an attack is pure evil — someone whose goal is to sabotage the system.

Flash uses the Red/Yellow/Green approach to maintain high networking performance. 
Flash is consistently accounting for all the nodes. If some nodes become slow or inaccessible, they will be removed from the network.

However, there is one case that needs to be covered. What will happen if more than 1/3 of the network becomes inaccessible?

Flash doesn't provide a solution to that. Be it an attack or a large power outage. It doesn't cover by Flash. People will have to pick up the phone and manually troubleshoot it.

