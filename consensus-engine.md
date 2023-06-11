# Consensus Engine

![](https://95401824-files.gitbook.io/\~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0Tecsoe4KNURIiIXeVdO%2Fuploads%2F9wQKloxWFDnm4cp3CnJG%2FConsensus-Engine.jpg?alt=media\&token=26b6f3b6-1aee-42ec-9f74-8581261d7d05)

We are dedicated to build a decentralized network that is **high-performing**, **low-cost**, and **low-consumption**. We must find a solution that achieves the above goals while allowing minimal changes to the core data structure to maintain compatibility with existing Ethernet clients. Upon investigating some popular PoA consensus protocols, we found that the consensus designs of Bor and Heco are similar to ours. Hence, we adapted some parts of

[Bor](https://polygon.technology/blog/heimdall-and-bor)

and Heco's designs to design a **Proof of Staked Authority(PoSA)** consensus protocol to achieve our goal.

### Infrastructure Components <a href="#infrastructure-components" id="infrastructure-components"></a>

1.  1\.

    **Validators**. Validators are responsible for creating and propagating blocks, verifying blocks, and ensuring the distributed ledger and distributed governance. They are rewarded with gas for each transaction in return.
2. 2\.
   * **Proposal Contract**. It provides the framework for proposal creation, voting, and KCC governance.
   * **Validators Contract.** It will select the top 29 validators with the highest number of votes and set them up as validators for the next epoch, responsible for staking and unstaking from general users and calculating and issuing block rewards to voters and validators.
   * **Reserve Pool Contract.** Its primary function is to provide incentives, including gas fees from transactions and eco-incentives provided by the Foundations.
   * **Slash Contract.** It is introduced to avoid the uncertainty of the validator's missing blocks due to various factors, which will bring uncertainty and performance problems to the KCC network. The Slash contract will record the missed block metrics of each validator. Once the metrics of a preset threshold are reached, the unclaimed rewards that the validator himself has received will be slashed and put back into the reserve pool. The slashed prizes will be shared with other validators who propagate the block.

### System Reward Distribution <a href="#system-reward-distribution" id="system-reward-distribution"></a>

The rewards in the KCC network are highly configurable. To better incentivize the healthy development of the KCC ecosystem, the Foundations provides incentive support to participants in the KCC network through subsidies. The transaction fees and the Foundations' eco-incentive are equally distributed to each vote (one KCS equals one vote). The rewards in the KCC network are highly configurable.

KCC is dedicated to building a decentralized network that is high-performing, low-cost, and low-consumption by using the Proof of Staked Authority (PoSA) consensus protocol. Those 29 validators who have received the most votes in the network will become active validators responsible for mining and validating the blocks.

Eco-participants (validators and voters) will be rewarded with transaction fees for each block and eco-incentives provided by the Foundations. To ensure network security, KCC also introduced a penalty mechanism to deal with dishonest and unstable verifiers and possible attacks.

An active validator node prepares the block header of the subsequent block.

* Create authorization snapshots
* Upon reaching an epoch block (`block number%epoch == 0`), it needs to retrieve the top active validator from the System Contract and store the selected validator set message in the `extraData` field of the block header to facilitate the implementation of the light client.

**Step 2: Finalize And Assemble**

* If the validator for the current block is not a duty validator, and the expected validator has not mined any blocks recently. In that case, the rewards of the validator will be reduced by the system contract.
* Distribute block rewards. Update active validators and recalculate missed block metrics if in an epoch block.
* Sign the header of the block and append the signature hash to extraData.
* An appropriate random time must wait before the off-duty validator can sign the block.

#### **How to Verify / Relay a Block** <a href="#how-to-verify-relay-a-block" id="how-to-verify-relay-a-block"></a>

Checks whether a block's header confirms the consensus rules when receiving the block.

* To prevent selfish validators from rushing to seal a block, verify the block time of the receiving block to be rejected if the time is earlier than the current time.
* Check the `metaData` and the `difficulty`.

There's no difference between the **Finalize** stage and the

[**Finalize And Assemble**](broken-reference)

stage, except that the witness node must verify the `extraData` information during the epoch block to complete the finalization process.

In a PoA-based consensus network, if `1/2*N+1` validators are honest, and the whole network will run safely and correctly. But in some cases, some malicious validators may gain benefits through some attacks, so we recommend that high-value or critical transactions wait for `2/3*N+1` blocks before confirming.

In the KCC network, with 29 validators, and 3 seconds for block intervals, a transaction should wait for `(2/3*29+1)*3 = 61` seconds to be finalized. Considering the double-signing attack and instability of validators in the network, KCC introduces a slash contract to make it difficult or unprofitable for malicious attackers. With this enhancement, `1/2*N+1`or even fewer blocks are enough to get most transactions to be finalized.

PoSA consensus protocol requires out-of-round validators to wait a random amount before sealing blocks. Each out-of-round validator is allowed a specific amount of time to seal blocks, which prevents validators from rushing to seal blocks. To avoid this delay and gain more profit, validators may be able to run modified full-node implementations driven by profit. Other witness nodes will discard any block generated by an out-of-round validator with an earlier blocking time.
