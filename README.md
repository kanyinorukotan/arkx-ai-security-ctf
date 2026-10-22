# ARKX AI Agent Security CTF — Red-Team Findings

Red-team assessment of six AI agent challenges on the [ARKX](https://ctf.arkx.ninja) platform, a benchmark environment for testing AI agent security. All six required levels were solved, each exposing a distinct class of vulnerability in how an AI agent's tools handle untrusted input.

> **Scope note:** This assessment was performed against a dedicated CTF/training environment designed for this purpose, not a production system. Live session cookies, access tokens, and credentials have been omitted from this report.

## Summary

| Track / Level | Name | Vulnerability Class | Severity |
|---|---|---|---|
| Advanced 5 | Bad Robot | Code injection (`eval()` on unsanitized input) | High / Critical |
| Advanced 7 | Master of Resources | Exposed backend endpoint bypassing UI-level access control | Low / Informational |
| Advanced 9 | Blind Network Ninja | OS command injection + out-of-band data exfiltration | High |
| Expert 6 | The Shape Shifter | Insecure deserialization (unsafe YAML loader) | High / Critical |
| Expert 7 | The Octopus | Multi-agent privilege escalation via parser differential | High |
| Expert 8 | The Heist | TOCTOU race condition (double-spend) | High |

All six levels solved, with a flag recovered for every level.

## What this project demonstrates

- **Vulnerability discovery** in AI agent tool design — not just prompting the model, but probing how its tools handle input at the code level
- **Exploit construction and verification** — building minimal, reproducible payloads and independently confirming execution (e.g. reading back an application's own log endpoint to prove out-of-band exfiltration actually landed, rather than trusting the tool's own "success" message)
- **Root cause analysis** distinguishing the proximate trigger (a crafted input) from the underlying design flaw (an unsandboxed interpreter, a naive blocklist, a missing atomic transaction)
- **Structured reporting** — each finding written up with outcome, exploit steps, root cause, impact/severity, and mapping to industry frameworks (OWASP LLM Top 10, MITRE ATLAS)

## Highlights

**Bad Robot (code injection).** A "Calculate" tool ran submitted expressions through a real Python interpreter rather than a restricted math grammar, filtering input with a substring blocklist. Bypassed the filter using string concatenation (`'o'+'s'` instead of the blocked literal `'os'`) to achieve arbitrary code execution, then extracted a protected file byte-by-byte through a numeric-only output channel using `ord()`.

**The Shape Shifter (insecure deserialization).** A YAML config parser honored Python-specific tags (`!!python/object/apply`), allowing arbitrary module/function calls to be constructed from user input. Chained an anchor/alias pair — reading a file via one call, then feeding the result into a type constructor guaranteed to fail on it — to leak the file's contents through the resulting exception message.

**The Heist (TOCTOU race condition).** A trading agent validated account balance and deducted funds as two separate steps. Issuing two purchase calls in a single parallel batch caused both to be evaluated against the same stale balance before either deduction was committed, allowing a $1000 budget to fund $2000 in purchases.

**The Octopus (multi-agent privilege escalation).** A low-privilege file-upload tool was used to forge an internal authorization record (`GRANT_ROOT_READ_FOR_SESSION`), which a separate, higher-privilege tool then accepted as valid — a parser differential between the security filter and the application's own JSON parser.

## Methodology

For each level: enumerate the exposed tool and its input handling → probe for unsafe behavior (unsandboxed execution, missing validation, timing gaps) → construct a minimal payload → **independently verify** execution rather than trusting the tool's self-reported status → document root cause and map to a recognized vulnerability framework.

## Frameworks referenced

- [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- [MITRE ATLAS](https://atlas.mitre.org/)
- Common Weakness Enumeration (CWE) — CWE-95 (Eval Injection), CWE-78 (OS Command Injection), CWE-502 (Deserialization of Untrusted Data), CWE-367/CWE-362 (TOCTOU / Race Condition)

## Full report

The complete findings report, with evidence screenshots and step-by-step reproduction for all six levels, is included in this repo: [`arkx_findings_report.pdf`](./arkx_findings_report.pdf).

---

*Part of an ongoing focus on AI evaluation and AI security — see also my [LLM evaluation framework](#) and [AI data annotation work](#).*
