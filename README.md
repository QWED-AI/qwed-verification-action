<div align="center">

  <img src="logo.png" alt="QWED Logo - AI Verification Engine" width="80" height="80">
  <h1>QWED Verification</h1>
  <h3>GitHub Action — a deterministic witness for your CI pipeline</h3>

  <p><i>Every pull request makes a claim. This cross-examines it.</i></p>

  [![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Available-1a1a1a?style=flat)](https://github.com/marketplace/actions/qwed-protocol-verification)
  [![Verified Publisher](https://img.shields.io/badge/Verified_Publisher-QWED-2ea44f?style=flat&logo=github&logoColor=white)](https://github.com/marketplace/actions/qwed-protocol-verification)
  [![Core Protocol](https://img.shields.io/badge/Core-QWED_Protocol-1a1a1a?style=flat)](https://github.com/QWED-AI/qwed-verification)
  [![License](https://img.shields.io/badge/License-Apache_2.0-1a1a1a?style=flat)](LICENSE)
  [![Verification Context v1.0](https://img.shields.io/badge/Verification_Context-v1.0-2ea44f?style=flat)](https://github.com/QWED-AI/qwed-verification/blob/main/spec/v1.0/verification-context.md)

  <br>

  [Quick Start](#quick-start) ·
  [What It Verifies](#what-it-verifies) ·
  [Modes](#verification-modes) ·
  [Inputs & Outputs](#inputs--outputs) ·
  [Verification Context v1.0](#verification-context-v10) ·
  [Why a Deterministic Judge](#why-a-deterministic-judge) ·
  [Security](#security--privacy)

</div>

---

## Quick Start

One file. One step. No account required to start.

```yaml
# .github/workflows/qwed.yml
name: QWED Verification
on: [pull_request]

jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: QWED-AI/qwed-verification-action@v1
        with:
          action: scan-secrets
          paths: "**/*.env,**/*.json"
          output_format: sarif
          fail_on_findings: "true"
```

From this commit forward, nothing merges to `main` on the strength of its own testimony.

---

## The Case Against Taking Its Word For It

Ask a model to calculate compound interest on $100,000 at 5% over ten years, and a fluent, confident, wrong answer comes back — simple interest, dressed as compound. Nothing in the tone gives it away. That's the actual failure mode: **fluency and correctness are different claims**, and only one of them is checkable.

In our benchmarks, a frontier model held **73% accuracy** on financial calculations — number-moving tasks, run through a solver instead of trusted on delivery. QWED caught the remaining errors before they reached anywhere that mattered. On unverified financial output, that gap has priced out as high as **$12,889 per transaction**.

QWED doesn't try to make the model smarter. It makes the model **accountable** — every claim gets handed to something that can't guess.

---

## What It Verifies

One action, four jurisdictions:

| Mode | Catches | Why It's on Trial |
|---|---|---|
| `scan-secrets` | Leaked API keys, tokens, SSH keys | Your secrets, in someone else's repo, before you've noticed |
| `scan-code` | `eval()`, `exec()`, `subprocess`, unsafe imports | The RCE that was one merge away |
| `verify-shell` | `curl \| bash`, `rm -rf`, sudo escapes | The script that owns the box it runs on |
| `verify` | Hallucinated math, logic, SQL, code | Output that reads correctly and isn't |

A linter tells you your code doesn't match the style guide. QWED tells you your code doesn't match reality — using **SymPy**, **Z3**, and **SQLGlot**, the same class of engine used to prove theorems, not to guess at them.

---

## Verification Modes

**Secret scanning**
```yaml
- uses: QWED-AI/qwed-verification-action@v1
  with:
    action: scan-secrets
    paths: "**/*.env,**/*.json,**/*.py"
    fail_on_findings: "true"
```

**Code security**
```yaml
- uses: QWED-AI/qwed-verification-action@v1
  with:
    action: scan-code
    paths: "**/*.py"
    output_format: sarif   # surfaces directly in the GitHub Security tab
```

**Shell verification**
```yaml
- uses: QWED-AI/qwed-verification-action@v1
  with:
    action: verify-shell
    paths: "**/*.sh"
```

**LLM output verification** (requires a QWED backend — see note below)
```yaml
- uses: QWED-AI/qwed-verification-action@v1
  with:
    action: verify
    engine: math
    query: "Integral of x^2"
    llm_output: "x^3/3"
    api_key: ${{ secrets.QWED_API_KEY }}   # optional in local mode
```
```
Result: REJECTED — the integral of x² is x³/3 + C, proven by SymPy, cited in the ruling.
```

**Verification Context output** (emit VC v1.0 JSON)
```yaml
- uses: QWED-AI/qwed-verification-action@v1
  with:
    action: scan-code
    paths: "**/*.py"
    output_format: verification-context
    fail_on_findings: "true"
  # outputs: verdict, admission, proof_ref, verification_context
```

> **Note on `verify` mode:** secret scanning, code scanning, and shell verification run **entirely inside the runner** — no backend needed. The `verify` mode (LLM output cross-examination) calls the QWED verification API; pass an `api_key` (or run a QWED backend locally) for that mode. Self-hosted deployments can use the `api_url` input to point to their own backend.

---

## Verification Context v1.0

Every QWED verification result is emitted as a [Verification Context v1.0](https://github.com/QWED-AI/qwed-verification/blob/main/spec/v1.0/verification-context.md) document — a machine-readable, schema-validated protocol with:

| Output | Values | Meaning |
|---|---|---|
| `verdict` | `VERIFIED` · `UNVERIFIABLE` · `BLOCKED` | The truth judgment |
| `admission` | `ADMIT` · `DENY` | Safe to merge? (truth ≠ admission) |
| `proof_ref` | `sha256:<64-hex>` or empty | Cryptographic evidence commitment |
| `verification_context` | JSON | Full VC v1.0 document (with `output_format: verification-context` or `json`) |
| `verified` | `true` · `false` | Backward-compatible boolean (`true` only when `verdict=VERIFIED` and `admission=ADMIT`) |

**Fail-closed guarantees:**
- `UNVERIFIABLE` and `BLOCKED` always produce `admission: DENY`
- `VERIFIED` requires a resolvable `proof_ref`
- Schema validation failure fails closed
- `fail_on_findings: "true"` gates on `admission == "ADMIT"`, not just a boolean

---

## Inputs & Outputs

**Inputs**

| Input | Default | Description |
|---|---|---|
| `action` | `verify` | `verify` · `scan-secrets` · `scan-code` · `verify-shell` · `verify-process` |
| `engine` | `math` | `math` · `logic` · `code` · `sql` · `shell` (for `verify`) |
| `query` | — | The original user query, e.g. *"Derivative of x²"* |
| `llm_output` | — | The output being cross-examined |
| `paths` | `.` | Glob patterns to scan, e.g. `**/*.py,**/*.env` |
| `output_format` | `text` | `text` · `json` · `sarif` · `verification-context` |
| `fail_on_findings` | `true` | Fail the build on any finding (admission != ADMIT) |
| `api_key` | — | Optional — local mode requires nothing |
| `api_url` | `https://api.qwedai.com` | QWED API base URL for self-hosted deployments |
| `mask_pii` | `false` | Redact PII in inputs and outputs |

**Outputs**

| Output | Description |
|---|---|
| `verdict` | `VERIFIED` · `UNVERIFIABLE` · `BLOCKED` (Verification Context v1.0) |
| `admission` | `ADMIT` · `DENY` — gate execution/shipping on `admission == "ADMIT"` |
| `proof_ref` | `sha256:<64-hex>` evidence commitment, or empty when not verified |
| `verification_context` | Full VC v1.0 JSON document (when `output_format: verification-context` or `json`) |
| `verified` | `true` if `verdict=VERIFIED` and `admission=ADMIT` (backward-compatible) |
| `explanation` | The proof, or the reason it didn't hold |
| `findings_count` | Number of issues found |
| `sarif_file` | Path to the SARIF report |
| `badge_url` | URL for your QWED verified badge |

---

## Why a Deterministic Judge

| | QWED | Most guardrails |
|---|---|---|
| The judge | A deterministic solver (Z3 / SymPy) | Another model, or an embedding distance |
| Verdict basis | Mathematical proof | Resemblance to a "good" answer |
| Result | `VERIFIED` with `verdict`, `admission`, and `proof_ref` | "Looks fine" |
| Latency | Under 100ms for most checks | Variable |
| Data handling | Never leaves the runner (except `verify` mode) | Usually a round trip to the cloud |

QWED isn't in competition with the models it checks. It's what lets you ship them.

---

## Security & Privacy

- **Scan modes stay on the runner.** `scan-secrets`, `scan-code`, `verify-shell`, and `verify-process` execute entirely inside your CI environment or VPC — no external call, no exception.
- **Verify mode calls a backend.** `verify` (LLM output cross-examination) calls the configured QWED API unless you point `api_url` at a self-hosted or local deployment — so only the query and output under examination traverse that boundary, and only to the backend you chose.
- **Nothing is learned from.** QWED is a deterministic execution engine, not a model. There is no training loop for your data to enter.
- **Every passing result includes an evidence commitment.** A passing result ships with `verdict=VERIFIED`, `admission=ADMIT`, and a `proof_ref` binding the ruling to the evidence that produced it.
- **SARIF native.** Findings land directly in the GitHub Security tab — no separate dashboard to check.

---

## Versioning

```yaml
- uses: QWED-AI/qwed-verification-action@v1      # tracks latest v1
- uses: QWED-AI/qwed-verification-action@v1.0.0  # pinned, reproducible
```

The action's version tags are decoupled from the engine's release train — action fixes ship on their own cadence. `v1` is a floating alias: the release workflow moves it to every new `v1.x.y`, so it always resolves to the latest action release (and whatever engine that release pins). For a fixed build, pin the full version.

| Action | Engine image |
|---|---|
| `v1` (alias) | Tracks latest `v1.x.y` — see rows below |
| `v1.0.0` | `:latest` (unpinned — resolves to the current latest engine) |

`main` already carries a digest pin (`sha256:be5d26f1…`, engine `3.2.0`); it ships as `v1.0.1`, which will get its own row here.

### Engine bump policy

- When the engine releases a new version, the action is consciously bumped to reference it (new action minor version, pin updated, matrix row added below).
- Patch action releases (`1.x.y`) never change the engine pin.
- Minor engine releases that change verification behavior require an action minor bump (`1.x`).
- The matrix above lists published tags only and is updated in the same PR as every pin change — a pin without a matrix row fails review.
- Verify any row: the full digest is in `action.yml` under `runs.image`; Docker Hub tags map it to the engine release.

---

## The Wider Protocol

This action runs on the open-source **[QWED Protocol](https://github.com/QWED-AI/qwed-verification)** — eleven-plus verification engines, agent-security guards, and SDKs for Python, TypeScript, Go, and Rust.

| Resource | Link |
|---|---|
| Core repository | [QWED-AI/qwed-verification](https://github.com/QWED-AI/qwed-verification) |
| Verification Context spec | [spec/v1.0/verification-context.md](https://github.com/QWED-AI/qwed-verification/blob/main/spec/v1.0/verification-context.md) |
| QWED Security (GitHub App) | [QWED-AI/qwed-security](https://github.com/QWED-AI/qwed-security) |
| Documentation | [docs.qwedai.com](https://docs.qwedai.com) |
| Verification course | [QWED-AI/qwed-learning](https://github.com/QWED-AI/qwed-learning) |
| Sponsor | [github.com/sponsors/QWED-AI](https://github.com/sponsors/QWED-AI) |

---

## Contributing

Found a bypass, or have a new engine in mind? Read [CONTRIBUTING.md](CONTRIBUTING.md) and [SECURITY.md](SECURITY.md) first — most good PRs start as a failed attempt to fool the verifier.

---

<div align="center">

<i>Where an argument ends, and a proof begins.</i>

[GitHub](https://github.com/QWED-AI/qwed-verification) ·
[Documentation](https://docs.qwedai.com) ·
[Marketplace](https://github.com/marketplace/actions/qwed-protocol-verification)

</div>
