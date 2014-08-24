	BIP:
	Title: SPV Improvement for Light Wallet
	Author: Zhou Qi, Bither Team
	Status: Pre-draft
	Type: Standards Track
	Created: 

# Abstract
This BIP adds new support to the peer-to-peer protocal that allows SPV clients to get the specific block hash from nodes(full clients), so SPV clients can sync block chain from this specific block. In this way, SPV client only sync necessary and less block header.

SPV : Simplified Payment Verification (The original SPV idea was coming from Satoshi Nakamoto's paper)

# Motivation

With the growth of Bitcon blockchain data (>30GB), normal users should use SPV clients for convenience.

SPV clients stores only the most recent block headers. When a user uses the SPV clients for the first time, SPV clients will sync with the Bitcoin nodes from a staring point of block header.

That means wallet developers should prepare the starting point header in wallet's installation package. For little data synchronisation, developers have to upgrade the packages periodically even only for the starting point updating.
That is definitely an issue.

# Design rationale

We suggest that nods should add a simple service to get a block hash by block number. This improvement is very easy for Bitcoin core protocol, but will help SPV clients's a lot.

# Specification

## New message

We start by adding one new messages to the protocol:

**getblockhash**

The getblockhash message contains only one uint32_t field that means block height to indicate the needed block.

The Bitcoin node which receives the getblockhash information may send inv message. inv message contains only a block hash with the point height.

The inv message is supported, we only need to support the getblockhash message.

In bitcoind's rpc command, getblockhash has already been implemented, so we think that bitcoind supporting this communication is feasible.

Specific node's communication can be as the following:

1. Connecting to the node, and get version message of the node. The version message will including its current maximum block height n to represent.
2. Calculate the block height m to represent. 

The code logic of light wallet can be 3. like:

	if n % 2016 < 100
		m = n - (n % 2016) - 2016 
	else 
		m = n - (n % 2016)	

After get the starting block hash we can sync block chain like.

3. Send getblockhash message
4. Receive inv message. So we get the starting point block's hash.
5. Send getheaders message. (The getheaders message only need block hash).
6. Receive block header information and circulation 5,6 until sync to the latest block.

### Safety
We ensure that the number of the block header taken at least 100. Light client verifies blocks at least 100, while also avoiding the risk of blockchain branching to some extent brought about.

# Copyright

This document is placed in the public domain.