```markdown
# Final Agent Checklist — Apply All Remaining Recommended Updates

> This checklist contains every concrete change needed to bring the TODO.md to its final state, based on all research and analysis rounds.  
> Changes are **cumulative** and should be applied directly to the most recent version of the file.  
> Where a change depends on a user decision (e.g., the scoring weight remap), it is clearly flagged.

---

## 1. BROK‑001‑2 – Update CA DROP timeline to verified 2026 regulations

**Location**: Subtask `BROK‑001‑2` → `CA DROP fields` block.

**Action**: Replace the entire `CA DROP fields` paragraph with:

```
CA DROP fields: activationDate (August 1, 2026), pollInterval (every 45 days), 
determinationDeadline (date of download + 90 days), compensationDailyFine ($200), 
status (pending/complete/suppressed).  
Note: All identifiers go on a permanent suppression list regardless of match status.
```

**Rationale**: The California DROP platform became operational on 1 Jan 2026; deletion processing begins 1 Aug 2026; brokers have 90 days to complete determination, not 45. Permanent suppression lists are mandatory.

---

## 2. COMP‑001‑5 – Realign scoring weights to published Privacy Fairness Index (PFI) methodology *(decision required)*

**Location**: Subtask `COMP‑001‑5` → `Real-world weights` comment.

**Current weights**: Encryption/data protection: 0.30, Transparency: 0.30, User control: 0.25, Legal compliance history: 0.15.

**Option A (recommended)** – Remap to the six‑category PFI methodology used by terms.law (2026):

```
Real-world weights (mapped from the Privacy Fairness Index, terms.law 2026):
- Data Collection & Sharing: 0.40  (merges "Data Collection" 30% + "Third‑Party Sharing" 20% → 50% of base; scaled to 40%)
- Transparency & User Control: 0.35 (merges "User Control & Consent" 15% + "Transparency & Access" 10%)
- Retention & Security: 0.15 (merges "Retention & Deletion" 20% + "Security" 5%)
- Legal Compliance History: 0.10 (external regulatory record — not directly measured by policy‑only PFI)
Standard PFI categories: data-collection, third-party-sharing, retention-deletion, 
user-control-consent, security-breach-notification, transparency-access.
```

**Option B** – Keep the current weights and add a note that they will be aligned to PFI in v2 after further validation.

**Agent instruction**: After user confirmation, apply the chosen option and also update the **parent task `COMP‑001`** Rules table to match the weight values.

---

## 3. ARCH‑001‑5 – Document `neverthrow` serialization limitation and `verdict‑ts` alternative

**Location**: Subtask `ARCH‑001‑5` → after the existing verification line.

**Action**: Append a new note:

```
Note: neverthrow is class‑based; Result objects lose their prototype chain when 
passed through structuredClone, postMessage, or Worker boundaries (relevant for 
Cloudflare Workers SSR). For the v1 static site this is not an issue. If v2 adds 
SSR API routes, consider verdict‑ts (491 B gzipped, plain‑object Result) or ensure 
results are unwrapped before serialization.
```

**Rationale**: Prevents a known runtime hazard in future SSR work.

---

## 4. LAW‑001 – Add structured legal compliance framework to parent task Rules

**Location**: Parent task `LAW‑001` → `Rules` block (after the existing line).  

**Action**: Append the following:

```
Legal compliance scoring follows a structured framework adapted from:
- EU AI Act 5‑dimension model (explainability, fairness, privacy, robustness, social well‑being)
- PET evaluation criteria (New America, 2026): absence of re‑identification, anonymity protections, 
  linkage‑attack prevention, scalability, and implementation complexity
- DROP broker compliance requirements (California 2026): 45‑day polling, 90‑day determination, 
  permanent suppression list
