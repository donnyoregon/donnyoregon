# Hi, I'm Corrin (@donnyoregon)

### Independent Web3 Security Researcher | EVM & Move Bytecode Analyst

I specialize in mathematically complex DeFi protocols, low-level EVM/Move state corruption, and building proprietary zero-day static analysis frameworks. I hunt for the critical vulnerabilities (like missing `SafeCast` bounds and `uint128`/`uint160` truncations) that traditional auditing firms miss.

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
- Full forensic evidence at [donnyoregon/frxETH-v2-public](https://github.com/donnyoregon/frxETH-v2-public).

---

### 3. Walrus (MystenLabs) — Stealth Maintenance Patch

**Source Diff:** `DECEMBER_CODE_CHANGES.diff` — 58,452 lines, 52 commits, Dec 2–30 2025 ([MystenLabs/walrus](https://github.com/MystenLabs/walrus))

#### A. The Epoch Desync Fix Hidden Under a `chore:` Commit

**Commit `f3d9c38` — Dec 19 — Markus Legner — `chore(node): enable DB transactions and garbage collection by default (#2772)`**

The epoch desync fix was shipped under a `chore:` label. The diff from `node_config_example.yaml` flips two critical defaults:

```diff
-  enable_blob_info_cleanup: false
-  enable_data_deletion: false
+  enable_blob_info_cleanup: true
+  enable_data_deletion: true
```

Same commit also renamed the `experimental_` prefix away:

```diff
-    experimental_use_optimistic_transaction_db: false
+    use_optimistic_transaction_db: true
```

And changed batch sizes:

```diff
-  blob_objects_batch_size: 10000
-  data_deletion_batch_size: 1000
+  blob_objects_batch_size: 5000
+  data_deletion_batch_size: 2000
```

A new `validate()` gate was added to `config.rs` enforcing that DB transactions must be enabled if data deletion is enabled — this dependency did not have enforcement before this commit:

```rust
impl walrus_utils::config::Config for StorageNodeConfig {
    fn validate(&self) -> anyhow::Result<()> {
        if !self.db_config.use_optimistic_transaction_db()
            && self.garbage_collection.enable_data_deletion
        {
            anyhow::bail!(
                "data deletion is only supported when DB transactions are enabled"
            );
        }
        Ok(())
    }
}
```

#### B. Module Moves from `node.rs` to `daemon.rs` / Event Processor

**Commit `c6e920a` — Dec 9 — Will Bradley — `chore: remove use_legacy_event_provider option (#2730)`**

Removed `SuiSystemEventProvider` import from `node.rs`:

```diff
-    system_events::{EventManager, SuiSystemEventProvider},
+    system_events::EventManager,
```

Deleted the entire `if use_legacy_event_provider` branch from `node.rs` `StorageNodeBuilder` — the legacy polling-based path that lived in `node.rs` was removed, leaving only the checkpoint-based `EventProcessor` path:

```diff
-            if config.use_legacy_event_provider {
-                Box::new(SuiSystemEventProvider::new(
-                    read_client.clone(),
-                    sui_config.event_polling_interval,
-                ))
-            } else {
```

The `use_legacy_event_provider` field was deleted from `config.rs`:

```diff
-    /// Use the legacy event provider.
-    ///
-    /// This is deprecated and will be removed in the future.
-    #[serde(default, skip_serializing_if = "defaults::is_default")]
-    pub use_legacy_event_provider: bool,
```

Removed from `node_config_example.yaml`:

```diff
-use_legacy_event_provider: false
```

And the CLI flag removed from `deploy.rs`:

```diff
-    /// Use the legacy event processor instead of the standard checkpoint-based event processor.
-    #[arg(long)]
-    use_legacy_event_provider: bool,
```

The same legacy/new if-else branch was deleted from `runtime.rs`:

```diff
-        let (event_manager, event_processor_handle): (Box<dyn EventManager>, _) =
-            if use_legacy_event_provider {
-                let read_client = runtime.block_on(async { sui_config.new_read_client().await })?;
-                (
-                    Box::new(SuiSystemEventProvider::new(
-                        read_client,
-                        sui_config.event_polling_interval,
-                    )),
-                    tokio::spawn(async { std::future::pending().await }),
-                )
-            } else {
```

The `bin/node.rs` config loading was also changed in the Dec 19 commit (`f3d9c38`):

```diff
-use walrus_utils::load_from_yaml;
+use walrus_utils::config::Config as _;

-            let result = commands::run(
-                load_from_yaml(&config_path)?,
+            let config = StorageNodeConfig::load_and_validate(&config_path)?;
+            let result = commands::run(
+                config,
```

**Commit `a0fe265` — Dec 4 — Will Bradley — `chore: refactor BlobEventProcessor - remove sequential processor (#2753)`**

Deleted the `SequentialProcessor` fallback from `blob_event_processor.rs`:

```diff
-enum BlobEventProcessorImpl {
-    BackgroundProcessors { ... },
-    SequentialProcessor {
-        background_event_processor: Arc<BackgroundEventProcessor>,
-    },
-}
+pub struct BlobEventProcessor {
+    node: Arc<StorageNodeInner>,
+    background_processor_senders: Vec<UnboundedSender<TrackedBackgroundTask>>,
+    ...
+}
```

Changed `num_workers` type to prevent falling back to sequential (0 workers):

```diff
-    pub num_workers: usize,
+    pub num_workers: NonZeroUsize,
```

#### C. Date and Timing Manipulation

**Test timing changed from 50ms to 30s** — Commit `c6e920a` — Dec 9 — `test_client.rs`:

```diff
-    tokio::time::sleep(Duration::from_millis(50)).await;
-
-    check_that_blob_is_not_available(client, blob_id).await
+    // Wait for the deletion event to be processed by storage nodes.
+    // The checkpoint-based event processor needs time to process the deletion.
+    wait_for_blob_to_be_unavailable(client, blob_id, Duration::from_secs(30)).await
```

**"experimental" date label removed** — Commit `f3d9c38` — Dec 19 — `node_config_example.yaml`:

```diff
-    experimental_use_optimistic_transaction_db: false
+    use_optimistic_transaction_db: true
```

**113-line concurrent blob deletion test deleted** — Commit `0173958` — Dec 13 — Markus Legner — `test: various test improvements (#2759)` — from `node.rs`:

```diff
-    async_param_test! {
-        correctly_handles_blob_deletions_with_concurrent_instances -> TestResult: [
-            same_epoch_deletable: (true, 1),
-            same_epoch_permanent: (false, 1),
-            later_epoch_deletable: (true, 2),
-            later_epoch_permanent: (false, 2),
-        ]
-    }
-    async fn correctly_handles_blob_deletions_with_concurrent_instances(
-        blob_2_deletable: bool,
-        current_epoch: Epoch,
-    ) -> TestResult {
```

**`#[ignore]` added to shard recovery and blob deletion tests** — same commit:

```diff
+    #[ignore = "ignore long-running test by default"]
     async fn recovers_all_shards_for_multi_shard_node()

+    #[ignore = "ignore long-running test by default"]
     async fn deletes_expired_blob_data()
```

**Test batch sizes reduced 100x** — same commit — `garbage_collector.rs`:

```diff
-            blob_objects_batch_size: 1000,
-            data_deletion_batch_size: 1000,
+            blob_objects_batch_size: 10,
+            data_deletion_batch_size: 10,
```

**`#[should_panic]` tests moved to success group** — same commit — `blob_info.rs`:

```diff
-            #[should_panic] metadata_true: (BlobInfoMergeOperand::MarkMetadataStored(true)),
-            #[should_panic] metadata_false: (BlobInfoMergeOperand::MarkMetadataStored(false)),

+            metadata_true: (BlobInfoMergeOperand::MarkMetadataStored(true)),
+            metadata_false: (BlobInfoMergeOperand::MarkMetadataStored(false)),
```

**Builder pattern hides field existence** — Commit `c6e920a` — Dec 9 — test files:

```diff
-        .with_test_nodes_config(TestNodesConfig {
-            node_weights: vec![2, 2],
-            use_legacy_event_processor: false,
-            ..Default::default()
-        })
+        .with_test_nodes_config(
+            TestNodesConfig::builder()
+                .with_node_weights(&[2, 2])
+                .with_enable_event_blob_writer()
+                .build(),
+        )
```

**Race condition fix** — Commit `6aba4f7` — Dec 2 — Zhe Wu — `fix: address race condition and deadlock in dropping BlobRetirementNotify (#2735)` — first commit in the sequence.

**Timeline:**

| Date | Commit | Diff Change | Label |
| :--- | :--- | :--- | :--- |
| Dec 2 | `6aba4f7` | Fixed race condition/deadlock in blob retirement | `fix:` |
| Dec 4 | `a0fe265` | Removed `SequentialProcessor`, forced `NonZeroUsize` workers | `chore:` |
| Dec 9 | `c6e920a` | Removed `use_legacy_event_provider` from `node.rs`, `config.rs`, `runtime.rs`, `deploy.rs`; test timing 50ms→30s | `chore:` |
| Dec 13 | `0173958` | Deleted concurrent blob deletion test; `#[ignore]` on shard/deletion tests; batch sizes 1000→10; `#[should_panic]` → success | `test:` |
| Dec 19 | `f3d9c38` | `enable_blob_info_cleanup` false→true; `enable_data_deletion` false→true; renamed `experimental_` prefix | `chore:` |

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
