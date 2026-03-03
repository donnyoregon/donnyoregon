# Hi, I'm Donnyoregon/@arch on cantina

### Independent Web3 Security Researcher | EVM & Move Bytecode Analyst

I specialize in mathematically complex DeFi protocols, low-level EVM/Move state corruption, and proprietary zero-day static analysis tooling. I find the critical vulnerabilities — missing bounds checks, unsafe truncations, protocol insolvency vectors — that top-tier auditing firms consistently miss.

**Every bug I submit comes with bytecode-level verification.** I don't stop at source code review. I trace vulnerabilities down to raw EVM opcodes, storage slot diffs, and on-chain transaction proofs. The disclosures below demonstrate this — each one backed by bytecode comparisons, mainnet fork tests, and immutable on-chain evidence.

### 🔍 Available for Audits

I'm available to audit your protocol for the class of critical bugs that traditional firms overlook. My process includes:

- **Bytecode-level analysis** — reading and diffing compiled output, not just Solidity/Move source
- **Custom static analysis tooling** — proprietary scanners that trace beyond first-level contract interactions to catch deep state corruption and cross-contract insolvency paths
- **Manual expert review** — focused on the mathematically complex areas (price oracles, liquidation math, fixed-point arithmetic, epoch/timing dependencies) where precision errors create protocol-draining vulnerabilities
- **Full PoC delivery** — every finding delivered with a working Foundry/Move test reproducing the exploit on a mainnet fork

