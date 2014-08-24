BIP proposal of SPV improvement for light wallet.

SPV : Simplified Payment Verification (The original SPV idea was coming from Satoshi Nakamoto's paper)

## SPV mode of Bitcoin light wallet
With the growth of Bitcon blockchain data (>30GB), normal users should use light wallet for convenience.

## Current implementation of the SPV
SPV mode stores only the most recent block headers. When a user uses the light wallet for the first time, the light wallet will sync with the Bitcoin nodes from a staring point of block header.
That means wallet developers should prepare the starting point header in wallet's installation package. For little data synchronisation, developers have to upgrade the packages periodically even only for the starting point updating.
That is definitely an issue.

## How to solve this issue
We suggest that the bitcoind should add a simple service to get a block hash by block number. This improvement is very easy for Bitcoin core protocol, but will help SPV mode light wallet's a lot.

## BIP details:
**getblockhash**

The getblockhash message contains only one uint32_t field that means block height to indicate the needed block.

The Bitcoin node which receives the getblockhash information may send inv message. inv message contains only a block hash with the point height.

The inv message is supported, we only need to support the getblockhash message.

In bitcoind's rpc command, getblockhash has already been implemented, so we think that bitcoind supporting this communication is feasible.

Specific node's communication can be as the following:

1. Connecting to the node, and get version message of the node. The version message will including its current maximum block height n to represent.
2. Calculate the block height m to represent. 

The code logic of light wallet can be like:

	if n % 2016 < 100
		m = n - (n % 2016) - 2016 
	else 
		m = n - (n % 2016)
		
3. Send getblockhash message
4. Receive inv message. So we get the starting point block's hash.
5. Send getheaders message. (The getheaders message only need block hash).
6. Receive block header information and circulation 5,6 until sync to the latest block.

## Implementation of the analysis

### Safety
We ensure that the number of the block header taken at least 100. Light client verifies blocks at least 100, while also avoiding the risk of blockchain branching to some extent brought about.
