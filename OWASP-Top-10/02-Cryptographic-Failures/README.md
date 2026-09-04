# OWASP A02:2021 — Cryptographic Failures

![OWASP Top 10](https://img.shields.io/badge/OWASP%20Top%2010-A02%3A2021-red)
![Target](https://img.shields.io/badge/target-Juice%20Shop-orange)
![Status](https://img.shields.io/badge/status-complete-brightgreen)
![Severity](https://img.shields.io/badge/max%20severity-Critical-critical)

**Target:** [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/) — local Docker instance (`127.0.0.1:3000`)
**Category:** [A02:2021 – Cryptographic Failures](https://owasp.org/Top10/A02_2021-Cryptographic_Failures/)
**Lab series:** OWASP Top 10 vs. Juice Shop — Lab 2 of 10 ([← Lab 1: A01 Broken Access Control](../01-Broken-Access-Control/README.md))

---

## Table of Contents
- [Overview](#overview)
- [Attack Chain](#attack-chain)
- [Summary of Findings](#summary-of-findings)
- [Evidence](#evidence)
- [Methodology](#methodology)
- [Remediation](#remediation)
- [Key Takeaway](#key-takeaway)
- [Reproducing This Lab](#reproducing-this-lab)
- [Skills Demonstrated](#skills-demonstrated)

---

## Overview

This lab targets how Juice Shop protects authentication secrets and sensitive data at rest and in transit. Rather than three unrelated bugs, the findings here form a single attack chain: a predictable credential gets you a valid session, that session's token reveals more than it should, and what it reveals exposes a fundamentally broken password-storage scheme underneath. Each step made the next one possible.

## Attack Chain

```
Default admin credentials  →  Valid JWT issued  →  JWT payload decoded
      (Finding 1)                                    (Finding 2)
                                                            │
                                                            ▼
                                          Password hash extracted from payload
                                                            │
                                                            ▼
                                        Confirmed unsalted MD5 (Finding 3)
```

## Summary of Findings

| # | Title | Location | Severity | Result |
|---|-------|----------|----------|--------|
| 1 | Default admin credentials accepted | `POST /rest/user/login` | 🔴 Critical | **Vulnerable** |
| 2 | Sensitive data exposed in JWT payload | Auth token (client-decodable) | 🟠 High | **Vulnerable** |
| 3 | Unsalted MD5 password hashing | Password storage | 🔴 Critical | **Vulnerable** |

Full write-up with reproduction steps, evidence, and impact analysis: **[`findings.md`](./findings.md)**

## Evidence

| # | Description | Screenshot |
|---|--------------|------------|
| 1 | Login succeeds with default admin credentials | [`01-admin-default-creds-login.png`](./screenshots/01-admin-default-creds-login.png) |
| 2 | Decoded JWT reveals `role: admin` and raw password hash | [`02-admin-jwt-role-decode.png`](./screenshots/02-admin-jwt-role-decode.png) |
| 3 | Locally-computed MD5 matches stored hash exactly (unsalted) | [`03-md5-unsalted-proof.png`](./screenshots/03-md5-unsalted-proof.png) |

![Admin default creds login](./screenshots/01-admin-default-creds-login.png)
![Admin JWT role decode](./screenshots/02-admin-jwt-role-decode.png)
![MD5 unsalted proof](./screenshots/03-md5-unsalted-proof.png)

Raw request/response data for every command executed is saved in **[`raw-output/`](./raw-output/)**.

## Methodology

Testing followed the attack chain above rather than testing findings in isolation — each step's output became the next step's input, mirroring how a real attacker would pivot. Full reasoning and tooling: **[`methodology.md`](./methodology.md)**

## Remediation

Concrete, prioritized fixes for each finding — credential policy hardening, minimal JWT payload design, and migration to a modern salted hashing algorithm (bcrypt/argon2id) — are documented in **[`remediation.md`](./remediation.md)**

## Key Takeaway

None of these three issues exist in isolation in a real breach. A leaked or guessed credential is only as damaging as what it unlocks — here, it unlocked a token that leaked internals, which in turn exposed a hashing scheme that would let an attacker offline-crack every password in the database in seconds. Defense in depth matters precisely because failures compound: fixing any one link (stronger credentials, a leaner token, or salted hashing alone) would have broken this specific chain, but only fixing all three closes the underlying pattern.

## Reproducing This Lab

Requires a local Juice Shop instance running on `localhost:3000`. Every command run during testing, in exact order, is logged in **[`commands.md`](./commands.md)** — paste them sequentially to reproduce each finding.

## Skills Demonstrated

- API security testing with `curl`
- JWT structure analysis and manual payload decoding
- Cryptographic weakness identification (hash algorithm fingerprinting, salt verification)
- Root-cause / attack-chain analysis rather than isolated bug reporting
- Security documentation: findings, methodology, and remediation written for a technical + non-technical audience

---

*Part of an ongoing OWASP Top 10 lab series against OWASP Juice Shop.*