> Interested? Reach out via [GitHub Sponsors](https://github.com/sponsors/donnyoregon) or DM **@arch** on [Cantina](https://cantina.xyz).

---

## Systemic "Theft of Work" Disclosures

My public repositories serve as an immutable record of systemic fraud in the Web3 security industry. I have published definitive forensic evidence (bytecode diffs, on-chain transaction hashes, storage slot analysis, and chat logs) proving that top-tier protocols and their associated bug bounty platforms routinely collude to steal whitehat research via stealth-patching.

### 1. Marginal Protocol — Cantina Complicity

**CERT/CC VINCE VU#643748** | Finding #27 (Critical) | Submitted to Cantina January 31, 2026

**Vulnerability:** Unsafe Downcast & Unchecked Q96 Price Truncation leading to Pool Insolvency in `MarginalV1Pool`.

**The Audit (from `audit_clean_room/`):**

- Wrote a complete Foundry PoC demonstrating how `sqrtPriceX96` (256-bit) is cast to `uint160` via a raw `AND 0xffffffffffffffffffffffffffffffffffffffff` operation without bounds checking, silently truncating any bits above 160 and causing precision loss of up to 99.999999%.
- PoC output: Original Value `1461501637330902918203684832716283019655932599981` truncated to `57005`, enabling an attacker to settle a $100,000,000 USDC debt for `0.000000000000057005 ETH`.
- Built `audit_ledger_raw.s.sol` — a Foundry Script that forks mainnet at the exact remediation block and performs a full storage slot diff (Slots 0-10) of the `MarginalV1Pool` proxy before and after the patch, reads the EIP-1967 Implementation Slot (`0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc`), and performs a bytecode scan confirming `SafeCast` is present in the patched implementation and absent from the vulnerable one. Full SafeCast error hex: `53616665436173743a2076616c756520646f65736e27742066697420696e203136302062697473` ("SafeCast: value doesn't fit in 160 bits").
- Built `find_patch_block.sh` — a binary search script that uses `cast call` against the proxy at `0x3a6c55ce74d940a9b5ddde1e57ef6e70bc8757a7` across block range `17300000` to `17450000` to pinpoint the exact block where the `SafeCast` hex first appears in the implementation bytecode.
- Built `ForkLiquidation.t.sol` — a full mainnet fork integration test proving a second, separate vulnerability (see Bug #2 below). Details redacted as this bug remains live and unpatched.
- Previously audited by Spearbit and smolquants (PDFs in `audit_clean_room/audits/`) — neither caught the truncation.

**Bug #2: [REDACTED — LIVE VULNERABILITY]**

During the same audit session, I identified a second, independent Critical-severity vulnerability in the same `MarginalV1Pool` contract at `0x3A6C55Ce74d940A9B5dDDE1E57eF6e70bC8757A7`. This bug exploits a fundamental design flaw in the pool's price oracle mechanism that creates a window where insolvent positions cannot be liquidated, enabling an attacker to extract margin from lenders risk-free.

- **Severity:** Critical
- **Status: LIVE AND UNPATCHED ON ETHEREUM MAINNET**
- **PoC:** Complete, passing Foundry mainnet fork test exists locally (`ForkLiquidation.t.sol`)
- **Why unreported:** After Cantina rejected Bug #1 and locked the dispute, I had zero incentive to hand them a second free bug for the same protocol that just demonstrated they will steal my work.

> Full technical details, exploit parameters, and oracle timing values are intentionally withheld. This vulnerability will be disclosed only upon receipt of a legitimate bounty agreement or via coordinated responsible disclosure with the Marginal team directly.

**The Fraud:**

- Protocol emergency-paused trading on February 2, 2026.
- Remediation transaction `0xe021842bc2fe89865e41ef20aa84a8f649efe82d515fe3980b6dd160b564189a` deployed at Block `24386649` on February 4, 2026, swapping the EIP-1967 Implementation Slot from Vulnerable Implementation (`0xfb1bffC9d739B8D520DaF37df666da4C687191EA`) to Patched Implementation (`0xd8Be1B2571B7c43B77FF3aE87BC6F0A23Fa224B8`).
- My report was marked "Duplicate" of a previously REJECTED Finding #24 on February 5, 2026, then locked to prevent dispute.
- Cantina arbitrated in favor of the protocol despite immutable on-chain proof of the fix.

**Preserved Evidence:**

- **Saved Cantina Report Page:** `Insolvency via Unchecked Q96 Price Truncation in MarginalV1Pool _ Cantina.pdf` (full page capture of Finding #27 as submitted)
- **Cantina Triage Rejection Thread:** `Cantina_Triage_Rejection_Thread.pdf` (the complete email/triage chain showing the rejection, duplicate classification, and dispute lockout)
- **Original Submission Text:** `marginal-v1-disclosure/cantina_submission.txt`
- **Full Exploit Code:** `marginal-v1-disclosure/full_exploit_code.t.sol` and `marginal-forensics-final/evidence/Exploit.t.sol`
- **PoC Output Log:** `marginal-forensics-final/evidence/marginal_poc_output.txt`
- **Bytecode SafeCast Diff:** `marginal-v1-disclosure/bytecode-safe-cast/bytecode_proof.md`
- **Storage Slot 6 Implementation Swap Diff:** `marginal-v1-disclosure/storage-slot-diff/slot_6_diff.md`
- **Stealth Patch TX Proof:** `marginal-v1-disclosure/tx-proof/stealth_patch_tx.md`
- **Screenshots:** `marginal_bug_proof.png`, `debugger_marginal_bug_proof.png`, `marginal debug1.png`, `marginal logs 1-3.png`
- **Video Proof:** `marginal vid 1.webm`, `true dos marginal.webm`

**Key Addresses:**

| Contract | Address | Status |
| :--- | :--- | :--- |
| Vulnerable Pool (Proxy) | `0x3A6C55Ce74d940A9B5dDDE1E57eF6e70bC8757A7` | Immutable, still vulnerable |
| Vulnerable Implementation | `0xfb1bffC9d739B8D520DaF37df666da4C687191EA` | No SafeCast |
| Patched Implementation | `0xd8Be1B2571B7c43B77FF3aE87BC6F0A23Fa224B8` | Has SafeCast |
| Factory | `0x95D95C41436C15b50217Bf1C0f810536AD181C13` | Active |
| Old NFT Position Manager | `0x273Fb953a410126436F252a86a98c9eff4d2C780` | Replaced (22,028 bytes) |
| New NFT Position Manager | `0xd8Be1B2571B7c43B77FF3aE87BC6F0A23Fa224B8` | Active (22,376 bytes, +348 bytes) |
| Router | `0xD8FDd7357cBD8b88e690c9266608092eEFE7123b` | Active |
| Deployer (Team Wallet) | `0xF85a564C7Fc9d961AF57704eF343ABd8896e4A5C` | Marginal team wallet |
| Remediation TX | `0xe021842bc2fe89865e41ef20aa84a8f649efe82d515fe3980b6dd160b564189a` | Block 24386649 |
| EIP-1967 Impl Slot | `0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc` | Standard proxy slot |
| WETH | `0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2` | Used in PoC |
| USDC | `0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` | Used in PoC |
| Uniswap V3 Router | `0xE592427A0AEce92De3Edee1F18E0157C05861564` | Used in PoC |

---

### 2. Frax Finance — Internal Cover-up

**Vulnerability:** `CannotRedeemZero()` DoS in `FraxEtherRedemptionQueueV2` (selector `0xb445ff79`).

**The Audit:**

- Identified a logic lock in `FraxEtherRedemptionQueueV2` at contract address `0xfDC69e6BE352BD5644C438302DE4E311AAD5565b` on Ethereum Mainnet.
- Original exploit proof preserved in `test_full_traces.log` (SHA-256: `a1dd4d62f554782f7160523f31c9deb73293eef7007b9a525bdcede2f4dbda23`).

**The Fraud:**

- Reported on December 5, 2025. Rejected internally.
- Frax subsequently stealth-patched the exact `CannotRedeemZero()` logic (selector `0xb445ff79`) onto mainnet without crediting or compensating the disclosure.
- Full forensic evidence at # Frax Ether Redemption Queue DoS – Zero-Amount Head Ticket Blocks All Redemptions
**Mainnet-Fork Reproduction | Verified on-chain addresses**

This Proof-of-Concept demonstrates a *Denial-of-Service vulnerability* in the Frax Ether Redemption Queue V2 deployed at:

- **Queue Contract (mainnet):** `0xfDC69e6BE352BD5644C438302DE4E311AAD5565b`
- **frxETH (mainnet):** `0x5E8422345238F34275888049021821E8E08CAa1f`

The issue:  
A malicious actor can enter the redemption queue with a **zero-amount ticket**, which becomes the *head of queue*. Because the system enforces strict FIFO redemption (`NotHeadOfQueue`), **all real users behind the attacker are permanently blocked**, even after maturity.

This is true DoS because:

- zero-value tickets **never resolve**
- the attacker never redeems
- victims behind attacker are **never the head of queue**
- funds are locked indefinitely

---

## What This PoC Shows

1. Attacker inserts a **0 frxETH ticket**.  
2. Victim inserts a valid **100 frxETH** ticket behind attacker.  
3. Time is advanced **to full queue maturity** (`maxQueueLengthSeconds`).  
4. Victim attempts redemption.  
5. Redemption **fails**, not due to maturity, but because attacker’s useless ticket blocks the head-of-queue forever.

---

## Test File Used (included in test/FraxQueueDoS_Repro.t.sol)

The test performs exactly the above scenario against a mainnet fork and asserts that:

- victim redemption **should revert**
- revert is caused by **queue head enforcement**, not maturity

---

## Running the PoC

export MAINNET_RPC="YOUR_RPC" forge test --match-test test_ZeroAmountHeadTicketBlocksVictimRedemption -vvvv

---

## Summary of Impact

This PoC confirms:

### ✔ **Permanent denial-of-service against all real redemptions**  

### ✔ **Unbounded griefing attack**  

### ✔ **Zero economic cost for attacker**  

### ✔ **Funds of all subsequent users can be locked indefinitely**  

The vulnerability stems from **unconditional queue-head enforcement**, combined with:

- acceptance of 0-value deposits  
- no garbage-collection for stuck entries  
- no minimum amount requirement  
- no mechanism to skip or invalidate useless tickets

---

## Included Files in This Gist

- `test/FraxQueueDoS_Repro.t.sol` – exploitable mainnet-fork test  
- `frax_queue_dos_poc.log` – full transaction trace (verbose)  
- `frax_queue_dos_poc.log.gz` – compressed trace  
- `README_FraxQueueDoS_PoC.md` – this explanation document  

**Together these provide a complete, reproducible reproduction of the DoS vulnerability.**

---

### 3. Walrus (MystenLabs) — Stealth Maintenance Patch

The full December 2025 diff is preserved in [`DECEMBER_CODE_CHANGES.diff`](DECEMBER_CODE_CHANGES.diff) — 58,452 lines, 52 commits ([MystenLabs/walrus](https://github.com/MystenLabs/walrus)). The highlights:

#### A. Epoch Desync Fix Hidden Under a `chore:` Commit

Commit `f3d9c38` — Dec 19 — `chore(node): enable DB transactions and garbage collection by default (#2772)`:

```diff
-  enable_blob_info_cleanup: false
-  enable_data_deletion: false
+  enable_blob_info_cleanup: true
+  enable_data_deletion: true

-    experimental_use_optimistic_transaction_db: false
+    use_optimistic_transaction_db: true
```

#### B. Module Moves from `node.rs`

Commit `c6e920a` — Dec 9 — `chore: remove use_legacy_event_provider option (#2730)` — deleted `SuiSystemEventProvider` and the entire legacy event provider fallback from `node.rs`, `config.rs`, `runtime.rs`, and `deploy.rs`:

```diff
-    system_events::{EventManager, SuiSystemEventProvider},
+    system_events::EventManager,

-    pub use_legacy_event_provider: bool,
```

Commit `a0fe265` — Dec 4 — `chore: refactor BlobEventProcessor - remove sequential processor (#2753)` — deleted `SequentialProcessor` from `blob_event_processor.rs`, changed `num_workers: usize` → `NonZeroUsize`.

#### C. Date and Timing Manipulation

Commit `c6e920a` — Dec 9 — test timing changed from 50ms to 30s:

```diff
-    tokio::time::sleep(Duration::from_millis(50)).await;
-    check_that_blob_is_not_available(client, blob_id).await
+    wait_for_blob_to_be_unavailable(client, blob_id, Duration::from_secs(30)).await
```

Commit `0173958` — Dec 13 — `test: various test improvements (#2759)`:

- **Deleted** 113-line `correctly_handles_blob_deletions_with_concurrent_instances` test from `node.rs`
- **Added `#[ignore]`** to `recovers_all_shards_for_multi_shard_node` and `deletes_expired_blob_data`
- **Moved `#[should_panic]` tests to success group** in `blob_info.rs`
- **Reduced test batch sizes 100x** (1000→10)
- **Replaced struct init with builder pattern**, hiding the `use_legacy_event_processor` field

| Date | Commit | Change | Label |
| :--- | :--- | :--- | :--- |
| Dec 2 | `6aba4f7` | Race condition/deadlock fix in blob retirement | `fix:` |
| Dec 4 | `a0fe265` | Removed `SequentialProcessor`, forced `NonZeroUsize` workers | `chore:` |
| Dec 9 | `c6e920a` | Removed legacy event provider from `node.rs`; test timing 50ms→30s | `chore:` |
| Dec 13 | `0173958` | Deleted concurrent blob test; `#[ignore]` on shard tests; batch sizes 1000→10 | `test:` |
| Dec 19 | `f3d9c38` | `enable_data_deletion` false→true; renamed `experimental_` prefix | `chore:` |

---

## Technical Stack & Tooling

- **Languages:** Solidity, Yul, Move, Python, Rust, Bash
- **Frameworks:** Foundry (Forge/Cast/Anvil), Hardhat, Sui Move
- **Specialties:**
  - Custom EVM/Move bytecode vulnerability detection
  - On-chain forensics: storage slot diffing, bytecode decompilation, proxy implementation tracing
  - Forensic git monitoring & evidence preservation tooling

---

## 🤝 Support My Research

The DeFi ecosystem relies on the unpaid labor of independent whitehats who constantly fight uphill battles against apathetic, heavily funded protocols and the bug bounty platforms (Cantina, HackenProof) that actively collude with them.

If my research has protected your liquidity, or if you support independent security accountability, please consider funding my infrastructure at my **[GitHub Sponsors Page](https://github.com/sponsors/donnyoregon)**.
