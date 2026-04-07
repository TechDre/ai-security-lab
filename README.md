# AI Security Lab

A hands-on AI security engineering learning lab covering prompt injection detection, LLM guardrails, network traffic analysis, supply chain security, excessive agency, and vector/embedding attacks — mapped to the OWASP Top 10 for LLM Applications.

**Platform:** Kali Linux (WSL on Windows 10) | **Language:** Python 3.13

---

## Table of Contents

- [Overview](#overview)
- [Labs](#labs)
- [OWASP Coverage](#owasp-coverage)
- [Prerequisites](#prerequisites)
- [Repository Structure](#repository-structure)

---

## Overview

This lab series builds practical AI security engineering skills by demonstrating real attack techniques and their defenses. Each lab targets a specific OWASP LLM risk, includes working code, and documents findings with security observations and defensive recommendations.

---

## Labs

| Lab | Topic | OWASP Reference | Folder |
|-----|-------|-----------------|--------|
| Lab 1 | Prompt Injection Detection | LLM01 | [GUIDE.md](./GUIDE.md) |
| Lab 2 | LLM Guardrails (Input/Output Validation) | LLM01, LLM06 | [llm-guardrails-lab/](./llm-guardrails-lab/) |
| Lab 3 | Network Traffic Analysis with TShark | LLM06 | [lab3-network-analysis/](./lab3-network-analysis/) |
| Lab 4 | Supply Chain Security with pip-audit | LLM03 | [lab4-supply-chain/](./lab4-supply-chain/) |
| Lab 5 | Excessive Agency | LLM06 | [lab5-excessive-agency/](./lab5-excessive-agency/) |
| Lab 6 | Vector/Embedding Security (RAG Poisoning) | LLM08 | [lab6-vector-security/](./lab6-vector-security/) |

### Lab 1 — Prompt Injection Detection (OWASP LLM01)

Multi-layered prompt injection detector using regex pattern matching, heuristic scoring, and a DeBERTa AI classifier. Detects system prompt overrides, role-play escapes, jailbreaks, and data exfiltration attempts with 99%+ accuracy.

### Lab 2 — LLM Guardrails (OWASP LLM01, LLM06)

A 5-layer input/output validation pipeline sitting between the user and an LLM. Guards against prompt injection, harmful content, and PII exposure — with output filtering to catch system prompt leakage in model responses.

### Lab 3 — Network Traffic Analysis (OWASP LLM06)

Captures and analyzes live network traffic with TShark to identify protocols, establish baselines, and detect security anomalies. Demonstrates how LLM agent network activity (C2 beaconing, exfiltration) would appear in DNS and TCP traffic.

### Lab 4 — Supply Chain Security (OWASP LLM03)

Audits a Python project with vulnerable dependencies using pip-audit, identifies 27 CVEs across 6 packages (including transitive dependencies), and patches to a clean state. Demonstrates the hidden risk of unpinned and unaudited packages in AI application stacks.

### Lab 5 — Excessive Agency (OWASP LLM06)

Builds an unsafe agent vulnerable to shell injection, then a safe agent with command allowlisting, input validation, and audit logging. Demonstrates how least-privilege principles and layered defenses block prompt injection attacks that reach the agent layer.

### Lab 6 — Vector/Embedding Security (OWASP LLM08)

Builds a simple RAG system, executes a keyword-stuffing vector poisoning attack that hijacks query results, and implements trusted source filtering as a defense. Shows how RAG knowledge bases are an attack surface when write access is not controlled.

---

## OWASP Coverage

| OWASP LLM Risk | Labs |
|----------------|------|
| LLM01: Prompt Injection | Lab 1, Lab 2 |
| LLM03: Supply Chain | Lab 4 |
| LLM06: Sensitive Information Disclosure / Excessive Agency | Lab 2, Lab 3, Lab 5 |
| LLM08: Vector and Embedding Weaknesses | Lab 6 |

---

## Prerequisites

- Python 3.10+
- Kali Linux (or any Linux environment; WSL on Windows works)
- Internet connection (Lab 1 only — downloads DeBERTa model ~700MB on first run)
- Labs 1-2: `pip install transformers torch sentencepiece protobuf`
- Lab 3: `sudo apt install -y wireshark`
- Lab 4: `pip install pip-audit`
- Labs 5-6: No external dependencies (Python stdlib only)

---

## Repository Structure

```
ai-security-lab/
├── GUIDE.md                  # Lab 1: Prompt Injection Detection setup guide
├── agent.py                  # Lab 1: Prompt injection detector script
├── test_prompts.txt          # Lab 1: Test prompt inputs
├── llm-guardrails-lab/       # Lab 2: LLM guardrails pipeline
├── lab3-network-analysis/    # Lab 3: Network traffic analysis
├── lab4-supply-chain/        # Lab 4: Supply chain security audit
├── lab5-excessive-agency/    # Lab 5: Excessive agency demo
├── lab6-vector-security/     # Lab 6: Vector/embedding security
└── README.md                 # This file
```
