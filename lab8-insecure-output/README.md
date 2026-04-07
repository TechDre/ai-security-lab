[README.md](https://github.com/user-attachments/files/26546348/README.md)
# Lab 8: Insecure Output Handling (OWASP LLM02)

## Environment
- OS: Kali Linux (WSL on Windows 10)
- Language: Python 3.13
- Date: 2026-04-07

## Objective
Demonstrate insecure output handling by showing how LLM output
passed directly to downstream systems (browser, database, shell)
can deliver XSS, SQL injection, and command injection attacks.
Then implement output sanitization defenses for each attack type.

## Background

When LLM output is passed to another system without sanitization,
the LLM becomes a delivery mechanism for classic injection attacks.
The attacker does not need to hack the LLM itself — they just need
to craft input knowing where the output will flow.

**Key distinction from Lab 7:**
- Lab 7: Sensitive data leaking OUT of the LLM
- Lab 8: Malicious code flowing THROUGH the LLM into downstream systems

**Real world examples:**
- LLM generates HTML with embedded script tags → XSS in browser
- LLM output interpolated into SQL query → database compromise
- LLM response passed to shell command → command execution
- LLM generates markdown with malicious links → phishing via UI

---

## Attack Flow

```
Attacker crafts malicious input
    → LLM processes and echoes it back in output
    → Output passed unsanitized to downstream system
    → Browser / Database / Shell executes malicious payload
    → System compromised
```

---

## Phase 1 — XSS Attack via LLM Output

**Attack payload:** Script tag injected through LLM output

**Unsafe result:**
```
[HTML RENDER]: ...content...<script>...</script>
[!] XSS ATTACK EXECUTED - Script tag rendered in browser!
```

In a real browser, the script tag would execute JavaScript,
stealing session cookies and sending them to an attacker.

**Defense — HTML escaping:**
```python
text.replace("<", "&lt;").replace(">", "&gt;")
```

**Safe result:**
```
[HTML RENDER - SAFE]: &lt;script&gt;...&lt;/script&gt;
[+] Script tags escaped - XSS blocked!
```

The browser renders the output as plain text — not executable code.

---

## Phase 2 — SQL Injection via LLM Output

**Attack payload:** SQL injection string through LLM output

**Unsafe result:**
```
[SQL QUERY]: SELECT * FROM users WHERE name = ''
OR '1'='1'; DROP TABLE users; --'
[!] SQL INJECTION ATTACK - Query manipulated by LLM output!
```

This query would return every user in the database AND
drop the entire users table.

**Defense — SQL sanitization:**
```python
# Remove dangerous characters and keywords
dangerous = ["'", '"', ";", "--", "DROP", "DELETE", "INSERT"]
# In production: always use parameterized queries
```

**Safe result:**
```
[SQL QUERY - SAFE]: SELECT * FROM users WHERE name =
' OR 1=1  TABLE users '
[+] SQL injection characters removed!
```

**Note:** In production always use parameterized queries
instead of string sanitization — they are more reliable.

---

## Phase 3 — Command Injection via LLM Output

**Attack payload:** Shell command chaining through LLM output

**Unsafe result:**
```
[SHELL CMD]: echo hello; cat /etc/passwd
[!] COMMAND INJECTION ATTACK - Shell command hijacked!
```

The semicolon chained a second command, same attack vector
as demonstrated in Lab 5 (Excessive Agency).

**Defense — Shell sanitization:**
```python
re.sub(r'[;&|`$><\\]', '', text)
```

**Safe result:**
```
[SHELL CMD - SAFE]: echo hello cat /etc/passwd
[+] Shell injection characters removed!
```

---

## Comparison: Unsafe vs Safe

| Attack | Unsafe Result | Safe Result |
|--------|--------------|-------------|
| XSS | Script executed in browser | Tags escaped to plain text |
| SQL Injection | Database dumped and dropped | Dangerous chars stripped |
| Command Injection | Shell command hijacked | Metacharacters removed |

---

## Security Observations

| Finding | Severity | Notes |
|---------|----------|-------|
| LLM output passed to HTML renderer | Critical | XSS via script tag injection |
| LLM output interpolated into SQL | Critical | Full database compromise possible |
| LLM output passed to shell | Critical | Command execution (see Lab 5) |
| No output sanitization = all attacks succeed | Critical | One missing control = full compromise |
| HTML escaping blocks XSS | Fix | Convert special chars to HTML entities |
| Parameterized queries block SQL injection | Fix | Never interpolate into SQL strings |
| Shell metachar stripping blocks injection | Fix | Combine with allowlisting from Lab 5 |

---

## Key Takeaways

1. **The LLM is the delivery mechanism, not the attacker** —
   attackers craft input knowing where output will flow
2. **Every downstream system needs its own sanitization** —
   HTML, SQL, and shell each have different dangerous characters
3. **Never interpolate LLM output directly** into queries,
   commands, or rendered HTML
4. **Parameterized queries > string sanitization** for SQL —
   Sanitization can be bypassed, but parameterization cannot
5. **Output handling is an application responsibility** —
   the LLM will not protect you from this

---

## OWASP LLM02: Insecure Output Handling

This lab demonstrates the output handling dimension of
**OWASP LLM02**:

- LLM output is untrusted data — treat it like user input
  to any other system
- The attack surface expands with every downstream system
  the LLM feeds into
- Defense must be applied at each integration point

### Connection to Other Labs

| Lab | Connection |
|-----|------------|
| Lab 1 (Prompt Injection) | Attacker input → LLM output → downstream system |
| Lab 2 (Guardrails) | Output guardrails catch malicious LLM responses |
| Lab 5 (Excessive Agency) | Command injection amplified by agent shell access |
| Lab 7 (Data Extraction) | Both are LLM02 — input extraction vs output injection |

---

## Defensive Recommendations

1. **Treat LLM output as untrusted** — sanitize before
   passing to any downstream system
2. **Use parameterized queries** — never interpolate LLM
   output into SQL strings
3. **HTML escape all output** rendered in browsers
4. **Apply shell metachar filtering** before any exec call
5. **Combine with Lab 5 defenses** — allowlisting + output
   sanitization for agent tool use
6. **Audit log all sanitization events** — blocked payloads
   reveal active attack patterns

---

## Commands Reference
```bash
# Set up environment
source ~/lab4-env/bin/activate

# Run the lab
python insecure_output.py
```
