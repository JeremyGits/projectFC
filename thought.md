# Fleet Credits: A Peer-to-Peer Electronic Credit System

**Version 1.0**  
**Based on Dogecoin Core Architecture**

---

## Abstract

Fleet Credits (FC) is a decentralized electronic credit system enabling direct value exchange between parties without financial intermediaries. The system uses a decentralized timestamp server implemented through a Scrypt-based Proof-of-Work consensus mechanism. Fleet Credits diverges from traditional cryptocurrency models by implementing post-scarcity economic principles: unlimited supply issuance, zero-fee micro-transactions, and contribution-based reward mechanisms. This paper proposes the technical architecture, economic model, and implementation strategy for a blockchain-based credit system optimized for high-transaction velocity and equitable contribution recognition.

---

## 1. Introduction

Traditional payment systems rely on financial institutions to process transactions. While the system works well for most transactions, it still suffers from inherent trust-based weaknesses. Fleet Credits proposes an alternative transaction system based on cryptographic proof rather than trust.

### 1.1 Motivation

Current cryptocurrency models enforce scarcity through supply caps and halving mechanisms. However, in a post-scarcity economy where production capacity approaches infinite abundance, a different monetary model is required. Fleet Credits implements:

- **Continuous issuance** via fixed block rewards without supply constraints
- **Zero-fee micro-transactions** to enable high-velocity exchanges
- **Contribution rewards** to incentivize community value creation
- **Cross-value interoperability** with fiat and existing cryptocurrencies

### 1.2 Core Innovations

The system introduces three novel mechanisms beyond traditional blockchain architectures:

1. **Dynamic Fee Structure**: Fees scale to transaction size, enabling zero-cost micro-payments
2. **Contribution Verification**: Cryptographic proofs enable trustless validation of community contributions
3. **Community Reserve**: Automated fee redirection to fund public goods

---

## 2. Network Architecture

Fleet Credits implements a peer-to-peer distributed timestamp server on a hash-based Proof-of-Work chain.

### 2.1 Timestamp Server

The timestamp server consists of the following algorithm:

1. Take the hash of a block of contributions to be timestamped
2. Broadcast the hash widely
3. Wait for enough work to be performed on the hash
4. The timestamp proves that the data existed at the time

Each timestamp includes the previous timestamp in its hash, forming a chain. Each timestamp is incorporated into the hash of its successor, reinforcing the chain.

### 2.2 Hash-Based Proof-of-Work Chain

The network timestamps transactions by hashing them into an ongoing chain of hash-based proof-of-work, forming a record that cannot be changed without redoing the proof-of-work. The longest chain not only serves as proof of the sequence of events witnessed, but proof that it came from the largest pool of CPU power.

The proof-of-work involves scanning for a value that when hashed, such as with SHA-256, the hash begins with a number of zero bits. The average work required is exponential in the number of zero bits required and can be verified by executing a single hash.

For Fleet Credits, the hash function is Scrypt, which provides:

1. Memory-hard properties preventing ASIC dominance
2. Accessibility to standard hardware miners
3. Energy efficiency compared to SHA-256

### 2.3 Network Protocol

The steps to run the network are as follows:

1. New transactions are broadcast to all nodes
2. Each node collects new transactions into a block
3. Each node works on finding a difficult proof-of-work for its block
4. When a node finds a proof-of-work, it broadcasts the block to all nodes
5. Nodes accept the block only if all transactions in it are valid and not already spent
6. Nodes express their acceptance of the block by working on creating the next block in the chain, using the hash of the accepted block as the previous hash

---

## 3. Transaction Model

### 3.1 Standard Transactions

A transaction moves value from inputs to outputs. Each input is a reference to a previous transaction's output. A double-spend is prevented by including the previous transaction in the hash of the current block, which serves as a proof-of-work.

### 3.2 Fee Structure

Transaction fees are calculated dynamically based on transaction size and network conditions:

```
fee(tx) = {
    0,                                      if total_output(tx) < 1000 FC
    min(tx_size × 0.2 FC/kB, cap),          otherwise
}
```

Where:
- `total_output(tx)` is the sum of all output values in FC
- `tx_size` is the transaction size in kilobytes
- `cap` is a configurable maximum fee (default: 100 FC)

The 1000 FC threshold enables frictionless micro-transactions while maintaining network security through fees on larger transactions.

### 3.3 Community Reserve Allocation

A portion of non-zero fees is allocated to the Community Reserve:

```
reserve_allocation = fee(tx) × 0.01
```

The Community Reserve is managed by a multisig wallet with governance-controlled spending rules.

### 3.4 Transaction Verification

