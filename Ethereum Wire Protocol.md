Peer-to-peer communications between nodes running Ethereum clients run using the underlying [ÐΞVp2p Wire Protocol](https://github.com/ethereum/wiki/wiki/%C3%90%CE%9EVp2p-Wire-Protocol).

### Basic Chain Syncing
- Two peers connect & say Hello and send their Status message. Status includes the TD & hash of their best block.
- The client with the worst TD asks peer for full chain of just block hashes.
- Chain of hashes is stored in space shared by all peer connections, and used as a "work pool".
- While there are hashes in the chain of hashes that we don't have in our chain:
  - Ask for N blocks from our peer using the hashes. Mark them as on their way so we don't get them from another peer.

### Ethereum Sub-protocol

**Status**
[+`0x00`: `P`, [`protocolVersion`: `P`, `networkId`: `P`, `td`: `P`, `bestHash`: `B_32`, `genesisHash`: `B_32`] Inform a peer of it's current **ethereum** state. This message should be send _after_ the initial handshake and _prior_ to any **ethereum** related messages.
* `protocolVersion` is one of:
    * `0x00` for PoC-1;
    * `0x01` for PoC-2;
    * `0x07` for PoC-3;
    * `0x09` for PoC-4.
    * `0x17` for PoC-5.
    * `0x1c` for PoC-6.
* `networkId` should be 0.
* `td`: Total Difficulty of the best chain. Integer, as found in block header.
* `bestHash`: The hash of the best (i.e. highest TD) known block.
* `genesisHash`: The hash of the Genesis block

**GetTransactions**
[+`0x01`: `P`] Request the peer to send all transactions currently in the queue. See Transactions.

**Transactions**
[+`0x02`: `P`, [`nonce`: `P`, `receivingAddress`: `B_20`, `value`: `P`, ... ], ... ] Specify (a) transaction(s) that the peer should make sure is included on its transaction queue. The items in the list (following the first item `0x12`) are transactions in the format described in the main Ethereum specification.

**GetBlockHashes**
[+`0x03`: `P`, `hash` : `B_32`, `maxBlocks`: `P` ] Requests a `BlockHashes` message of at most `maxBlocks` entries, of block hashes from the blockchain, starting at the parent of block `hash`. Does not _require_ the peer to give `maxBlocks` hashes - they could give somewhat fewer.

**BlockHashes**
[+`0x04`: `P`, `hash_0`: `B_32`, `hash_1`: `B_32`, `....` ] Gives a series of hashes of blocks (each the child of the next). This implies that the blocks are ordered from youngest to oldest.

**GetBlocks**
[+`0x05`: `P`, `hash_0`: `B_32`, `hash_1`: `B_32`, `....` ] Requests a `Blocks` message detailing a number of blocks to be sent, each referred to by a hash. Note: Don't expect that the peer necessarily give you all these blocks in a single message - you might have to re-request them.

**Blocks**
[`+0x06`, [`blockHeader`, `transactionList`, `uncleList`], ... ] Specify (a) block(s) as an answer to `GetBlocks`. The items in the list (following the message ID) are blocks in the format described in the main Ethereum specification. This may validly contain no blocks if no blocks were able to be returned for the `GetBlocks` query.

**NewBlock**
[`+0x07`, [`blockHeader`, `transactionList`, `uncleList`], `totalDifficulty` ] Specify a single block that the peer should know about. The composite item in the list (following the message ID) is a block in the format described in the main Ethereum specification.
- `totalDifficulty` is the total difficulty of the block (aka score).

### Session Management

For the Ethereum sub-protocol, upon an active session, a `Status` message must be sent. Following the reception of the peer's `Status` message, the ethereum session is active and any other messages may be sent.

### Upcoming changes
- [Light Client Protocol](https://github.com/ethereum/wiki/wiki/Light-client-protocol)

### Changes (PoC-7)
- [NewBlock Message](https://github.com/ethereum/wiki/wiki/NewBlock-Message)

### Changed (PoC-6)
- [Parallel Block Downloads](https://github.com/ethereum/wiki/wiki/Parallel-Block-Downloads)