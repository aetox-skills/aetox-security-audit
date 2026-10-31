# Install — Aetox Security Audit

No dependencies. This skill is a Markdown protocol — the agent reads it and follows it.

The full size uses Aetox's `task` helpers for the hunters, the critic and the verifiers. Where `task` is not in the tools at all, the same loop is walked by the main agent and the report's first line says the verification was not independent.

## Aetox

Install from the repository URL — a `SKILL.md` at the root is one skill:

```bash
plugin_install("https://github.com/aetox-skills/aetox-security-audit")
```

Once it is on the shelf:

```bash
skill("aetox-security-audit")
```

## OpenCode

Or copy `SKILL.md`, `references/` and `LICENSE` to `~/.config/opencode/skills/aetox-security-audit/`

## Claude Code

Place `SKILL.md`, `references/` and `LICENSE` in your project or `~/.claude/skills/aetox-security-audit/`.

## Other AI Tools

Varies by platform. Keep `SKILL.md` and `references/` together — the skill loads each file by its relative path from `references/`. See the [Aetox Skills catalog](https://github.com/aetox-skills/aetox-skills).

## Verify

```bash
ls SKILL.md references/loop.md references/audit.md references/agentic.md
```