```

**Rationale**: Gives the legal domain an objective, research‑grounded basis for scoring law compliance, making `INT‑002` testable and consistent.

---

## 5. INT‑001‑2 & VIOL‑001‑4 – Synchronise event payload shapes

**a) INT‑001‑2**

**Location**: Subtask `INT‑001‑2` → description.

**Action**: Change `reads companyName from CompanyEvaluated payload` to:

```
reads { companyId, companyName } from CompanyEvaluated payload 
(same shape as COMP‑001‑4)
```

**b) VIOL‑001‑4**

**Location**: Subtask `VIOL‑001‑4` → comment.

**Action**: Append to the description:

```
CompanyReference value object (companyId: string, companyName: string — 
populated by CompanyEvaluated event handler; see INT‑001)
```

**Rationale**: Ensures the handler’s interface exactly matches the event emitted by `COMP‑001‑4`, preventing integration defects.

---

## 6. ARCH‑001‑2 – Provide concrete CSP configuration using Astro 6’s stable API

**Location**: Subtask `ARCH‑001‑2` → the `Security` line.

**Action**: Replace the current security note with:

```
Security: Configure Content Security Policy using Astro 6's stable CSP API 
(security.csp in astro.config.mjs). Default policy: default-src 'self', 
script-src 'self', style-src 'self', frame‑ancestors 'none'. 
No third‑party tracking scripts permitted.
```

**Rationale**: The CSP API graduated from experimental in Astro 6 and should be configured with production‑ready defaults.

---

## 7. COMP‑002‑3 – Document KV secondary index pattern and rate‑limit caveat

**Location**: Subtask `COMP‑002‑3` → existing `Note`.

**Action**: Replace the note with:

```
Note: KV supports prefix‑based listing for secondary indexes 
(key pattern: category:{value}:{companyId}). For v1 static build, write rate 
limits (1/sec/key) are not a concern. For v2 with SSR, consider D1 or Durable 
Objects for write‑heavy workloads.
```

**Rationale**: The KV secondary index pattern is viable, but its rate limits must be documented for future scaling decisions.

---

## 8. Phase 4 preamble – Add hydration strategy for React islands

**Location**: Phase 4 section header, before `### WEB‑001`.

**Action**: Add a new paragraph:

```
Hydration strategy: Static pages use Astro .astro files (zero JS shipped). 
Interactive components (CompanyListing, BrokerDashboard, etc.) use React islands 
with targeted hydration directives: client:visible for filter panels and listings, 
client:idle for search bars and auxiliary widgets.
```

**Rationale**: Aligns the web layer with the zero‑JS‑by‑default philosophy of the privacy‑first site and Astro’s islands architecture.

---

## 9. DATA‑001‑4 – Include complete 2026 US state law abbreviation reference

**Location**: Subtask `DATA‑001‑4` → after the existing verification.

**Action**: Append a comment with the full law list:

```
Complete 2026 US state law abbreviations:
CCPA/CPRA, VCDPA, CPA, CTDPA, UCPA, ICDPA, INCDPA, TIPA, TDPSA, FDBR, MODPA, 
MCDPA (MN), MCDPA (MT), OCPA, DPDPA, NHDPA, NJDPA, KYCDPA, NDPA, RHDPA.
Oklahoma (ODPA) effective Jan 1, 2027 (not included in v1 seed).
```

**Rationale**: Provides a ready reference for populating the seed data with correct law names and abbreviations.

---

## 10. ARCH‑001‑2 – Clarify default output mode as `static`

**Location**: Subtask `ARCH‑001‑2` → existing `verification` line.

**Action**: Add a note:

```
Note: for v1, output: 'static' is recommended. Hybrid mode (output: 'hybrid') 
is available for v2 when SSR pages with KV/D1 bindings are needed.
```

**Rationale**: Ensures the build is configured correctly for the all‑static MVP.

---

## Verification

After applying all the above changes, run the following sanity checks:

- `BROK‑001‑2` deadline mentions 90 days and permanent suppression.
- `COMP‑001‑5` weights are consistent with either Option A or B and match parent table.
- `ARCH‑001‑5` includes a serialization caveat.
- `LAW‑001` Rules now lists the three legal frameworks.
- `INT‑001‑2` and `VIOL‑001‑4` event payload descriptions are in sync.
- Phase 4 preamble mentions hydration strategy.
- `DATA‑001‑4` includes the full list of 20 state law abbreviations.

The TODO.md will then be fully aligned with the latest research and ready for agent execution.
```