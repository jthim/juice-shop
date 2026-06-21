# 🚀 Automated AppSec CI/CD Pipeline & Vulnerability Remediation

## 📌 Project Overview

This project establishes a continuous application security (AppSec) testing workflow using a localized sandbox. By targeting a notoriously vulnerable application (**OWASP Juice Shop**), I engineered a customized static analysis (SAST) scanning lifecycle within GitHub Actions and implemented automated software composition analysis (SCA) to secure the codebase from build to mainline merge.

> **Disclaimer:** This is a personal fork of OWASP Juice Shop for testing and documentation purposes. It is not an official OWASP release or distribution.

## 🏗️ Pipeline Architecture & Scan Orchestration

Instead of running a single automated policy, I designed a dual-stage conditional traffic routing pattern inside **GitHub Actions** to optimize developer velocity and scanning depth.

```text
                        ┌────────────────────────┐
                        │   Developer Git Push   │
                        └───────────┬────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
         [ Pull Request Event ]            [ Merge to Master Event ]
                    │                               │
       (Custom Local Rules Only)              (Full Comprehensive Scan)
                    │                               │
                    ▼                               ▼
       Fails if severity = error       Syncs with Cloud Dashboard
       (Fast local verification)           (Full multi-file inventory)

```

### 1. Shift-Left Pull Request Gating (Fast Feedback)

- **Strategy:** Runs optimized custom rules offline on incoming PRs to maximize validation speed and developer feedback loops.
- **Custom Rule Engineering:** Rather than relying solely on out-of-the-box defaults, I leveraged this stage to author and test custom Semgrep rule definitions (semgrep.yml). This allowed me to practice syntax targeting and define precise code-pattern matching.
- **Build Hardening:** Overrode Semgrep’s default behavior (which exits with code `0` and passes broken builds). Configured the engine to explicitly return **`exit code 1`** upon detecting errors, successfully **breaking the CI pipeline** and blocking insecure code from merging.

### 2. Mainline Merge Synchronization (Deep Analysis)

- **Strategy:** Triggers a full, comprehensive codebase audit via `semgrep ci` upon a successful merge to the `master` branch, synchronizing metrics to a centralized dashboard.

---

## 🛠️ Security Engineering & Noise Reduction

### 1. Signal-to-Noise Ratio Optimization (`.semgrepignore`)

- **Problem:** OWASP Juice Shop contains internal interactive tutorial modules that purposefully track challenge progress. These modules generated extensive false-positives, blinding the scan results to actual codebase vulnerabilities.
- **Solution:** Engineered a targeted `.semgrepignore` configuration file to drop noise vectors and isolate scanning scope exclusively to functional production code layers.

### 2. Software Supply Chain Security (SCA)

- Integrated **GitHub Dependabot** to continuously scan manifest dependencies.
- Analyzed out-of-date and highly vulnerable upstream library dependencies, leading to an automated, isolated Pull Request patch verification cycle that successfully updated vulnerable code packages on `master`.

---

## 📊 Vulnerability Lifecycle & Remediation Log

To validate the configuration, I containerized the workspace via **Docker** to test patches locally against live endpoints.

| Vulnerability                  | Detection Source    | Context / Finding                                                                                     | Engineering Fix / Remediation                                                                                       | Verification Method                                                                      |
| ------------------------------ | ------------------- | ----------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **SQL Injection (SQLi)**       | Local SAST Scan     | User inputs directly concatenated into active database query execution strings.                       | Rewrote backend controllers to enforce **parameterized queries**, decoupling user strings from structural commands. | Verified resolution via automated clean pipeline runs & manual application verification. |
| **Cross-Site Scripting (XSS)** | Local SAST Scan     | Malicious raw text scripts directly parsed and rendered directly into browser DOM views.              | Updated rendering pathways to use **context-aware sanitization / output encoding** mechanisms.                      | Verified resolution via automated clean pipeline runs & manual application verification. |
| **Vulnerable Dependency**      | Dependabot Analysis | Outdated third-party packages in `package.json` exposing system infrastructure to active public CVEs. | Orchestrated automated patch logic verification via dependency update pull requests.                                | Clean automated dashboard metric confirmation.                                           |
