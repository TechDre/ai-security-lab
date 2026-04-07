# Lab 4: Supply Chain Security with pip-audit

## Environment

| Property | Value |
|----------|-------|
| OS | Kali Linux (WSL on Windows 10) |
| Tool | pip-audit |
| Python | 3.13 |
| Date | 2026-04-06 |

## Objective

Simulate a real-world supply chain attack scenario by auditing a Python project with outdated dependencies, identifying known CVEs, and patching them to a clean state.

## Setup

Create an isolated virtual environment:

```bash
python3 -m venv ~/lab4-env
source ~/lab4-env/bin/activate
pip install pip-audit
```

## Phase 1 — Simulate a Vulnerable Project

Created a `requirements.txt` with outdated packages known to have CVEs:

```
requests==2.25.0
flask==2.0.0
pyyaml==5.3.1
cryptography==36.0.0
```

Ran the audit:

```bash
pip-audit -r requirements.txt
```

## Phase 2 — Vulnerability Scan Results

Found 27 known vulnerabilities across 6 packages including transitive dependencies (urllib3, idna) pulled in automatically by requests.

**requests 2.25.0**

| CVE | Fix Version | Risk |
|-----|-------------|------|
| CVE-2024-35195 | 2.32.0 | SSL certificate verification bypass |
| CVE-2024-47081 | 2.32.4 | Credential leakage via redirects |
| CVE-2026-25645 | 2.33.0 | Header injection |

**cryptography 36.0.0** (most critical — 11 CVEs)

| CVE | Fix Version | Risk |
|-----|-------------|------|
| CVE-2023-0286 | 39.0.1 | OpenSSL X.509 memory corruption |
| CVE-2023-50782 | 42.0.0 | RSA decryption vulnerability |
| CVE-2024-0727 | 42.0.2 | PKCS12 parsing crash (DoS) |

**pyyaml 5.3.1**

| CVE | Fix Version | Risk |
|-----|-------------|------|
| PYSEC-2021-142 | 5.4 | Arbitrary code execution via yaml.load() |

**flask 2.0.0**

| CVE | Fix Version | Risk |
|-----|-------------|------|
| PYSEC-2023-62 | 2.2.5 | Open redirect vulnerability |
| CVE-2026-27205 | 3.1.3 | Unspecified security fix |

**urllib3 + idna (transitive dependencies)**

- 4 CVEs in urllib3 — pulled in automatically via requests
- 2 CVEs in idna — pulled in automatically via requests
- Key lesson: You didn’t install these directly but they are still your responsibility to patch.

## Phase 3 — Patch and Re-audit

Updated `requirements.txt` to patched versions:

```
requests==2.33.0
flask==3.1.3
pyyaml==6.0.2
cryptography==46.0.6
```

Re-ran audit:

```bash
pip-audit -r requirements.txt
```

**Result: 0 vulnerabilities found**

## Security Observations

| Finding | Severity | Notes |
|---------|----------|-------|
| 27 CVEs in 4 packages | Critical | Unpatched deps = open attack surface |
| Transitive dep vulnerabilities | High | urllib3/idna not in requirements.txt but still vulnerable |
| PyYAML arbitrary code execution | Critical | yaml.load() on untrusted input = RCE |
| Cryptography library CVEs | High | Broken crypto exposes encrypted data |

## Key Takeaways

1. **Transitive dependencies are invisible risks** — packages you didn’t install can still compromise your app
2. **Version pinning without auditing is dangerous** — pinned versions accumulate CVEs over time
3. **PyYAML is a common attack vector** — always use `yaml.safe_load()` instead of `yaml.load()`
4. **Patch = fix, awareness alone is not enough** — knowing about a CVE without patching leaves you exposed
5. **Typosquatting is a real threat** — `crypotography` vs `cryptography` — one typo installs a malicious package

## OWASP LLM Relevance

**OWASP LLM03: Supply Chain**

This lab directly demonstrates OWASP LLM03 — Supply Chain:

- LLM applications built on vulnerable Python dependencies can be compromised before your own code runs
- An attacker exploiting CVE-2023-0286 in the cryptography library could decrypt LLM API keys or session tokens
- Malicious PyPI packages targeting AI frameworks (e.g. fake `torch` or `transformers` packages) use the same attack surface demonstrated here

## Defensive Recommendations

1. Run `pip-audit` in CI/CD pipeline on every build
2. Use `pip-audit --fix` to auto-update vulnerable packages
3. Pin dependencies AND audit them regularly
4. Use `yaml.safe_load()` — never `yaml.load()`
5. Monitor PyPI advisories for packages you depend on

## Commands Reference

```bash
# Create virtual environment
python3 -m venv ~/lab4-env
source ~/lab4-env/bin/activate

# Install pip-audit
pip install pip-audit

# Audit a requirements file
pip-audit -r requirements.txt

# Auto-fix vulnerabilities
pip-audit -r requirements.txt --fix

# Audit current environment
pip-audit
```
