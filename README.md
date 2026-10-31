# 🔍 Aetox Security Audit

> Vulnerability review of code that already exists: a threat model first, then read-only hunters, a critic that names the units nobody covered, and fresh verifiers that try to refute every finding.

**Aetox Security Audit** is an AI agent skill for code that already exists — a diff about to ship, a few files, or a whole project. A candidate becomes a finding only when it names the lower-trust principal, the input they control, the control that should stop it, the boundary crossed, and who is hurt. Everything else is kept: `needs_validation` names the one fact outside the source that blocks it and carries no severity, and `rejected` keeps the gate that failed so a later run cannot raise it again.

Sources: [cloudflare/security-audit-skill](https://github.com/cloudflare/security-audit-skill) (MIT) for the two sizes, the boundary-and-result gate, coverage units, the hunter and verifier briefs, finder-never-verifier, and the severity anchors · [anthropics/claude-code-security-review](https://github.com/anthropics/claude-code-security-review) (MIT) for the exclusion list · [trailofbits/skills](https://github.com/trailofbits/skills) differential-review, fp-check and variant-analysis (CC BY-SA 4.0, method only, no text taken) · [OWASP Top 10:2025](https://owasp.org/Top10/), [CWE Top 25](https://cwe.mitre.org/top25/) and OWASP Top 10 for Agentic Applications 2026 for the category lists.

## Quick Start

```bash
skill("aetox-security-audit")
```

Or open [`SKILL.md`](SKILL.md).

## Use When

- Someone asks to audit, review or check existing code for security, or asks whether it is safe to ship
- A diff, PR or commit is about to merge and the question is what it introduces, weakens or newly exposes
- A whole project needs an audit whose report the next run builds on
- The app has an agent, tools or MCP servers, so it is read against `references/agentic.md` as well as the code

Not for: writing or building something while nobody asks for a review. That is [aetox-security](https://github.com/aetox-skills/aetox-security) — the two skills are split at the verb, so review rules never leak into the writing pass.

## Two sizes

| Size | When | What runs |
|---|---|---|
| **Small**, the default | a diff, a PR, a commit, a few files, one "is this safe", one finding to triage | the main agent, answered in the chat; no run folder, no fan-out |
| **Full** | "audit the project", an end-to-end review, or the report as a file | read-only `explore` hunters per unit, a critic for the units nobody covered, fresh verifiers, and `.aetox/security/run-<N>/` holding `findings.json` and `REPORT.md` |

A request that could be either gets the small size, and the answer ends with one line offering the full audit and what it would cover.

## Files

| File | Holds |
|---|---|
| `SKILL.md` | the moment, the two sizes, the safety rules, the five gates, the verdicts, the report order |
| `references/loop.md` | hunter, critic and verifier briefs; how units are cut; the ledger and report file shapes |
| `references/audit.md` | the questions per category, with grep leads, for a few files or one question |
| `references/classes.md` | the attack classes one hunter looks for |
| `references/agentic.md` | OWASP Top 10 for Agentic Applications 2026 questions for apps with agents, tools or MCP servers |
| `references/desktop-ipc.md` | local IPC, deep links and renderer-to-native trust boundaries |
| `references/isolation.md` | tenant, environment and sandbox isolation questions |
| `references/exhaustion.md` | when availability is a finding and when it is not |
| `references/scanners.md` | what may be installed and run, and only with the user's yes |

## Measured results

A review skill that cannot beat an unassisted review should say so. On Aetox's own behaviour benchmark (13 Oct 2026, same model with and without the skill, targets chosen so the answer key is known):

| Target | Result |
|---|---|
| OWASP BenchmarkPython, 12 real + 12 decoy, small size | no change in any cell (scorecard TPR − FPR) |
| OWASP NodeGoat, whole project, 16 known flaws | one more finding, for about 2.5× the run time |
| OWASP crAPI, 23k lines, 5 services, 23 known flaws | 15 findings without this skill → 13 with it |

That gate is why this skill is not on Aetox's bundled shelf: it did not pay for its cost, while the write-side skill kept its measured gain. It is published here because the method is the most complete of its kind we have written, and because the full size is expensive enough to be worth re-measuring on another model or a narrower scope before trusting it. The two sizes are the real difference: the small size costs one pass, the full size costs helpers.

## License

MIT for the skill text, © Aetox Skills. Portions are adapted from the projects named above; [`LICENSE`](LICENSE) carries the upstream MIT notice (Copyright (c) 2025-2026 Cloudflare, Inc.). Trail of Bits material contributed method only, with no text taken.

## Related

- [Aetox Security](https://github.com/aetox-skills/aetox-security) — the writing side
- [Aetox Skills catalog](https://github.com/aetox-skills/aetox-skills)
- [Skill Template](https://github.com/aetox-skills/skill-template)
