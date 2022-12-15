---
mgp: mgp-0001
title: Add Klaytn Light Client
author: Alvin Law(https://github.com/Musalaaa)
discussions-to: https://forum.mapprotocol.io/t/klaytn-light-client-development-2022-12-a-001/4569
status: Draft
created: 2022-12-22
license: Apache 2.0
---

## Simple Summary
Enable cross-chain service between Klaytn and MAPO

## Abstract

According to MAPO's tech spec, a pair of light clients needs to be built on both Klaytn main chain and MAPO relay chain to enable cross-chain service. The light client on those two chains will be responsible for verifying blocks coming from the other chain and validating user's cross-chain transactions.

## Motivation
Since Klaytn Chain has very large ecosystem, it is a must-connected chain for MAPO to expand its omnichain ecosystem. Hence, the light clients implementation is the very first approach to achieve this final goal.

## Specification
#### Core Data Structure

**BlockHeader**

```solidity
struct blockHeader {
    bytes parentHash;
    address reward;
    bytes stateRoot;
    bytes transactionsRoot;
    bytes receiptsRoot;
    bytes logsBloom;
    uint256 blockScore;
    uint256 number;
    uint256 gasUsed;
    uint256 timestamp;
    uint256 timestampFoS;
    bytes extraData;
    //json
    bytes governanceData;
    bytes voteData;
    uint256 baseFee;
}
```

**Vote**

```solidity
struct vote{
  address validator;
  bytes key;
  bytes value;
}
```

**ExtraData**

```solidity
struct extraData{
  address[] validators;
  bytes seal;
  bytes [] committedSeal;
}
```



#### BlockHeader Verification

1. **Process 'extraData' from blockHeader data structure**
   - Take the 'extraData' bytes after the 32th byte and decode to get 'extraData structure' showing above;
   - Set 'extraData.seal' and 'extraData.committedSeal' to 'Null';
   - Encode the newly-set 'extraData structure';
   - Set the 32th byte of the 'extraData' to 0;
   - Concatenate the first 32 bytes with the newly-encoded 'extraData structure' to get a new 'extraData'
   - Hash the new 'extraData' with the original blockHeader;

2. **Verify signed 'extraData.seal'**

   - Decode hash to retrieve signer for 'extraData.seal'

   - Verify if the signer is in the previous committee

3. **Verify signed set of 'extraDat.committedSeal'**

   - Decode hash+msgCommit(2) to retrieve signer for every 'commitedSeal' in the set

   - Verify if the signer is in the previous committee

   - Votes for the signer is greater than 2/3 of the total votes

     

#### Block Sync and Verification

Since Klaytn's consensus algorithm will reset set of validators every 3600 blocks, the light client is set to update and sync a block every 3600 blocks to ensure transactions is verifiable;

## Implementation

The implementation of the Klaytn light client is still in progress.

## License

This work is licensed under the Apache License, Version 2.0.