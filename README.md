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
