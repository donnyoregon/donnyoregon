# Hi, I'm Donnyoregon / @arch on Cantina

### Independent Web3 Security Researcher | EVM & Move Bytecode Analyst

I specialize in mathematically complex DeFi protocols, low-level EVM/Move state corruption, and proprietary zero-day static analysis tooling. I find the critical vulnerabilities — missing bounds checks, unsafe truncations, protocol insolvency vectors — that TOP-TIER auditing firms consistently miss.

**I am available to audit your project for real security flaws that the big firms miss.** My tooling traces these sorts of critical project insolvency bugs using manual bytecode reading and custom scanners that go deeper than basic first-level address checks. Every bug I submit comes with bytecode-level verification and mainnet fork tests.

> **Interested?** Reach out github or email clarkcorrin@gmail.com

---

## Systemic "Theft of Work" Disclosures

The following repositories serve as an immutable record of systemic fraud in the Web3 security industry.

### 1. Marginal Protocol — Cantina Complicity

**CERT/CC VINCE VU#643748** | Finding #27 (Critical) | Submitted Jan 31, 2026

**Vulnerability:** Unsafe Downcast & Unchecked Q96 Price Truncation leading to Pool Insolvency.

- **Root Cause:** `sqrtPriceX96` (256-bit) cast to `uint160` via raw `AND` operation without bounds checking.
- **Impact:** Attacker settles $100M debt for $0.000000000000057005 ETH (99.999% loss).
- **The Fraud:** Protocol paused (Feb 2), stealth-patched (Feb 4, Block 24386649, TX `0xe021...89a`), and then rejected my report as a "duplicate" of a previously REJECTED finding.

**Preserved Evidence (Archived in [marginal-v1-disclosure](https://github.com/donnyoregon/marginal-v1-disclosure)):**

- **Full Forensic Breakdown:** [`report-summary.md`](https://github.com/donnyoregon/marginal-v1-disclosure/blob/main/report-summary.md)
- **On-Chain Proof:** [`bytecode_proof.md`](https://github.com/donnyoregon/marginal-v1-disclosure/blob/main/bytecode-safe-cast/bytecode_proof.md)
- **Storage Slot Diff:** [`slot_6_diff.md`](https://github.com/donnyoregon/marginal-v1-disclosure/blob/main/storage-slot-diff/slot_6_diff.md)
- **Stealth Patch TX Proof:** [`stealth_patch_tx.md`](https://github.com/donnyoregon/marginal-v1-disclosure/blob/main/tx-proof/stealth_patch_tx.md)
- **Exploit Code:** [`full_exploit_code.t.sol`](https://github.com/donnyoregon/marginal-v1-disclosure/blob/main/full_exploit_code.t.sol)
- **Cantina Triage Thread:** [`Cantina_Triage_Rejection_Thread.pdf`](https://github.com/donnyoregon/marginal-v1-disclosure/blob/main/cantina-rejection/Cantina_Triage_Rejection_Thread.pdf)

---

### 2. Frax Finance — Internal Cover-up

**Vulnerability:** `CannotRedeemZero()` DoS in `FraxEtherRedemptionQueueV2` (selector `0xb445ff79`).

**The Fraud:**

- Discovered Dec 2, 2025. Submitted Dec 5. Rejected internally as "impossible."
- Frax subsequently upgraded the contract at `0xfDC69e6BE352BD5644C438302DE4E311AAD5565b` to include the `CannotRedeemZero()` logic without disclosure or payment.

> **Full Case Evidence:** [Gist: Frax Ether Redemption Queue DoS](https://gist.github.com/donnyoregon/077da021e0a2534dc811cb7ab96b40e6)

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
