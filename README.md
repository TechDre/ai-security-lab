# AI Security Lab

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![Security](https://img.shields.io/badge/Focus-AI%20Security-red?logo=shield)

> A hands-on engineering lab for learning AI security — covering prompt injection detection, LLM guardrails, PII redaction, and output filtering.

---

## Overview

The **AI Security Lab** is a collection of practical, self-contained labs focused on protecting LLM-powered applications from real-world attacks. Each lab builds on the previous one, taking you from detection to full pipeline defense.

| Lab | Topic | Key Skills |
|-----|-------|------------|
| [Lab 1 — Prompt Injection Detection](#lab-1--prompt-injection-detection) | Detect adversarial prompts using regex, heuristics, and a DeBERTa classifier | Multi-layer detection, threshold tuning, JSON output |
| [Lab 2 — LLM Guardrails Pipeline](#lab-2--llm-guardrails-pipeline) | Block injections and strip PII before they reach your LLM | Input/output validation, PII redaction, custom content policy |

---

## Repository Structure

```
ai-security-lab/
├── agent.py                   # Lab 1 — Prompt injection detector
├── test_prompts.txt           # Lab 1 — Sample attack prompts
├── GUIDE.md                   # Lab 1 — Full setup & usage guide
└── llm-guardrails-lab/
    ├── agent.py               # Lab 2 — Guardrails pipeline
    ├── test_inputs.txt        # Lab 2 — Sample inputs (safe + malicious)
    └── GUIDE.md               # Lab 2 — Full setup & usage guide
```

---

## Lab 1 — Prompt Injection Detection

**Goal:** Detect prompt injection attacks against LLM applications using three stacked detection layers — no false negatives, minimal false positives.

### Detection Architecture

| Layer | Method | Speed | Requires |
|-------|--------|-------|----------|
| 1 — Regex | 25+ known attack patterns | < 1ms | Nothing |
| 2 — Heuristic | Structural anomaly scoring | < 5ms | Nothing |
| 3 — Classifier | DeBERTa AI model (99%+ accuracy) | ~150ms | `transformers` + `torch` |

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
Verdict         : INJECTION DETECTED
Composite Score : 0.7102
Regex Score     : 0.5000   Matches: [system_prompt_override]
Heuristic Score : 0.3009
Classifier      : INJECTION (1.0000)
Detection Time  : 144.73 ms
```

**Score guide:** 0.0–0.3 safe · 0.3–0.5 suspicious · 0.5–1.0 injection detected

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

Full setup guide: [GUIDE.md](./GUIDE.md)

---

## Lab 2 — LLM Guardrails Pipeline

**Goal:** Build a production-ready middleware layer that validates every input and output around your LLM — blocking attacks, stripping PII, and enforcing content policies.

### Pipeline Architecture

```
User Input
    |
    v
[Length Guard]       <- Blocks oversized inputs
    |
    v
[Injection Guard]    <- Blocks prompt injections & jailbreaks
    |
    v
[Content Policy]     <- Blocks harmful or off-topic requests
    |
    v
[PII Guard]          <- Strips SSNs, emails, credit cards, API keys
    |
    v
  LLM API            <- Receives only clean, sanitized input
    |
    v
[Output Guard]       <- Catches system prompt leakage & PII in responses
    |
    v
User Response        <- Safe, redacted output
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
|----------|--------------|---------------|
| US SSN | 123-45-6789 | `[SSN_REDACTED]` |
| Email | user@example.com | `[EMAIL_REDACTED]` |
| Credit Card | 4111 1111 1111 1111 | `[CARD_REDACTED]` |
| Phone | (555) 123-4567 | `[PHONE_REDACTED]` |
| IP Address | 192.168.1.1 | `[IP_REDACTED]` |
| AWS Key | AKIA... | `[AWS_KEY_REDACTED]` |

### Embed in Your Application

```python
from agent import GuardrailsPipeline

pipeline = GuardrailsPipeline()  # or pass policy_path="custom_policy.json"

# --- Before sending to LLM ---
result = pipeline.validate_input(user_message)
if not result.safe:
    return f"Request blocked: {result.blocked_reason}"

# Send sanitized text (PII already stripped)
llm_response = your_llm.generate(result.sanitized_text)

# --- Before returning to user ---
output = pipeline.validate_output(llm_response)
return output.sanitized_text
```

Full setup guide: [llm-guardrails-lab/GUIDE.md](./llm-guardrails-lab/GUIDE.md)

---

## Prerequisites

| Lab | Python Version | External Dependencies |
|-----|---------------|----------------------|
| Lab 1 | 3.10+ | `transformers`, `torch`, `sentencepiece`, `protobuf` |
| Lab 2 | 3.10+ | None — pure Python stdlib |

---

## Learning Path

This lab is designed as a progressive curriculum in AI security engineering:

1. **Lab 1 — Prompt Injection Detection** — Understand and detect adversarial inputs
2. **Lab 2 — LLM Guardrails Pipeline** — Build a full input/output defense layer
3. *Coming next:* Malware Behavior Analysis with Cuckoo Sandbox
4. *Coming next:* Network Traffic Analysis with Wireshark

---

## License

This project is open source and available under the [MIT License](LICENSE).
