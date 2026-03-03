# Hi, I'm Corrin (@donnyoregon) 🛡️

### Independent Web3 Security Researcher | EVM & Move Bytecode Analyst

I specialize in mathematically complex DeFi protocols, low-level EVM/Move state corruption, and building proprietary zero-day static analysis frameworks. I hunt for the critical vulnerabilities (like missing `SafeCast` bounds and `uint128`/`uint160` truncations) that traditional auditing firms miss.

---

## 🔍 Systemic "Theft of Work" Disclosures

My public repositories serve as an immutable record of systemic fraud in the Web3 security industry. I have published definitive forensic evidence (bytecode diffs, on-chain transaction hashes, storage slot analysis, and chat logs) proving that top-tier protocols and their associated bug bounty platforms routinely collude to steal whitehat research via stealth-patching.

### 1. Marginal Protocol — Cantina Complicity

**Vulnerability:** Unsafe Downcast & Unchecked Q96 Price Truncation leading to Pool Insolvency in `MarginalV1Pool`.

**The Audit (from `audit_clean_room/`):**

- Wrote a complete Foundry PoC demonstrating how `sqrtPriceX96` (256-bit) is cast to `uint160` via a raw `AND 0xffffffffffffffffffffffffffffffffffffffff` operation without bounds checking, silently truncating any bits above 160 and causing precision loss of up to 99.999999%.
- PoC output: Original Value `1461501637330902918203684832716283019655932599981` truncated to `57005`, enabling an attacker to settle a $100,000,000 USDC debt for `0.000000000000057005 ETH`.
- Built `audit_ledger_raw.s.sol` — a Foundry Script that forks mainnet at the exact remediation block and performs a full storage slot diff (Slots 0-10) of the `MarginalV1Pool` proxy before and after the patch, reads the EIP-1967 Implementation Slot (`0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc`), and performs a bytecode scan confirming `SafeCast` is present in the patched implementation and absent from the vulnerable one. Full SafeCast error hex: `53616665436173743a2076616c756520646f65736e27742066697420696e203136302062697473` ("SafeCast: value doesn't fit in 160 bits").
- Built `find_patch_block.sh` — a binary search script that uses `cast call` against the proxy at `0x3a6c55ce74d940a9b5ddde1e57ef6e70bc8757a7` across block range `17300000` to `17450000` to pinpoint the exact block where the `SafeCast` hex first appears in the implementation bytecode.
- Built `ForkLiquidation.t.sol` — a full mainnet fork integration test proving a second, separate vulnerability (see Bug #2 below). Details redacted as this bug remains live and unpatched.
- Previously audited by Spearbit and smolquants (PDFs in `audit_clean_room/audits/`) — neither caught the truncation.
- Submitted as Finding #27 (Critical) to Cantina on January 31, 2026.

**Bug #2: [REDACTED — LIVE VULNERABILITY]**

During the same audit session, I identified a second, independent Critical-severity vulnerability in the same `MarginalV1Pool` contract at `0x3A6C55Ce74d940A9B5dDDE1E57eF6e70bC8757A7`. This bug exploits a fundamental design flaw in the pool's price oracle mechanism that creates a window where insolvent positions cannot be liquidated, enabling an attacker to extract margin from lenders risk-free.

- **Severity:** Critical
- **Status:** ⚠️ **LIVE AND UNPATCHED ON ETHEREUM MAINNET**
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

### 2. MystenLabs / Walrus — HackenProof Complicity

**Vulnerability:** Split-Brain Exploit (WALRUS-SC-81) & Epoch Subsidy State Corruption in the Walrus decentralized storage network.

**The Audit:**

- Identified a critical split-brain race condition in `garbage_collection.rs` where blob registration (Address `0x15a62`, Stack Offset `[rsp+0x60]`) reads Epoch 42 while the deletion check (Address `0x15baf`, Stack Offset `[rsp+0x68]`) reads Epoch 43, creating a non-atomic state where a node reports "Registered" while simultaneously executing "Delete", causing unauthorized permanent data loss.
- Submission date: December 1, 2025, against vulnerable commit `50702d7` (full hash: `6aba4f7b87359f3d8be3ec57e09fbec7465e1d6e`).
- Wrote three independent proof scripts (`prove_patch.sh`, `prove_patch_v2.sh`, `prove_patch_red.sh`) performing git time-travel from the Dec 1st vulnerable commit to `main`, demonstrating the function signature change in `garbage_collection.rs`:
  - **Before (Dec 1, Vulnerable):** `fn start_garbage_collection_task` reads `contract_epoch` internally with no external epoch input.
  - **After (Patched on `main`):** `fn start_garbage_collection_task` now accepts `epoch: Epoch` as an explicit parameter, eliminating the race condition.
- Built a full dual-node protocol attack script (`walrus_split_brain_exploit.sh`) that launches two `walrus-node` instances to force the exact race condition between `node.rs` Line 3118 (The Check) and Line 1735 (The Action).
- Tracked epoch subsidy anomalies on Sui Mainnet via the Walrus System Object at `0x2134d52768ea07e8c43570ef975eb3e4c27a39fa6396bef985b5abc58d03ddd2`.
- Compared Testnet contracts (`testnet-contracts/walrus_subsidies/sources/walrus_subsidies_inner.move`) against Mainnet contracts (`mainnet-contracts/walrus_subsidies/sources/walrus_subsidies_inner.move`) to identify stealth-patched logic differences in `walrus_subsidies.move` and `walrus_context.move`.

**The Fraud (captured live by `watchdog_verify/watchdog.sh`):**

- Built a custom Forensic Git Watchdog (`watchdog.sh`) that clones a bare `--mirror` of a target repository on a 60-second polling loop. On every fetch, it compares `git show-ref` snapshots to detect any force-push or history rewrite. When a force-push is detected, the watchdog immediately tags the deleted commit locally (preventing garbage collection) and exports a forensic evidence report.
- **Live capture on February 20, 2026 at 15:59:45 UTC:** The watchdog detected a **HISTORY REWRITE** (force-push) on `refs/heads/master`.
  - **Deleted/Overwritten Commit:** `046efb93f35344ae9ddc153e1f64f6ffd75a4df3`
  - **New (Replacement) Commit:** `5d4716592516074b80c26bdb19b33f316be06b4c`
  - **Evidence locked as tag:** `evidence_LOCK_refs_heads_master_046efb9`
  - **Proof file exported:** `EVIDENCE_evidence_LOCK_refs_heads_master_046efb9.txt`
- MystenLabs refactored their GitHub repository to obscure the stealth patch: the vulnerable `garbage_collection.rs` function signature was silently changed on `main` without a dedicated security advisory, CVE, or changelog entry.
- HackenProof assisted in the process, allowing MystenLabs to clean up the commit history before closing the bounty.
- Full evidence archive preserved in `walrus_replay/walrus_evidence/` including testnet vs mainnet contract diffs, Walrus whitepapers (v1, v2), and docker service configurations.

---

### 3. Frax Finance — Internal Cover-up

**Vulnerability:** `CannotRedeemZero()` DoS in `FraxEtherRedemptionQueueV2` (selector `0xb445ff79`).

**The Audit:**

- Identified a logic lock in `FraxEtherRedemptionQueueV2` at contract address `0xfDC69e6BE352BD5644C438302DE4E311AAD5565b` on Ethereum Mainnet.
- Original exploit proof preserved in `test_full_traces.log` (SHA-256: `a1dd4d62f554782f7160523f31c9deb73293eef7007b9a525bdcede2f4dbda23`).

**The Fraud:**

- Reported on December 5, 2025. Rejected internally.
- Frax subsequently stealth-patched the exact `CannotRedeemZero()` logic (selector `0xb445ff79`) onto mainnet without crediting or compensating the disclosure.
- Full forensic evidence at [donnyoregon/frxETH-v2-public](https://github.com/donnyoregon/frxETH-v2-public).

---

## 🛠️ Technical Stack & Tooling

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
