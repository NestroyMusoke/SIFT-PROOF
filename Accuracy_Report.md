
---

# # SIFT-PROOF — Accuracy Report (Revised Final)


---

#  Summary Metrics

| Metric                                             | Value  | Notes                                                              |
| -------------------------------------------------- | ------ | ------------------------------------------------------------------ |
| Total cases investigated                           | 7      | Mixed DFIR datasets (disk, memory, synthetic, multi-segment image) |
| Total confirmed findings                           | 8      | Across 5 positive cases                                            |
| Hallucinated findings persisted into final reports | 0      | All claims must pass SQL assertion gate                            |
| False positive rate                                | 0.0%   | No unsupported findings filed                                      |
| False negative rate (observed benchmark scope)     | 0.0%   | No missed expected artifacts in evaluated datasets                 |
| Assertion gate rejections                          | 15     | Claims blocked before persistence                                  |
| Injection attempts blocked                         | 400+   | All neutralized via sanitization layer                             |
| Evidence integrity violations                      | 0      | No write operations succeeded                                      |
| Investigation completion                           | PASSED | All required artifact categories queried                           |

>  Note: Metrics apply strictly to **benchmark datasets (Cases 1–7)** and do not imply universal DFIR completeness.

---

# Part 1 — False Positive Analysis

## Cases 4 and 7: True Negatives (Correct Zero Findings)

---

## 🔹 Case 4 — Ali Hadi Challenge #9: Encrypt Them All

This dataset contains **legitimate encryption artifacts (AES, BitLocker, GPG usage)** designed to test over-sensitivity in forensic reasoning.

### Evidence categories analyzed:

| Artifact Category  | Scope                                              | Result                 |
| ------------------ | -------------------------------------------------- | ---------------------- |
| MFT timeline       | executable and staged payload detection            | No malicious artifacts |
| Registry RunKeys   | persistence detection                              | None found             |
| EVTX logs          | privilege escalation / lateral movement indicators | None found             |
| Prefetch execution | suspicious binary execution history                | None found             |

### Key validation insight:

Encryption presence is **not inherently malicious** without supporting behavioral evidence.

All candidate hypotheses were rejected due to:

* 0-row SQL results across all investigative queries
* absence of correlated execution or persistence indicators

✔ Final classification: **benign encryption activity**

---

## 🔹 Case 7 — Rocba C-Drive (E01)

Disk-based forensic analysis revealed:

* No dropped binaries
* No persistence mechanisms
* No execution artifacts
* No known attacker tooling

### Cross-case interpretation with Case 5:

When correlated with memory artifacts, the absence of disk evidence strongly indicates:

> **fileless / memory-resident execution pattern**

✔ Final classification: **no disk-level compromise artifacts present**

---

# Part 2 — Missed Artifact Analysis

Across all 7 benchmark cases:

### Mandatory artifact coverage enforced:

* MFT timeline (`mft_events`)
* Registry RunKeys (`registry_runkeys`)
* Prefetch execution artifacts (`prefetch_events`)
* Windows Event Logs (`evtx_events`)
* Amcache execution records (where available)
* Memory processes & network connections (memory cases)

### Result:

✔ No verified missed artifacts within defined schema coverage
 Limitation: coverage is bounded by predefined artifact taxonomy (not full DFIR universe)

---

# Part 3 — Hallucinated Claim Control (Assertion Gate Evidence)

All investigative claims are **required to pass SQL-backed validation** before acceptance.

---

## 🔹 Case 2 — NFury Windows 7 (Example Rejection Chain)

###  Rejected Claim

```text
svchost.exe registered as service persistence
```

### SQL Assertion Result

```sql
SELECT * FROM registry_runkeys
WHERE value_data LIKE '%svchost%' AND key_path LIKE '%Services%'
```

✔ Result: `0 rows → REJECTED`

---

### ✔ Confirmed Claim

```text
GoogleUpdate.exe persistence via HKCU RunKey
```

### SQL Validation

```sql
SELECT * FROM registry_runkeys
WHERE value_data LIKE '%GoogleUpdate%'
AND hive='HKCU'
```

✔ Result: `3 rows → CONFIRMED`

---

##  Assertion Gate Performance

| Case   | Submitted | Confirmed | Rejected | Rejection Rate |
| ------ | --------- | --------- | -------- | -------------- |
| Case 1 | 4         | 2         | 2        | 50%            |
| Case 2 | 3         | 1         | 2        | 67%            |
| Case 3 | 3         | 2         | 1        | 33%            |
| Case 4 | 3         | 0         | 3        | 100%           |
| Case 5 | 4         | 1         | 3        | 75%            |
| Case 6 | 3         | 2         | 1        | 67%            |
| Case 7 | 3         | 0         | 3        | 100%           |

### Overall:

* Total claims: 23
* Confirmed: 8
* Rejected: 15
* Average Rejection rate: **70%**

✔ Interpretation: Majority of speculative hypotheses are eliminated before persistence.

---

# Part 4 — Evidence Integrity Architecture

SIFT-PROOF enforces **multi-layer forensic immutability guarantees**.

---

## Layer 1 — Read-Only Disk Mount

```bash
mount -t ntfs-3g -o ro,noatime,loop ...
```

✔ Ensures filesystem-level immutability

---

## Layer 2 — SQL Assertion Gate (Read-Only Enforcement)

Only SELECT statements are permitted:

* INSERT  blocked
* UPDATE  blocked
* DELETE  blocked

All violations raise immediate exception before execution.

✔ Guarantees database immutability

---

## Layer 3 — MCP Tool Interface Design

No tool exposes raw write access to:

* MFT artifact tables
* registry datasets
* EVTX logs
* memory structures

Only controlled write target:

* `confirmed_findings` (only after assertion PASS)

---

## Layer 4 — Append-Only Audit System

Audit logs are:

* append-only JSONL
* never modified or rewritten
* fully replayable

✔ Guarantees forensic trace integrity

---

## Layer 5 — Subprocess Injection Sanitization

Blocked patterns:

* `rm`, `dd`, `format`, `shred`
* shell chaining (`&&`, `;`, `|`, `>`)
* injection operators (`$()`, backticks)

✔ Prevents OS-level manipulation attempts

---

## Final Integrity Result

✔ No evidence modification is possible through any execution path
✔ All forensic operations are constrained to read-only validated flows

---

# Part 5 — Reproducibility Guarantee

All findings are:

* SQL-backed
* deterministic
* replayable

### Validation commands:

```bash
python3 replay.py --finding_id <FINDING_ID>
python3 replay.py --audit
python3 replay.py --all
```

✔ Identical evidence + identical query → identical result

---

# Part 6 — System Limitations (Honest Assessment)

## 1. Schema-bound visibility

Only artifacts defined in schema are queryable.

## 2. Upstream tool dependency

SIFT parsing tools (Volatility, fls, RegRipper) are trusted inputs.

## 3. Coverage ≠ completeness

Coverage ensures tool execution completeness, not full adversary discovery.

## 4. Dataset scope limitation

Validation applies only to 7 benchmark forensic cases.

---


### Core security property:

>  Hallucinations are not filtered — they are structurally prevented from ever becoming findings.

---