A transaction is valid if:

1. All inputs reference valid, unspent previous outputs
2. The sum of input values equals or exceeds the sum of output values (including fees)
3. The transaction is properly cryptographically signed
4. The transaction size does not exceed network limits

---

## 4. Contribution Reward System

### 4.1 Motivation

Traditional Proof-of-Work rewards only computational effort. Fleet Credits extends rewards to include verifiable community contributions, creating economic incentives for public goods.

### 4.2 Contribution Transaction Format

A contribution transaction (`contrib_tx`) is a special transaction type that includes:

```
ContribTransaction = {
    timestamp: uint64,
    contributor: public_key,
    contribution_type: enum,
    proof_data: bytes,
    signature: signature
}
```

Where `contribution_type` can be:
- CODE_CONTRIBUTION: Open-source code commits
- CHARITABLE_ACT: Verified charitable donations
- CREATIVE_WORK: Art, writing, music with IPFS hash
- EDUCATIONAL_CONTENT: Teaching materials or courses

### 4.3 Proof Verification

For automated verification (e.g., GitHub commits):

```
verify_code_contribution(tx) → bool {
    commit_hash = tx.proof_data
    if github_api.verify_commit(commit_hash):
        return true
    return false
}
```

For manual verification (e.g., charitable acts):

```
verify_charitable_contribution(tx, oracles) → bool {
    votes = []
    for oracle in oracles:
        if oracle.verify(tx.proof_data):
            votes.append(1)
        else:
            votes.append(0)
    return sum(votes) >= (oracles.length * 3 / 5)
}
```

### 4.4 Block Reward Calculation

The block reward is calculated as:

```
base_reward = 10000 FC

bonus_multiplier = {
    1.05,   if contrib_tx verified and bonus level = LOW
    1.10,   if contrib_tx verified and bonus level = MEDIUM
    1.15,   if contrib_tx verified and bonus level = HIGH
    1.20,   if contrib_tx verified and bonus level = CRITICAL
}

block_reward = base_reward × bonus_multiplier
```

The bonus level is determined by:
- Contribution type
- Impact assessment
- Verification method (automated vs. oracle-based)
- Historical contribution pattern

### 4.5 Replay Prevention

Contribution transactions include timestamps and are bound to specific block heights to prevent replay across different blocks or chains. The same contribution proof cannot be used in multiple blocks.

---

## 5. Consensus Mechanism

### 5.1 Proof-of-Work Parameters

Fleet Credits uses Scrypt as its Proof-of-Work hash function:

```
scrypt(passphrase, salt, N, r, p, dkLen)
```

Where:
- `N` = CPU/memory cost parameter (2^14)
- `r` = block size parameter (8)
- `p` = parallelization parameter (1)
- Target block time: 60 seconds

### 5.2 Difficulty Adjustment

The difficulty adjusts every block to maintain the target block time:

```
new_difficulty = old_difficulty × (target_time / actual_time)
```

Where `actual_time` is the time between the current block and the block 144 blocks ago.

### 5.3 Network Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| Block Time | 60s | Target time between blocks |
| Block Size | 2 MB | Maximum block size |
| Base Reward | 10,000 FC | Fixed per-block issuance |
| Difficulty Adjustment | Every block | Dynamic adjustment |
| Halving | None | Unlimited supply |

---

## 6. Economic Model

### 6.1 Money Supply

Fleet Credits implements unlimited money supply:

```
annual_supply_increase = 10,000 FC/block × 525,600 blocks/year
                       = 5.256 × 10^9 FC/year
```

Unlike fixed-supply cryptocurrencies, Fleet Credits follows a post-scarcity model where issuance continues indefinitely.

### 6.2 Value Mechanism

In the absence of artificial scarcity, value derives from:

1. **Velocity**: High transaction throughput creates demand
2. **Utility**: Contribution rewards drive adoption
3. **Interoperability**: DEX integration enables price discovery
4. **Network Effects**: Growing user base increases utility

### 6.3 Inflation Management

While supply increases indefinitely, inflation rate decreases over time:

```
inflation_rate(t) = annual_issuance / circulating_supply(t)
```

As `circulating_supply(t)` grows, `inflation_rate(t)` asymptotically approaches zero.

### 6.4 Community Reserve Dynamics

The Community Reserve accumulates fees over time:

```
reserve_balance(T) = Σ (fee(tx) × 0.01) for all tx in blocks 0..T
```

Governance rules determine spending from the reserve, typically funding:
- Open-source development grants
- Educational initiatives
- Infrastructure improvements
- Security audits

---

## 7. Governance

### 7.1 On-Chain Voting

