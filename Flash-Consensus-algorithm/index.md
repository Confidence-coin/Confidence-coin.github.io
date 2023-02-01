# Flash Consensus algorithm
*v1.2 last update: 2/1/2023*

Flash is a consensus algorithm that enables fast blockchain transaction speeds due to its ability to process a large number of transactions per second (TPS). The algorithm achieves this by having each node produce a block and all the blocks being validated simultaneously. This cycling process creates an efficient loop that allows for faster processing.

# Flash nodes
There are two types of nodes in Flash, Validators and Wallets.
Validators create and validate blocks, while Wallets create new transactions. Both Validators and Wallets help to propagate the blocks through the system.

# Snapshots
The blocks in Flash are unique due to their limitless size. This is enabled by the fact that a full system Snapshot is created after every block cycle, meaning information on a Wallet balance doesn't require downloading the entire blockchain but only the most recent Snapshot

# Voting system
The secret of Flash speed comes from consolidating transactions with validation logic. Validators vote on topics in other blocks and store their vote in the block they produce. Topics contain a variety of things, one of which are transactions. This way, there is only one communication channel between Validators: Share a block.

# Three steps Consensus
To reach a consensus, each Validator runs the below steps in a loop.

1. collectBlocks -> Wait for each Validator in the system to produce a block with id `N`(block `N`).
2. createASnap -> Look at blocks `N` voting section for transactions in blocks `N-1` and create a snapshot(Snap) `N-1`
3. voteAndCreateABlock -> Create block `N+1` with voting for transactions in blocks `N` using the Snap `N-1`

It is simple, complex, and beautiful altogether. 

## Step 2: createASnap 
To verify a transaction in block `N`, the validator needs to check the balance in Snap `N-1`. After all the transactions are verified, a new Snap can be created. This is done in step 2.

## Step 3: voteAndCreateABlock 
After a new Snap is created, it is time to create a new block. In step 2, we looked at `N` blocks voting data. But in this step, the Validator votes on the transactions in `N`.
You may ask yourself, if all the validators look on the same `N` blocks, how is it even possible to have different Votes? The answer is resilience. Sometimes nodes go offline, and blocks are lost in IO. Flash requires 2/3 of the nodes to agree on a transaction to make it valid.

## Step 1: collectBlocks 
Both the end and the beginning of the cycle. Validators are exchanging the next block and waiting to collect all the pieces. 

# Validators List
The blockchain is managing a validators list. Each Validator is scored for performance. The faster the system can produce blocks, the better. There is a particular topic to vote for regarding the addition and removal of validators from that list. It creates a consensus around how many validators are in each block cycle.

Each validator scores all the other Validators based on the time it took to get a block from them. There are also timeouts beyond which the Validator will not wait any longer for a block. 

Scores affect the reward validators are getting, which can be used to remove the Validator from the Validator list, especially in case of timeouts.
## How are new validators added?
New Validators are invited by existing Validators. The information is shared with other validators over the block.

# Block Structure
 - block id - a running number
 - Snap hash - a hash of the most recent Snap
 - block hash - It can also serve as InfoHash in BitTorrent
 - Validator id - It could be a wallet address for rewards or another identifier. All the blocks produced by a validator should have the same validator id.
 - block hashes - this is the voting system described in “The Consensus magic.”
 - topics - optional data topics. It could include:

# Reward system
The flash reward system is based on Stack. It is a combination of Stack and performance. The stack places a basic evaluation value, and there are penalties for the below items.
 - Negatively scoring other Validators - This is to discourage ignoring other validators' blocks to gain performance.
 - Slow performance - Based on the scoring system
 - low Number of transactions - This is to discourage producing blocks with no transactions
 - low Number of blocks - This is to discourage going offline

 Also, the Stack is not counted linearly. It is calculated as `1.0001^stack`. This is to discourage making multiple nodes. Investing in node quality over quantity is better for performance.
# Attacks
## Double-spending attack
Flash is highly resistant to the double-spend attack since there are no forks in the system. All the blocks are valid and become part of the blockchain.
But still, let’s review the possible cases on Flash.
 
 - **Case 1**: A wallet attempts to double-spend a coin in block 19(Each validator produces block 19) after spending it in block 10.<br>
A validator has knowledge about all the blocks up to 19 in the blockchain. It will detect the attack and not even add the transaction.<br>
 - **Case 2**: A wallet attempts to double-spend a coin by sending two transactions simultaneously to the same Validator.
A Validator will decline the second transaction. It will have no record in the blockchain.<br>
 - **Case 3**: A wallet attempts to double-spend a coin by sending two transactions simultaneously to different Validators.
Each Validator will approve the transaction and add it to the block it creates. When other Validators receive those blocks, they will randomly approve one transaction and reject the other since they are looking at the full blockchain picture and waiting for all the blocks from all the validators before making a vote. One of those transactions may get over 2/3 Validator’s votes, but there are not enough votes to approve both of them.<br>
 - **Case 4**: A wallet attempts to double-spend a coin by sending two transactions simultaneously to different Validators, and 90% of the validators are compromised and willing to help.
Validator votes are part of the blockchain. Not only that the blockchain will record two conflicting blocks, but it will also record who were the validators who allowed it. When the genuine validators see that, they will automatically remove the compromised validators from the validators list.

## Network attack
There is no motivation for compromised Validators to perform attacks since there is no way to double-spend coins. So the motivation to run an attack is pure evil — someone whose goal is to sabotage the system.
Flash uses the Red/Yellow/Green approach to maintain high networking performance. Flash is consistently accounting for all the nodes. If some nodes become slow or inaccessible, they will be removed from the network.
However, there is one case that needs to be covered. What will happen if more than 1/3 of the network becomes inaccessible?
Flash doesn’t provide a solution to that. Be it an attack or a large power outage. It doesn’t cover by Flash. People will have to pick up the phone and manually troubleshoot it.

