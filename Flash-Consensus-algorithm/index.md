# Flash Consensus algorithm
Flash offers the fastest possible transaction per second time(TPS). It means that it has zero synchronization. When new transactions are submitted, they are instantly added to the blockchain.
 - No waiting for blocks to be shared in the network
 - No waiting for voting
 - No leader nodes synching between themself
 - No waiting, period!

The TPS is only limited by the network performance, it's not just fast, but it's mathematically the fastest(TPS) possible consensus solution. 
## Flash nodes
There is only one type of node in Flash - Validators
Users use wallet apps to send transactions to validators. Validators then verify those transactions and add them to the blockchain.

There is no transactions pool in Flash. When wallets send transactions to validators, validators create new blocks and share them with the network.
Validators use only one command to reach a consensus - Share a block.
I will keep the network implementation abstract here. The network must allow a validator to share a block and have all the peers on the network receive it.
Block references/links
Flash uses a directed acyclic graph (DAG) to store blocks. Each block references the hashes of the last N blocks known to the validator, where N is the number of nodes in the network. To illustrate the size of it, let's use Bitcoin as an example. Based on bitnodes.io, Tue Jan 3 09:46:24 2023 EST, Bitcoin has 15,567 nodes. It has a hash size of 32 bytes, so referencing all the nodes adds up to 32 * 15,567 = 486KB.

Note that the references are part of the block and hashed together with the rest of it. It is infeasible to break the DAG with cycling reference since that implies solving a hash collision problem.
## Consensus 
Flash reaches consensus by looking into the referencing blocks. When a block is referenced by more than ⅔ of the validators, it is considered committed and immutable.

There are several additional rules that I will go over, but at this point, it must be clear to you how Flash is so fast. All it takes to push a transaction into the blockchain is to share it with the network. Each validator is doing that in parallel, and nothing is synchronized.
### Zero transaction blocks
Blocks serve two purposes, they contain transactions, and they reference other blocks. It works like a voting system. 
There could be a situation where transactions are not sent evenly to all the nodes. Some produce many blocks, and some are none. It's a problematic situation since it means that validation will not work. To avoid this situation, a stabilization mechanism is needed, pressure management on the validator level, and a minimum block time. Validators must create blocks every few seconds, or their vote will be lost. This is why blocks without transactions are valid in the system.
You may argue that blocks with zero transactions are a deviation from the mathematical TPS perfection, but it will only be the case for yang cryptos. Once the system is more established, it will become unlikely to see blocks with no transactions. Still, it is an important point to note.
## Invalid transactions
When validators receive a block evaluated as valid by another validator, still the validator decides that some of the transactions in that block are invalid, the validator will not reject the block; instead, it will log the invalid transactions in the next block that it will create.

## Block structure
 - Transactions
 - Reference to other blocks
 - Reference to invalid transactions - this could be the index of the transaction in a block.
 - Signature(and optionally the identity) of the validator who created the block
A block hash - A hash of the above items

Note that there are no timestamps and no enumerators in the above structure; both are security risks that do not exist in Flash.
## Validate the validator
When validators validate a block, they validate not only the transactions but the decision made by the creator of that block(another validator). It's convenient since the block contains all the inputs and outputs of a validator decision. 
But let's not jump into this. It's outside the scope of this article. Let's talk more about transaction validation.
## Double spent attack
To double spent a coin, one needs to make a transaction(T1), then have ⅔ of the nodes mark it as valid(keep it out of the invalid transactions list), and then do it again(T2) while double spending the coin. Let's consider all the options here:

1. A validator gets T1 after T1 is fully committed. It gets T2 - the validator approves T1 since there are coins in the sending wallet, but it will reject T2 since there will be an insufficient balance for the requesting wallet. In such a case, the transaction will not even be included in the block.
2. A validator gets T1, and before it is fully committed, it gets T2 - The validator will approve T1 and will reject T2 since that validator is consistent with its actions, and if T1 is valid, then T2 cannot be. Some validators may also get T2 before T1. In such a case, the decision will be reversed. But due to the pigeon effect, the invalid transaction can get at most 50% of the approvals. Beyond that, the validators must be aware of the other transaction, and they will rule it out.
3. What will happen if ⅓ of the validators are compromised? In such a case, if you can make ⅓ of the honest validator approve T1 and have the other ⅓ approve T2, then the compromised validators can approve both T1 and T2. They will be able to reach ⅔ of approvals but not over ⅔ as required by Flash. In reality, it will be very difficult to manipulate the network in such a way. 

The best defense Flash has against compromised validators is the traces they leave behind. The honest validators will be able to detect bad decisions the compromised validators made and take action against them.
## Summary
I kept the description of Flash very abstract on purpose. Flash can be adapted to both PoS and PoW, with an arbitrary hash algorithm and choice of network.
Flash provides the fastest possible TPS and a very fast confirmation time.

Flash is a decentralized and secure consensus algorithm. I plan to use it in [Confidence Coin](https://confidence-coin.com).