Fleet Credits implements on-chain governance through voting transactions:

```
VoteTransaction = {
    proposal_id: hash256,
    voter: public_key,
    vote: enum(YES, NO, ABSTAIN),
    voting_power: amount,
    signature: signature
}
```

Voting power is proportional to stake:

```
voting_power = min(FC_balance, cap_per_address)
```

Where `cap_per_address` prevents whale domination.

### 7.2 Governance Proposals

Proposals require:
1. Submission fee: 10,000 FC (prevents spam)
2. Minimum stake: 100,000 FC to submit
3. Voting period: 7 days
4. Quorum: 10% of circulating supply

### 7.3 Proposal Types

1. **Parameter Changes**: Adjust fee caps, bonus rates, etc.
2. **Reserve Spending**: Allocate Community Reserve funds
3. **Feature Proposals**: Request new functionality
4. **Oracle Elections**: Add or remove oracle validators

### 7.4 Execution

Approved proposals are executed automatically through protocol upgrades or multisig wallet transactions for the Community Reserve.

---

## 8. Cross-Value Integration

### 8.1 Decentralized Exchange Integration

Fleet Credits integrates with DEX protocols for liquidity:

```
DEXSwap = {
    input: amount_in FC,
    output_token: address,
    output_min: amount,
    deadline: timestamp
}
```

Initial liquidity pairs:
- FC/USDC (fiat peg)
- FC/BTC (crypto integration)
- FC/ETH (DeFi compatibility)

### 8.2 Payment Gateway API

Merchants can accept Fleet Credits via payment processors that:
1. Convert FC to fiat at market rates
2. Handle transaction fees
3. Provide instant settlement

```
MerchantAPI = {
    create_invoice(amount_fiat, description) → address,
    verify_payment(address) → bool,
    withdraw_to_bank(fc_address) → transaction_id
}
```

### 8.3 Oracle Price Feeds

Price oracles provide real-time FC/USD rates:

```
getPriceFeed() → {
    fc_usd_rate: float,
    timestamp: uint64,
    confidence_interval: float
}
```

Price feeds are aggregated from multiple sources to prevent manipulation.

---

## 9. Security Considerations

### 9.1 51% Attack Prevention

Fleet Credits resists 51% attacks through:

1. **Consensus Finality**: 12 confirmations (~12 minutes) for large transactions
2. **Network Monitoring**: Alert systems for hashrate centralization
3. **Economic Incentives**: Attack cost exceeds potential benefit

### 9.2 Oracle Security

Oracle collusion is prevented by:
1. Requiring 3 of 5 oracles to agree
2. Oracle stake requirements (500,000 FC minimum)
3. Reputation system with slashing for malicious behavior
4. Rotation mechanism for oracle selection

### 9.3 Spam Prevention

Zero-fee transactions could enable spam. Mitigations include:

1. **Rate Limiting**: Maximum transactions per address per hour
2. **Minimum Output Size**: 546 FC minimum prevents dust
3. **Mempool Size Limits**: Reject transactions exceeding thresholds
4. **Dynamic Fee Escalation**: Fees increase with transaction count

### 9.4 Smart Contract Security

Governance contracts are:
1. Formally verified where possible
2. Professionally audited before deployment
3. Time-locked for upgrades
4. Multi-signature controlled

---

## 10. Implementation

### 10.1 Codebase

Fleet Credits is a fork of Dogecoin Core (v1.14), modified for:

1. Block reward adjustment (10,000 FC)
2. Fee structure implementation
3. Contribution transaction support
4. Oracle integration framework
5. Governance voting system

### 10.2 Network Deployment

**Testnet Phase** (P2P Port 44556, RPC Port 44555):
- Launch date: Q2 2026
- Purpose: Feature testing, economic modeling
- Duration: 6 months minimum

**Mainnet Phase** (P2P Port 22556, RPC Port 22555):
- Launch date: Q4 2026
- Genesis block: Fair launch, no premine
- Distribution: Mining from block 1

### 10.3 Wallet Implementation

The Fleet Credits wallet provides:

1. **Standard Features**: Send, receive, stake
2. **Contribution Interface**: Submit and track contributions
3. **Governance Voting**: Cast votes on proposals
4. **DEX Integration**: Swap FC for other assets
5. **Price Display**: Real-time FC/USD rates

### 10.4 Mining Compatibility

Fleet Credits is compatible with:
- Any Scrypt-capable miner
- CPU mining (accessibility)
- GPU mining (efficiency)
- ASIC mining (specialized hardware)

---

## 11. Future Research Directions

### 11.1 Layer-2 Scaling

