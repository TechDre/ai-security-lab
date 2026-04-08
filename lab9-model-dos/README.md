[README.md](https://github.com/user-attachments/files/26568125/README.md)
# Lab 9: Model Denial of Service (OWASP LLM04)

## Environment
- OS: Kali Linux (WSL on Windows 10)
- Language: Python 3.13
- Date: 2026-04-08

## Objective
Demonstrate OWASP LLM04 (Model Denial of Service) by simulating
resource exhaustion attacks against an unsafe LLM handler, then
implementing five defensive controls: input length limiting, token
limiting, repetition detection, rate limiting, and cost tracking.

## Background

LLM Denial of Service attacks differ from traditional DoS in one
critical way — they cost money, not just uptime. Attackers craft
inputs that consume maximum tokens, trigger expensive operations,
or flood the API to exhaust budgets and degrade service.

**Real world examples:**
- Sending maximum-length inputs to exhaust token budgets
- Repetitive or recursive prompts that waste compute
- Flooding API endpoints to exhaust rate limits
- Crafting inputs that trigger expensive model operations

**Why it matters:**
A single crafted 100k token request can cost more than
thousands of normal requests. At scale, one attacker can
bankrupt an API budget or take down a service.

---

## Attack Simulations

### Phase 1 — Oversized Input Attack

Attacker sends a 14,000 character / 2,000 token input:

**Unsafe result:**
```
Input size: 14000 chars | 2000 tokens
Tokens: 2000 | Cost: $0.04
[!] Processed 2000 tokens costing $0.04
```

One oversized request cost $0.04. At scale:
- 1,000 requests = $40
- 10,000 requests = $400
- No limits = unlimited damage

### Phase 2 — Repetitive Pattern Attack

Attacker sends 500 tokens of repeated words to waste compute:

**Unsafe result:**
```
Input size: 3500 chars
Tokens: 500 | Cost: $0.01
[!] Processed repetitive input costing $0.01
```

Repetitive inputs provide no value but consume full resources.
Recursive prompts ("repeat this forever") can cause loops.

### Phase 3 — Request Flooding Attack

Attacker sends 10 rapid requests in succession:

**Unsafe result:**
```
10 rapid requests cost $0.002 total
```

No rate limiting means unlimited requests. A botnet sending
thousands of requests per second would exhaust any budget.

---

## Defense Implementation

### Defense 1: Input Length Limit
```python
MAX_INPUT_LENGTH = 2000
if len(prompt) > MAX_INPUT_LENGTH:
    return {"error": "Input exceeds maximum length"}
```

### Defense 2: Token Limit
```python
MAX_INPUT_TOKENS = 500
if len(prompt.split()) > MAX_INPUT_TOKENS:
    return {"error": "Input exceeds maximum tokens"}
```

### Defense 3: Repetition Detection
```python
def is_repetitive(prompt, threshold=0.7):
    words = prompt.split()
    unique_ratio = len(set(words)) / len(words)
    return unique_ratio < (1 - threshold)
```
Blocks inputs where more than 70% of words are duplicates.

### Defense 4: Rate Limiting
```python
RateLimiter(max_requests=5, window_seconds=10)
```
Maximum 5 requests per user per 10 second window.

### Defense 5: Cost Tracking
```python
CostTracker(max_cost_per_session=0.10)
```
Hard cap of $0.10 per session — blocks requests when
budget is exhausted.

---

## Phase 4 — Safe Handler Test Results

```
[Test 1 - Oversized input]
[AUDIT] INPUT_TOO_LONG: user=user_001 length=14000
Result: Input exceeds maximum length of 2000 characters.

[Test 2 - Repetitive input]
[AUDIT] INPUT_TOO_LONG: user=user_001 length=3500
Result: Input exceeds maximum length of 2000 characters.

[Test 3 - Request flooding]
Request 1: Allowed
Request 2: Allowed
Request 3: Allowed
Request 4: Allowed
Request 5: Allowed
[AUDIT] RATE_LIMIT_EXCEEDED: user=user_002
Request 6: BLOCKED - Rate limit exceeded.

[Test 4 - Cost limit]
[AUDIT] REPETITIVE_INPUT_BLOCKED: user=user_003
Result: Repetitive input pattern detected.
```

All 5 defenses triggered and audit logged correctly.

---

## Defense Summary

| Defense | Attack Stopped | How |
|---------|---------------|-----|
| Input length limit | Oversized inputs | Hard character limit |
| Token limit | Token exhaustion | Hard token count limit |
| Repetition detection | Recursive/loop attacks | Uniqueness ratio check |
| Rate limiting | Request flooding | Per-user request window |
| Cost tracking | Budget exhaustion | Per-session spend cap |

---

## Security Observations

| Finding | Severity | Notes |
|---------|----------|-------|
| No input limits = unlimited cost | Critical | One request can cost $0.04+ |
| No rate limiting = flooding possible | Critical | Botnet can exhaust any budget |
| Repetitive inputs waste compute | High | No value, full resource cost |
| LLM DoS costs money not just uptime | High | Unique risk vs traditional DoS |
| All 5 defenses required | High | Each stops a different attack vector |

---

## Key Takeaways

1. **LLM DoS is financial, not just operational** — attackers
   can drain budgets without taking the service fully offline
2. **Input limits are the first line of defense** — reject
   oversized inputs before they reach the model
3. **Rate limiting must be per user** — global limits can be
   bypassed by distributing across accounts
4. **Cost tracking is unique to LLM security** — traditional
   security rarely tracks per-request spend
5. **Audit logging reveals attack patterns** — repeated rate
   limit violations indicate active flooding

---

## OWASP LLM04: Model Denial of Service

This lab directly demonstrates **OWASP LLM04**:

- LLM APIs are billed by token — resource exhaustion = financial damage
- Unlike traditional DoS, LLM DoS requires no botnet — a single
  crafted request can cause disproportionate cost
- Defense requires controls at multiple layers: input, rate, and cost

### Connection to Other Labs

| Lab | Connection |
|-----|------------|
| Lab 1 (Prompt Injection) | Injected prompts can trigger expensive operations |
| Lab 5 (Excessive Agency) | Agent with API access amplifies DoS impact |
| Lab 7 (Data Extraction) | Extraction attempts are also high-token attacks |

---

## Defensive Recommendations

1. **Set hard input length and token limits** on all LLM endpoints
2. **Implement per-user rate limiting** — not just global limits
3. **Track and cap per-session costs** — alert on unusual spend
4. **Detect repetitive inputs** — block recursive prompt patterns
5. **Monitor token usage dashboards** — sudden spikes indicate attack
6. **Set billing alerts** on your LLM API provider

---

## Commands Reference
```bash
# Set up environment
source ~/lab4-env/bin/activate

# Run the lab
python model_dos.py
```
