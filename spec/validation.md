## Block Validation

In order to ensure they are always on the correct latest state of the chain a filecoin full node must accept and process blocks continuously. Blocks are propagated was described in the [Data Propagation](data-propagation.md) document.

For every block received, the node must validate it before executing it or passing it on. Before a node can validate a block, it first must ensure the block is structurally correct by decoding it (see [block](data-structures.md#block)) and ensuring that no field contains any illegal values. After that the node must check the signature of the block, then moves on to validate the block itself.

A valid block:

- must only have valid parents in the tipset, meaning
  - that each parent itself must be a valid block
  - all parents must be at same height
- must have a valid ticket:
  - the ticket must be the winning ticket
  - the ticket must be generated from the smallest ticket in the parent tipset
  - all tickets in the ticket array must have been generated by the same miner
  - if it includes intermediary failed tickets (i.e. null blocks) in the ticket array, the node must confirm that each ticket correctly generates the next in the array
- must only have valid state transitions:
  - all messages in the block must be valid
  - the execution of each message, in the order they are in the block, must produce a receipt matching the corresponding one in the receipt set of the block, see [the state machine spec](state-machine.md).
- the resulting state root after all messages are applied, must match the one in the block

Note: Once the block passes validation, it should be added to the local datastore, even in the case where we don't accept it right now. Future blocks from other miners may be mined on top of it and in that case we will want to have it around to avoid refetching. Blocks a certain distance from the current chain height may be dropped (exact number TBD, but blocks that havent been included after several days may be purged).

To make certain validation checks simpler, blocks should be indexed by height and by parent set. That way sets of blocks with a given height and common parents may be quickly queried. It may also be useful to compute and cache the resultant aggregate state of blocks in these sets, this saves extra state computation when checking which state root to start a block at when it has multiple parents.
