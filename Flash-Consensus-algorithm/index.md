# The **Fast** Flash Consensus Algorithm
*v1.2 last update: 2/8/2023*

The Fast Flash Consensus Algorithm is a speedy consensus mechanism for blockchain transactions. Its innovative design allows every node to generate and validate blocks at the same time, leading to a high transaction per second(TPS) rate, thereby ensuring quick and secure transactions.
# Validators and Wallets in Flash

There are two types of nodes in Flash, Validators and Wallets. 
Validators create and validate blocks, driving the network's high TPS, while Wallets create new transactions.

# # Unlimited Block Size in Flash 
In Flash the block size is unlimited, allowing for a high volume of transactions to be processed in a single block. This is achieved through the combination of two key features: 
 - Direct Communication between Wallets and Validators 
 - and Snapshots.

Snapshots are created by Validators after each block cycle. Snapshots contain a full system record, meaning that information about a Wallet balance can be obtained without downloading the entire blockchain, as only the most recent Snapshot is required.

In traditional consensus algorithms, transactions are typically propagated through all nodes twice - once when they are shared with the network and again when they are included in a block. However, in Flash, Wallets communicate directly with Validators, eliminating the need for multiple propagations. Transactions are received by the Validator only once, either as part of a new block or directly from a Wallet, ensuring the efficient and streamlined processing of transactions.

# Combined Voting and Transactions in Flash
In Flash, the voting process and transaction validation are combined in a single block, making blocks the sole communication channel between Validators. This unique approach allows for each Validator to produce only one new block per cycle, streamlining the consensus process and resulting in the creation of a new Snapshot. These Snapshots, which provide a full record of the system, are an integral part of the Flash network and help to maintain its fast and efficient operation.

# Three-Step Consensus 
Reaching consensus is achieved through a three-step process that Validators run in a loop.
Collect Blocks - Validators wait for each Validator in the system to produce a block with the same ID (block N).
Create a Snapshot - Validators examine the voting section of blocks N for transactions included in blocks N-1 and create a corresponding Snapshot (Snap N-1).
Vote and Create a Block - Validators use Snap N-1 to cast votes on transactions in blocks N and create the next block (block N+1).
This simple yet intricate process allows for efficient consensus and the creation of new Snapshots with each block cycle, ensuring a complete system record is always maintained.

# Bootstrapping and Recovery 
In the event of a system outage or for a new node joining the network for the first time, the Validator must obtain the latest Snapshot and blocks from the last two cycles to start the three steps loop. At least 2/3 of the Validators must be online in order to reach a consensus and create a new Snapshot

# Validators List and Performance
The blockchain maintains a list of validators. The performance of each validator is measured and evaluated based on their ability to produce blocks efficiently. The better the performance, the higher the reward.

The network reaches a consensus on the number of validators to include in each block cycle through voting on a specific topic related to adding or removing validators from the list.

Each validator assesses the performance of the other validators by measuring the time it takes to receive a block from them. There are also timeouts in place, beyond which a validator will no longer wait for a block.

The scores that validators receive have an impact on the rewards they receive. In cases where a validator experiences frequent timeouts, they may be removed from the validator list.

## Addition of New Validators
New Validators are invited by existing Validators. This process is a manual one and involves communication between human beings. The information about the new Validators is added to a dedicated topic in the block, which is shared with other Validators in the network. 

# Block Structure
A block in the system consists of several components:

 - Block ID: A unique, incremental number assigned to each block
 - Snap Hash: A hash of the most recent Snapshot created by the Validators
 - Block Hash: Can also serve as an InfoHash in BitTorrent
 - Validator ID: A unique identifier for the Validator, which can be a wallet address for rewards or another identifier. All blocks produced by a single Validator should have the same Validator ID.
 - Block Hashes: A reference to previous blocks on which the current block is casting votes. The block hashes should refer to the block with an ID one less than the current block's ID.
 - Topics: Optional data topics that can include various information.

# Reward System
The reward system in Flash is based on a combination of stack and performance. The stack serves as a basic evaluation value, while penalties are applied for the following factors: negatively scoring other validators, slow performance, low number of transactions, and low number of blocks produced.

Additionally, the stack is not calculated linearly, but as `S^stack` where `S` is a number close to 1 (e.g. 1.0001). This encourages investing in node quality over quantity for better performance.

# Holders Incentive
The Flash blockchain empowers holders with a unique reward system. The block reward is split 50/50 between the validators and the holders. To become a holder, coins must be put on hold for a period of 1-120 months. The number of coins held is used as the baseline for calculating rewards.

Holders are incentivized to keep their coins on hold for longer periods by multiplying the rewards by 1.016 for each additional month of holding. The system also encourages holders to keep adding more coins to their holding by implementing a linear penalty that reduces the rewards each day, resulting in zero rewards for holding coins on the last day.

# Attacks
## Double-spending Attack
Flash is highly resistant to the double-spend attack as there are no forks in the system, and all blocks become part of the blockchain.
Let's examine some potential cases and see how Flash responds:
 - **Scenario 1**: A wallet tries to double-spend a coin in block 19 after previously spending it in block 10. In this case, a validator with knowledge of all blocks up to block 19 in the blockchain would detect the attack and refuse to add the transaction.
 - **Scenario 2**: A wallet tries to double-spend a coin by simultaneously sending two transactions to the same validator. The validator would decline the second transaction, which would not be recorded in the blockchain.
 - **Scenario 3**:Case 3: A wallet attempts to double-spend a coin by sending two transactions simultaneously to different Validators. Each Validator will approve the transaction and add it to the block it creates. The other Validators will then compare the transactions to the full blockchain picture based on the last snapshot they have and wait for all the blocks from all Validators before making a vote. Only one of the transactions will receive over 2/3 of the Validator's votes, as there are not enough votes to approve both of them
 - **Scenario 4**: A wallet tries to double-spend a coin by simultaneously sending two transactions to different validators, and 90% of the validators are compromised and willing to cooperate. In this case, the blockchain would record two conflicting blocks and also record the validators who allowed it. Genuine validators would then remove the compromised validators from the list of validators.

## Network Attack
Attacks on the network by compromised Validators are not motivated as there is no way for them to double-spend coins. Only an attacker with the goal of sabotaging the system would perform such an attack.

To counter this, Flash encourages Validators to maintain high network performance, and it accounts for all nodes. If any nodes become slow or inaccessible, they are removed from the network.

In the event of a power outage or network attack, Flash waits for a consensus to be established among the Validators. The nodes cannot work on the next block until there is a consensus on the latest snapshot. In such scenarios, the block waiting timeout may be extended indefinitely.

To recover from a network attack, at least 2/3 of the active nodes must be available. In such cases, it is expected that people would pick up the phone and help to troubleshoot the problem manually.
