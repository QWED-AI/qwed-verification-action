# Security Policy

## Reporting a Vulnerability

The QWED team takes security seriously. This action is a security product — a bypass is a product bug.

**Do not open a public issue for a security vulnerability.** Report privately to **rahul@qwedai.com** or via GitHub's private vulnerability reporting:

https://github.com/QWED-AI/qwed-verification-action/security/advisories

Please include:

- The mode affected (`scan-secrets`, `scan-code`, `verify-shell`, `verify`)
- A minimal reproducer (a file or command that evades detection)
- Expected vs. actual behavior

## Response

- **Acknowledgement:** within 48 hours
- **Triage:** within 5 business days
- **Fix + disclosure:** coordinated, typically 90 days for critical findings

## Scope

This action is a thin wrapper over the QWED core image (`docker://qwedai/qwed-verification:latest`). Vulnerabilities in the underlying engines and guards belong to the core repository:

https://github.com/QWED-AI/qwed-verification/security

## Preferred Reporting for Findings

- Logic/logic bypass, injection, or RCE in scanned content — core repo
- Input/env handling, output leakage, or SARIF malformation in this wrapper — this repo
