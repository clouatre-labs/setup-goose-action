# Security Examples

Choose based on your threat model:

## Tier 1: Maximum Security
**File:** [tier1-maximum-security.yml](tier1-maximum-security.yml)

AI analyzes tool output only. Immune to prompt injection.

**Use for:** Public repos, external contributors

---

## Tier 2: Balanced
**File:** [tier2-balanced-security.yml](tier2-balanced-security.yml)

AI sees file stats. Manual approval for posting.

**Use for:** Private repos, trusted contributors

---

## Tier 3: Advanced
**File:** [tier3-advanced-patterns.yml](tier3-advanced-patterns.yml)

AI analyzes diffs. Vulnerable to prompt injection.

**Use for:** Private repos, 100% trusted team only

---

## Quick Comparison

| Feature | Tier 1 | Tier 2 | Tier 3 |
|---------|--------|--------|--------|
| Input | Tool output | File stats | Full diff |
| Prompt Injection Risk | None | Low | High |
| Use Case | Default | Balanced | Advanced |