Potential layer-2 solutions include:
- State channels for instant payments
- Sidechains for additional functionality
- Off-chain oracle networks for contribution verification

### 11.2 Enhanced Contribution Types

Future contribution categories may include:
- Environmental actions (carbon credits, reforestation)
- Healthcare contributions (medical data sharing, research participation)
- Infrastructure development (mesh networks, open-source hardware)

### 11.3 Privacy Enhancements

Optional privacy features:
- Confidential transactions
- Zero-knowledge proofs
- Mixing protocols

### 11.4 AI Integration

Long-term vision:
- AI-assisted contribution verification
- Automated contribution quality assessment
- Personalized reward recommendations

---

## 12. Conclusion

Fleet Credits proposes a novel approach to cryptocurrency design by prioritizing post-scarcity economics over artificial scarcity. The system enables high-velocity micro-transactions, incentivizes community contributions, and maintains decentralization through Proof-of-Work consensus.

Key innovations include the contribution reward system, dynamic fee structure, and community-governed reserve allocation. These mechanisms create sustainable economic incentives for public goods production while maintaining the security and decentralization of traditional blockchain systems.

The unlimited supply model, while unconventional, aligns with post-scarcity economic principles where production capacity becomes effectively infinite. Value emerges from utility, velocity, and network effects rather than artificial scarcity.

Implementation will proceed through rigorous testing on testnet, security audits, and gradual mainnet deployment. The open-source nature of the project enables community participation in development, governance, and ecosystem growth.

---

## References

[1] Nakamoto, S. (2008). Bitcoin: A Peer-to-Peer Electronic Cash System.

[2] Dogecoin Core (2024). Dogecoin: The fun and friendly internet currency. https://github.com/dogecoin/dogecoin

[3] Larimer, D. (2013). Delegated Proof-of-Stake Consensus.

[4] Back, A. (2002). Hashcash - A Denial of Service Counter-Measure.

[5] Percival, C. (2009). Stronger Key Derivation via Sequential Memory-Hard Functions.

[6] Fleet Credits Core Repository (2026). https://github.com/JeremyGits/DogecoinProposal

---

## Appendix A: Transaction Serialization

```
Standard Transaction:
+---------------+---------------+----------+----------+
| Field         | Type          | Size     | Value    |
+---------------+---------------+----------+----------+
| version       | uint32        | 4 bytes  | 1        |
| input_count   | varint        | 1-9      | N        |
| inputs[]      | TxInput[]     | variable | ...      |
| output_count  | varint        | 1-9      | M        |
| outputs[]     | TxOutput[]    | variable | ...      |
| locktime      | uint32        | 4 bytes  | time/height |
+---------------+---------------+----------+----------+

Contribution Transaction:
+---------------+---------------+----------+----------+
| Field         | Type          | Size     | Value    |
+---------------+---------------+----------+----------+
| version       | uint32        | 4 bytes  | 1        |
| tx_type       | uint8         | 1 byte   | 0x01     |
| contributor   | pubkey        | 33 bytes | ...      |
| contrib_type  | uint8         | 1 byte   | ...      |
| proof_data    | varbytes      | variable | ...      |
| timestamp     | uint64        | 8 bytes  | ...      |
| signature     | signature     | 64 bytes | ...      |
+---------------+---------------+----------+----------+
```

## Appendix B: Network Messages

```
version:
+---------------+--------+
| Field         | Size   |
+---------------+--------+
| version       | 4      |
| services      | 8      |
| timestamp     | 8      |
| addr_recv     | 26     |
| addr_from     | 26     |
| nonce         | 8      |
| user_agent    | var    |
| start_height  | 4      |
| relay         | 1      |
+---------------+--------+

inv (inventory):
+---------------+--------+
| Field         | Size   |
+---------------+--------+
| count         | varint |
| inventory[]   | 36*N   |
+---------------+--------+

block:
+---------------+--------+
| Field         | Size   |
+---------------+--------+
| header        | 80     |
| txn_count     | varint |
| transactions  | var    |
+---------------+--------+
```

## Appendix C: Mathematical Notations

- `H(x)`: Cryptographic hash function
- `H_S(x)`: Scrypt hash function
- `SIG_k(H(x))`: Digital signature of hash x with key k
- `PK_a`: Public key of address a
- `SK_a`: Private key of address a
- `F(x)`: Fee calculation function
- `R(B)`: Block reward for block B
- `T(B)`: Timestamp of block B
- `D(B)`: Difficulty of block B

---

**Document Version:** 1.0  
**Last Updated:** 2026-01-XX  
**Authors:** Fleet Credits Development Team  
**License:** MIT
