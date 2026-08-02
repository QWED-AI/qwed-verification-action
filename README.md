<div align="center">

  <h1>QWED Verification</h1>
  <h3>GitHub Action — a deterministic witness for your CI pipeline</h3>

  <p><i>Every pull request makes a claim. This cross-examines it.</i></p>

  [![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Available-1a1a1a?style=flat)](https://github.com/marketplace/actions/qwed-protocol-verification)
  [![Verified Publisher](https://img.shields.io/badge/Verified_Publisher-QWED-2ea44f?style=flat&logo=github&logoColor=white)](https://github.com/marketplace/actions/qwed-protocol-verification)
  [![Core Protocol](https://img.shields.io/badge/Core-QWED_Protocol-1a1a1a?style=flat)](https://github.com/QWED-AI/qwed-verification)
  [![License](https://img.shields.io/badge/License-Apache_2.0-1a1a1a?style=flat)](LICENSE)

  <br>

  [Quick Start](#quick-start) ·
  [What It Verifies](#what-it-verifies) ·
  [Modes](#verification-modes) ·
  [Inputs & Outputs](#inputs--outputs) ·
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

**LLM output verification**
```yaml
- uses: QWED-AI/qwed-verification-action@v1
  with:
    action: verify
    engine: math
    query: "Integral of x^2"
    llm_output: "x^3/3"
```
```
Result: REJECTED — the integral of x² is x³/3 + C, proven by SymPy, cited in the ruling.
```

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
| `output_format` | `text` | `text` · `json` · `sarif` |
| `fail_on_findings` | `true` | Fail the build on any finding |
| `api_key` | — | Optional — local mode requires nothing |
| `mask_pii` | `false` | Redact PII in inputs and outputs |

**Outputs**

| Output | Description |
|---|---|
| `verified` | `true` if the claim held |
| `explanation` | The proof, or the reason it didn't |
| `findings_count` | Number of issues found |
| `sarif_file` | Path to the SARIF report |
| `badge_url` | URL for your QWED verified badge |

---

## Why a Deterministic Judge

| | QWED | Most guardrails |
|---|---|---|
| The judge | A deterministic solver (Z3 / SymPy) | Another model, or an embedding distance |
| Verdict basis | Mathematical proof | Resemblance to a "good" answer |
| Result | `VERIFIED` with a cryptographic `proof_ref` | "Looks fine" |
| Latency | Under 100ms for most checks | Variable |
| Data handling | Never leaves the runner | Usually a round trip to the cloud |

QWED isn't in competition with the models it checks. It's what lets you ship them.

---

## Security & Privacy

- **Nothing leaves the runner.** Code and secrets are evaluated inside your CI environment or VPC — no external call, no exception.
- **Nothing is learned from.** QWED is a deterministic execution engine, not a model. There is no training loop for your data to enter.
- **Every verdict is signed.** A passing result ships with a JWT attestation and a SHA-256 `proof_ref` binding the ruling to the evidence that produced it.
- **SARIF native.** Findings land directly in the GitHub Security tab — no separate dashboard to check.

---

## Versioning

```yaml
- uses: QWED-AI/qwed-verification-action@v1      # tracks latest v1
- uses: QWED-AI/qwed-verification-action@v1.2.0  # pinned, reproducible
```

The action's version tags are decoupled from the core protocol's release train — action fixes ship on their own cadence, and the underlying engine always runs from the latest published QWED image.

---

## The Wider Protocol

This action runs on the open-source **[QWED Protocol](https://github.com/QWED-AI/qwed-verification)** — eleven-plus verification engines, agent-security guards, and SDKs for Python, TypeScript, Go, and Rust.

| Resource | Link |
|---|---|
| Core repository | [QWED-AI/qwed-verification](https://github.com/QWED-AI/qwed-verification) |
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
