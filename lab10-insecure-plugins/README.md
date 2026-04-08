[README.md](https://github.com/user-attachments/files/26569270/README.md)
# Lab 10: Insecure Plugin/Tool Use (OWASP LLM07)

## Environment
- OS: Kali Linux (WSL on Windows 10)
- Language: Python 3.13
- Date: 2026-04-08

## Objective
Demonstrate OWASP LLM07 (Insecure Plugin Design) by simulating
attacks against unsecured LLM tools (web search, code executor,
email sender), then implementing per-tool defenses: output
scanning, blocked pattern lists, and domain allowlisting.

## Background

Modern LLM applications give models access to tools — web search,
code execution, email, databases, file systems. Each tool is an
independent attack surface. Securing the LLM itself is not enough
— every tool it can call must have its own validation layer.

**Key distinction from Lab 5 (Excessive Agency):**
- Lab 5: Agent has TOO MANY permissions
- Lab 10: Agent has the RIGHT permissions but tools are unsecured

**Real world examples:**
- LLM web search retrieves malicious content injected into results
- LLM code executor runs attacker-supplied system commands
- LLM email tool sends data to unauthorized external addresses
- LLM database tool returns records it should not have access to

---

## Tool Architecture

```
User Input
    → LLM processes request
    → LLM calls tool (web_search / code_executor / email_sender)
    → Tool returns result
    → LLM incorporates result into response

Vulnerability: No validation at tool input OR output layer
```

---

## Phase 1 — Malicious Web Search Result

**Attack:** Web search returns a result containing a malicious
script tag embedded in otherwise legitimate content.

**Unsafe result:**
```
Result passed to LLM: Today's weather is sunny.
<script>steal_credentials()</script>
[!] LLM received malicious script tag in search result!
```

The LLM received and processed malicious JavaScript. In a
browser-rendered application this executes and steals credentials.

**Defense — Output Scanner:**
Scan all tool output before passing back to the LLM:
```python
blocked = ["<script>", "javascript:", "steal_", "eval(", "exec("]
```

**Safe result:**
```
[AUDIT] MALICIOUS_SEARCH_RESULT: blocked output from query: weather today
Result: Search result contained malicious content and was blocked.
```

---

## Phase 2 — Dangerous Code Execution

**Attack:** LLM manipulated into passing a system command
through the code execution tool.

**Unsafe result:**
```
[EXECUTING CODE]: import os; os.system('rm -rf /tmp/test')
Result: [SIMULATED] System command executed - files deleted
[!] Dangerous system command executed via code tool!
```

A single line of attacker-controlled code deleted files.
In a real system this could wipe databases or exfiltrate data.

**Defense — Blocked Pattern List:**
```python
BLOCKED_CODE_PATTERNS = [
    "import os", "import subprocess", "import sys",
    "open(", "exec(", "eval(", "__import__"
]
```

**Safe result:**
```
[AUDIT] DANGEROUS_CODE_BLOCKED: user=user_001 pattern=import os
Result: Code contains blocked pattern: import os
```

---

## Phase 3 — Unauthorized Email Sending

**Attack:** LLM tricked into sending customer data to an
external attacker-controlled email address.

**Unsafe result:**
```
[EMAIL SENT] To: [ATTACKER-EMAIL-REDACTED] | Subject: Confidential Data
Body: Here are all the customer records you requested...
[!] Email sent to unauthorized external domain!
```

No domain validation meant the LLM could email any address —
a direct data exfiltration channel.

**Defense — Domain Allowlist:**
```python
ALLOWED_EMAIL_DOMAINS = ["company.com", "trusted-partner.com"]
```

**Safe result:**
```
[AUDIT] EMAIL_BLOCKED_DOMAIN: user=user_001 domain=[REDACTED]
Result: Email domain not in approved list.
```

---

## Phase 4 — Safe Plugin Handler Test Results

```
[Test 1 - Malicious search result]
BLOCKED - Search result contained malicious content.

[Test 2 - Dangerous code]
BLOCKED - Code contains blocked pattern: import os

[Test 3 - Unauthorized email domain]
BLOCKED - Email domain not in approved list.

[Test 4 - Legitimate email]
ALLOWED - Email sent to colleague@company.com

[Test 5 - Safe code]
ALLOWED - Simple arithmetic code executed
```

All attacks blocked. Legitimate use cases still work.

---

## Defense Summary

| Tool | Attack | Defense |
|------|--------|---------|
| Web search | Malicious result injection | Output scanner |
| Code executor | System command execution | Blocked pattern list |
| Email sender | Data exfiltration | Domain allowlist |

---

## Security Observations

| Finding | Severity | Notes |
|---------|----------|-------|
| Web search returns malicious content | Critical | LLM processes and propagates it |
| Code executor runs system commands | Critical | Full system compromise possible |
| Email tool has no domain restriction | Critical | Direct data exfiltration channel |
| No tool has input or output validation | Critical | Every tool is an open attack surface |
| Output scanning catches injected content | Fix | Last defense before LLM receives result |
| Pattern blocklist stops dangerous code | Fix | Prevents system-level access |
| Domain allowlist prevents exfiltration | Fix | Restricts email to trusted recipients |

---

## Key Takeaways

1. **Every tool is an independent attack surface** — securing
   the LLM is not enough, each tool needs its own defenses
2. **Tool output is untrusted** — treat results from web
   search and external APIs like user input
3. **Allowlists beat blocklists** — define what is permitted,
   block everything else (same principle as Lab 5)
4. **Scan in both directions** — validate tool input AND
   scan tool output before returning to LLM
5. **Least privilege applies to tools** — only expose tools
   the LLM actually needs for its specific task

---

## OWASP LLM07: Insecure Plugin Design

This lab directly demonstrates **OWASP LLM07**:

- LLM plugins extend the attack surface beyond the model
- Each tool integration point requires independent security controls
- Attackers target the weakest tool, not the strongest defense

### Connection to Other Labs

| Lab | Connection |
|-----|------------|
| Lab 1 (Prompt Injection) | Injection manipulates LLM to misuse tools |
| Lab 5 (Excessive Agency) | Too many tools + insecure tools = maximum risk |
| Lab 8 (Insecure Output) | Tool output needs same sanitization as LLM output |
| Lab 9 (Model DoS) | Tools can be abused to exhaust resources |

---

## Complete Lab Series — OWASP Coverage

| Lab | Topic | OWASP |
|-----|-------|-------|
| Lab 1 | Prompt injection detector | LLM01 |
| Lab 2 | LLM guardrails pipeline | LLM01 |
| Lab 3 | Network traffic analysis | LLM06 |
| Lab 4 | Supply chain security | LLM03 |
| Lab 5 | Excessive agency | LLM06 |
| Lab 6 | Vector/embedding security | LLM08 |
| Lab 7 | Model inversion & data extraction | LLM02 |
| Lab 8 | Insecure output handling | LLM02 |
| Lab 9 | Model denial of service | LLM04 |
| Lab 10 | Insecure plugin/tool use | LLM07 |

---

## Defensive Recommendations

1. **Validate all tool inputs** before execution
2. **Scan all tool outputs** before passing to LLM
3. **Use domain allowlists** for any communication tools
4. **Block dangerous code patterns** in execution tools
5. **Apply least privilege** — only expose necessary tools
6. **Audit log all tool calls** — blocked and allowed
7. **Treat external tool results as untrusted data**

---

## Commands Reference
```bash
# Set up environment
source ~/lab4-env/bin/activate

# Run the lab
python insecure_plugins.py
```
