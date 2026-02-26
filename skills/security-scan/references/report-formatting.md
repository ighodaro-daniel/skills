# Report Formatting

## Disclaimer

Open the report with this disclaimer, verbatim:

> This review was performed by an AI assistant. AI analysis can never verify the complete absence of vulnerabilities and no guarantee of security is given. Follow-up reviews are strongly recommended.

---

## Severity Classification

| Severity     | Criteria                                                                                                                                                                            |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CRITICAL** | Direct theft or permanent loss/freeze of user or protocol funds; full protocol takeover; governance capture that gives an attacker unilateral control.                              |
| **HIGH**     | Significant financial loss through realistic attack paths; temporary freeze of user funds; theft of unclaimed yield or rewards; loss of core protocol functionality.                |
| **MEDIUM**   | Limited or conditional financial impact requiring specific preconditions; DoS or griefing that causes disruption without direct profit; protocol misbehavior under edge conditions. |
| **LOW**      | No direct financial risk; best-practice violations, code-quality issues, or incorrect behavior in edge cases that degrade correctness or gas efficiency but leave user assets safe. |

Do not report INFO findings.

---

## Output Format

```
# Security Review

> This review was performed by an AI assistant. AI analysis can never verify the complete absence of vulnerabilities and no guarantee of security is given. Team security reviews, bug bounty programs, and on-chain monitoring are strongly recommended.

## Findings

| # | Severity | Title |
|---|---|---|
| 1 | CRITICAL | <title> |
| 2 | HIGH | <title> |
| 3 | MEDIUM | <title> |
| ... | ... | ... |

---

### 1. [CRITICAL, N] <Title>

**Location:** ContractName.functionName (line N)
**Description:** <vector name — what is wrong and why it matters — what an attacker can do>
**Mitigation:** <concrete recommendation using text and inline code references, no fenced code blocks>

### 2. [HIGH, N] <Title>
...

## Scope
<files reviewed, mode, confidence threshold in effect>
False positives skipped: N (entry, ...)
Below confidence threshold: N
```

**Rules:**

- Order findings Critical first, then High, Medium, Low within the table and the detail section.
- Number findings sequentially; the number in the table matches the heading number in the detail section.
- Omit severity levels that have no findings.
- The disclaimer is always printed, even when there are no findings.
- Confidence score goes inside the severity brackets: `[HIGH, 91]`, `[MEDIUM, 78]`, etc.
- Do not use fenced code blocks in Mitigation. Use prose with inline code references (backtick identifiers, line numbers, short snippets inline).
