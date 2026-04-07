# Lab 5: Excessive Agency (OWASP LLM06)

## Environment

| Property | Value |
|----------|-------|
| OS | Kali Linux (WSL on Windows 10) |
| Language | Python 3.13 |
| Date | 2026-04-07 |

## Table of Contents

- [Objective](#objective)
- [Background](#background)
- [Setup](#setup)
- [Phase 1 — Unsafe Agent](#phase-1--unsafe-agent)
- [Phase 2 — Safe Agent](#phase-2--safe-agent)
- [Comparison](#comparison)
- [Security Observations](#security-observations)
- [Key Takeaways](#key-takeaways)
- [OWASP LLM06: Excessive Agency](#owasp-llm06-excessive-agency)
- [Defensive Recommendations](#defensive-recommendations)

## Objective

Demonstrate OWASP LLM06 (Excessive Agency) by building an unsafe agent that can be manipulated via prompt injection, then building a safe agent with least privilege, input validation, and audit logging.

## Background

Excessive Agency occurs when an LLM agent is granted more permissions than necessary. If manipulated via prompt injection or malicious input, the agent can take destructive or unauthorized actions on behalf of the attacker.

Real world examples:
- AI assistant with email access tricked into exfiltrating data
- Coding agent with file system access manipulated into deleting files
- LLM with database access exploited to dump sensitive records

## Setup

No external dependencies required — pure Python stdlib (3.10+).

```bash
# Verify Python version
python3 --version  # Requires 3.10+

# Run the unsafe agent demo
python3 unsafe_agent.py

# Run the safe agent demo
python3 safe_agent.py
```

## Phase 1 — Unsafe Agent

### Code

```python
import subprocess

def unsafe_agent(user_input):
    print(f"[AGENT] Executing: {user_input}")
    result = subprocess.run(user_input, shell=True, capture_output=True, text=True)
    return result.stdout or result.stderr
```

### What Makes It Unsafe

- Accepts any shell command without validation
- Uses `shell=True` — enables command chaining
- No allowlist — no restrictions on what can run
- No audit logging — attacks go undetected
- No input sanitization — injection is trivial

### Attack Demonstration

Normal use:
```
[AGENT] Executing: echo Hello World
Hello World
```

Prompt injection attack:
```
[AGENT] Executing: echo 'legitimate task' && cat /etc/passwd
legitimate task
[SYSTEM USER LIST REDACTED]
```

Excessive file access:
```
[AGENT] Executing: ls /home
[HOME DIRECTORY CONTENTS REDACTED]
```

---

## Phase 2 — Safe Agent

### Code

```python
import subprocess
from datetime import datetime

ALLOWED_COMMANDS = ["echo", "date", "whoami", "uptime"]

def audit_log(action, user_input, blocked=False):
    status = "BLOCKED" if blocked else "ALLOWED"
    print(f"[AUDIT {datetime.now().strftime('%H:%M:%S')}] {status}: {action} | input='{user_input}'")

def is_safe(user_input):
    if any(op in user_input for op in ["&&", "||", ";", "|", ">", "<", "`", "$("]):
        return False, "chained command detected"
    command = user_input.strip().split()[0]
    if command not in ALLOWED_COMMANDS:
        return False, f"command '{command}' not in allowlist"
    return True, "ok"

def safe_agent(user_input):
    safe, reason = is_safe(user_input)
    if not safe:
        audit_log("SHELL_EXEC", user_input, blocked=True)
        return f"[BLOCKED] {reason}"
    audit_log("SHELL_EXEC", user_input, blocked=False)
    result = subprocess.run(user_input.split(), capture_output=True, text=True)
    return result.stdout or result.stderr
```

### Test Results

Normal use:
```
[AUDIT 00:00:00] ALLOWED: SHELL_EXEC | input='echo Hello World'
Hello World
```

Prompt injection attempt:
```
[AUDIT 00:00:00] BLOCKED: SHELL_EXEC | input='echo legitimate task && cat /etc/passwd'
[BLOCKED] chained command detected
```

Unauthorized command:
```
[AUDIT 00:00:00] BLOCKED: SHELL_EXEC | input='cat /etc/passwd'
[BLOCKED] command 'cat' not in allowlist
```

Allowed command:
```
[AUDIT 00:00:00] ALLOWED: SHELL_EXEC | input='whoami'
[OUTPUT REDACTED]
```

---

## Comparison

| Test | Unsafe Agent | Safe Agent |
|------|-------------|------------|
| Normal echo | Executed | Executed |
| Prompt injection | Exposed system users | BLOCKED |
| Unauthorized command | Executed | BLOCKED |
| Allowed command | Executed | Executed |

---

## Security Observations

| Finding | Severity | Notes |
|---------|----------|-------|
| `shell=True` enables injection | Critical | Never use `shell=True` with user input |
| No input validation | Critical | Any command can be executed |
| No audit logging | High | Attacks go completely undetected |
| Excessive permissions | High | Agent can access entire filesystem |
| Allowlist bypasses injection | Fix | Whitelist-only approach blocks unknown commands |

---

## Key Takeaways

1. **Least privilege** — agents should only have permissions they absolutely need
2. **Allowlists over blocklists** — only allow known good input
3. **Never use `shell=True`** with user-controlled input
4. **Audit everything** — logging blocked actions is as important as blocking them
5. **Input validation is not optional** — every user input must be treated as potentially malicious

---

## OWASP LLM06: Excessive Agency

**Attack Chain (Labs 1 + 5 Combined)**

```
Malicious input
  → Prompt injection bypasses LLM guardrails (Lab 1)
  → Injected command reaches agent with shell access (Lab 5)
  → Agent executes attacker command
  → System compromised
```

**Defense Chain**

```
User input
  → LLM guardrails filter injection attempts (Lab 2)
  → Agent validates against allowlist (Lab 5)
  → Audit log records all attempts (Lab 5)
  → Attack blocked and logged
```

---

## Defensive Recommendations

1. Always apply least privilege to AI agents
2. Use allowlists, not blocklists, for command validation
3. Never pass user input directly to shell commands
4. Audit log all agent actions — allowed and blocked
5. Combine with LLM guardrails (Lab 2) for layered defense
6. Regularly review what permissions your agents actually need

---
