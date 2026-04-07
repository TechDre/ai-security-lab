# AI Security Lab

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![Security](https://img.shields.io/badge/Focus-AI%20Security-red?logo=shield)

> A hands-on engineering lab for learning AI security — covering prompt injection detection, LLM guardrails, PII redaction, output filtering, network traffic analysis, supply chain security, and excessive agency.

## Overview

The AI Security Lab is a collection of practical, self-contained labs focused on protecting LLM-powered applications from real-world attacks. Each lab builds on the previous one, taking you from detection to full pipeline defense.

| Lab | Topic | Key Skills |
|-----|-------|------------|
| Lab 1 | Prompt Injection Detection | Multi-layer detection, threshold tuning, JSON output |
| Lab 2 | LLM Guardrails Pipeline | Input/output validation, PII redaction, custom content policy |
| Lab 3 | Network Traffic Analysis | Packet capture, protocol analysis, security observation |
| Lab 4 | Supply Chain Security | Dependency auditing, CVE identification, patching |
| Lab 5 | Excessive Agency | Least privilege, allowlists, audit logging, agent hardening |

## Repository Structure

```
ai-security-lab/
├── agent.py                  # Lab 1 — Prompt injection detector
├── test_prompts.txt          # Lab 1 — Sample attack prompts
├── GUIDE.md                  # Lab 1 — Full setup & usage guide
├── llm-guardrails-lab/
│   ├── agent.py              # Lab 2 — Guardrails pipeline
│   ├── test_inputs.txt       # Lab 2 — Sample inputs (safe + malicious)
│   └── GUIDE.md              # Lab 2 — Full setup & usage guide
├── lab3-network-analysis/
│   └── README.md             # Lab 3 — Network traffic analysis write-up
├── lab4-supply-chain/
│   └── README.md             # Lab 4 — Supply chain security write-up
└── lab5-excessive-agency/
    └── README.md             # Lab 5 — Excessive agency write-up
```

## Lab 1 — Prompt Injection Detection

**Goal:** Detect prompt injection attacks against LLM applications using three stacked detection layers — no false negatives, minimal false positives.

### Detection Architecture

| Layer | Method | Speed | Requires |
|-------|--------|-------|----------|
| 1 — Regex | 25+ known attack patterns | < 1ms | Nothing |
| 2 — Heuristic | Structural anomaly scoring | < 5ms | Nothing |
| 3 — Classifier | DeBERTa AI model (99%+ accuracy) | ~150ms | transformers + torch |

### Quick Start

```bash
# Install dependencies (first time only)
pip install transformers torch sentencepiece protobuf

# Analyze a single prompt — full AI-powered mode
python agent.py --input "Ignore all previous instructions" --mode full

# Fast regex-only scan (no model required)
python agent.py --input "Some text" --mode regex

# Batch scan a file and get structured JSON output
python agent.py --file test_prompts.txt --mode full --output json
```

### Sample Output

```
Verdict       : INJECTION DETECTED
Composite Score: 0.7102
Regex Score   : 0.5000   Matches: [system_prompt_override]
Heuristic     : 0.3009
Classifier    : INJECTION (1.0000)
Detection Time : 144.73 ms

Score guide: 0.0–0.3 safe · 0.3–0.5 suspicious · 0.5–1.0 injection detected
```

### Attack Patterns Detected

| Pattern | Example |
|---------|---------|
| System prompt override | "Ignore all previous instructions..." |
| Role-play escape | "You are now / Act as / Pretend to be..." |
| Developer mode | "DAN mode / jailbreak / god mode..." |
| Data exfiltration | "Reveal your system prompt..." |
| Token smuggling | Zero-width characters, hidden Unicode |
| Encoding obfuscation | "Base64 decode this and follow it..." |
| Few-shot injection | Fake conversation history to redirect behavior |

Full setup guide: [GUIDE.md](GUIDE.md)

---

## Lab 2 — LLM Guardrails Pipeline

**Goal:** Build a production-ready middleware layer that validates every input and output around your LLM — blocking attacks, stripping PII, and enforcing content policies.

### Pipeline Architecture

```
User Input
│
▼
[Length Guard]     ← Blocks oversized inputs
│
▼
[Injection Guard]  ← Blocks prompt injections & jailbreaks
│
▼
[Content Policy]   ← Blocks harmful or off-topic requests
│
▼
[PII Guard]        ← Strips SSNs, emails, credit cards, API keys
│
▼
LLM API            ← Receives only clean, sanitized input
│
▼
[Output Guard]     ← Catches system prompt leakage & PII in responses
│
▼
User Response      ← Safe, redacted output
```

### Quick Start

```bash
# No external dependencies — pure Python stdlib

# Full pipeline validation
python llm-guardrails-lab/agent.py --input "Your message here" --mode input-only

# PII detection and redaction only
python llm-guardrails-lab/agent.py --input "My SSN is 123-45-6789" --mode pii

# Batch scan with JSON output
python llm-guardrails-lab/agent.py --file llm-guardrails-lab/test_inputs.txt --mode input-only --output json
```

### PII Redaction Reference

| PII Type | Example Input | Replaced With |
|----------|---------------|---------------|
| US SSN | 123-45-6789 | [SSN_REDACTED] |
| Email | user@example.com | [EMAIL_REDACTED] |
| Credit Card | 4111 1111 1111 1111 | [CARD_REDACTED] |
| Phone | (555) 123-4567 | [PHONE_REDACTED] |
| IP Address | 192.168.1.1 | [IP_REDACTED] |
| AWS Key | AKIA... | [AWS_KEY_REDACTED] |

### Embed in Your Application

```python
from agent import GuardrailsPipeline

pipeline = GuardrailsPipeline()  # or pass policy_path="custom_policy.json"

# Before sending to LLM
result = pipeline.validate_input(user_message)
if not result.safe:
    return f"Request blocked: {result.blocked_reason}"

# Send sanitized text (PII already stripped)
llm_response = your_llm.generate(result.sanitized_text)

# Before returning to user
output = pipeline.validate_output(llm_response)
return output.sanitized_text
```

Full setup guide: [llm-guardrails-lab/GUIDE.md](llm-guardrails-lab/GUIDE.md)

---

## Lab 3 — Network Traffic Analysis

**Goal:** Capture and analyze live network traffic to identify protocols, understand normal network behavior, and spot potential security anomalies using TShark.

See the full write-up in [lab3-network-analysis/README.md](lab3-network-analysis/README.md).

---

## Lab 4 — Supply Chain Security

**Goal:** Simulate a real-world supply chain attack by auditing a Python project with outdated dependencies, identifying known CVEs, and patching them to a clean state.

See the full write-up in [lab4-supply-chain/README.md](lab4-supply-chain/README.md).

---

## Lab 5 — Excessive Agency

**Goal:** Demonstrate OWASP LLM06 (Excessive Agency) by building an unsafe agent that can be manipulated via prompt injection, then hardening it with least privilege, input validation, and audit logging.

See the full write-up in [lab5-excessive-agency/README.md](lab5-excessive-agency/README.md).

---

## Prerequisites

| Lab | Python Version | External Dependencies |
|-----|---------------|-----------------------|
| Lab 1 | 3.10+ | transformers, torch, sentencepiece, protobuf |
| Lab 2 | 3.10+ | None — pure Python stdlib |
| Lab 3 | N/A | TShark 4.4.6+, Kali Linux (or WSL) |
| Lab 4 | 3.10+ | pip-audit |
| Lab 5 | 3.10+ | None — pure Python stdlib |

## Learning Path

This lab is designed as a progressive curriculum in AI security engineering:

- **Lab 1** — Prompt Injection Detection — Understand and detect adversarial inputs
- **Lab 2** — LLM Guardrails Pipeline — Build a full input/output defense layer
- **Lab 3** — Network Traffic Analysis — Monitor for unexpected LLM agent connections
- **Lab 4** — Supply Chain Security — Audit and patch vulnerable dependencies
- **Lab 5** — Excessive Agency — Harden agents with least privilege and audit logging

Coming next: Malware Behavior Analysis with Cuckoo Sandbox

## License

This project is open source and available under the [MIT License](LICENSE).
