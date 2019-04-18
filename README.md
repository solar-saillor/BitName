# BitName: A Metanet Domain Name Protocol

> Protocol prefix: `1LX1b3YmwdE82kMPE291TCUsBErQ7sSfRJ`

BitName is an `OP_RETURN` protocol to obtain one-to-one mappings between Bitcoin txids and human-readable strings. 

Users register and exchange ownership of BitName by simply sending specific `OP_RETURN` on the blockchain. 

Planaria nodes that run the BitName protocol then crawl the blockchain and construct the BitName mapping in order to serve search queries.

## Background

### 1. OP_RETURN Contents

Hosting server-less websites by storing page contents in Bitcoin's `OP_RETURN` space has become a common practice in the Bitcoin (BSV) development world . 

Currently such pages are mostly accessed by looking up their respective 256 bits txids, as defined by the `B://` protocol. A txid looks like this:

```
729dcc63aa3cf388296177e86a564f8cd5d112be1a3cd43f5c71195fe5eab716
```

### 2. Problems

While txid can serve as the unique identifier of any `OP_RETURN` contents, they are difficult for humans to identify, to associate, and to remember. 

What is needed is a standard mapping from txids in machine format to the limited and valuable human cognitive space, much like the mapping between IP addresses and human-readable domain names on the current internet.

## Philosophy

Short, human-readable strings form a much smaller space than that of Bitcoin txids. To resolve conflicts and to establish a coherent mapping between these two spaces in a decentralized manner, BitName relies on the Bitcoin blockchain itself as the single source of truth that records all the state changes, and a set of clearly-defined rules that apply these changes. 

Any state machine that follows the BitName protocol can crawl the blockchain and reach the same mapping, as both the blockchain and the protocol is open.

## Protocol

### 1. Structure of BitName

Like DNS domain names, BitNames are hierarchical strings that contains one or more labels, separated by forward slashes. The labels are case-insensitive and are formed from the set of ACII letters, digits, and hyphens (a-z, A-Z, 0-9, -), but not starting or ending with a hyphen. 

An example of BitName would look like the following: 

```
apple/macbook-air/specs
```

Each BitName is owned by a Bitcoin address, which can be a different address than that of its parent BitName. E.g. `apple`, `apple/macbook-air` and `apple/macbook-air/specs` can belong to different addresses.

### 2. Registration

Registering a BitName under an address and establishing its mapping to a txid is as simple as sending an `OP_RETURN` transaction that contains a message in the following format:

```
OP_RETURN 1LX1b3YmwdE82kMPE291TCUsBErQ7sSfRJ [owner address] [mapped txid] [BitName] 	
```

The registration tx needs to satisfy the following conditions to be considered valid:

- No other transaction that registers the same BitName can be found in earlier blocks
- The mapped txid corresponds to an existing tx that has `OP_RETURN` output 
- If the BitName to be registered falls under a parent BitName, the registration tx has to have at least one input from the owner address of its parent BitName

If a registration transaction meets the above conditions and is the only one in its block for this BitName, then the registration will be in effect after the block is mined. 

If there are competing valid registration transactions for the same BitName in the same block, all registrants will engage in an all-pay auction for the BitName, with the miner fees paid by their registration tx as their bids. Before the block is mined, all registrants can repeatedly send their `OP_RETURN` messages to register and attach any miner fees to them. When the block is mined, whichever BitName/owner address/mapped txid combination has the highest total miner fees paid by its corresponding registration tx will win the auction. If there is a tie in the amount of satoshis paid, the auction will be extended to the next block and will keep going until there is a winner after a block is mined.

An auction example can be illustrated below:

![Auction example](https://i.imgur.com/lGea8EH.png)

### 3. Update

Once a BitName is registered, its owner can update its owner address and mapped txid. All that is needed is an update transaction in the blockchain that has an `OP_RETURN` output containing a message in the following format, similar to that of the registration message:

```
OP_RETURN 1LX1b3YmwdE82kMPE291TCUsBErQ7sSfRJ [new owner address] [new mapped txid] [BitName] 
```

The update tx needs to satisfy the following conditions to be considered valid:

- The update tx has to have at least one input from the previous owner address of the updated BitName
- The new mapped txid corresponds to an existing tx that has `OP_RETURN` output 
- There is no other update transaction for the same BitName in the same block that meets the first condition. Otherwise, all updates for this BitName in this block are voided. 

If all the above conditions are met, the update transaction will be in effect after the block is mined. 

## Where to go from here

You can register you own BitName today. 

A public planaria node that runs the BitName protocol is under development and will be online soon. 
