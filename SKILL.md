---
name: aetox-security-audit
before: a security review the user asked for: checking an existing change, folder or project for vulnerabilities, or whether it is safe to ship
desks: coding
area: code
description: When the user asks to audit, review or check existing code for security, or asks whether it is safe to ship. Small asks (a diff, a few files, one question) are done by the main agent; a whole-project audit sends read-only helpers to hunt and fresh helpers to refute what they found, and writes a report the next run builds on. Not while building - writing new code safely is aetox-security.
source: cloudflare/security-audit-skill (MIT) for the two sizes, the boundary-and-result gate, coverage units, hunter and verifier briefs, finder never verifier, needs_validation without severity, carrying prior runs, the severity anchors and the domain files (rewritten, cut and fitted to the task tool and read-only helpers); anthropics/claude-code-security-review (MIT) for the exclusion list; trailofbits/skills differential-review, fp-check and variant-analysis (CC BY-SA 4.0, method only, no text taken) for blame on removed guards, the refute gates and the variant search; OpenAI Codex Security for the threat model first; OWASP Top 10:2025, CWE Top 25 and OWASP Top 10 for Agentic Applications 2026 for the category lists
license: MIT
copyright: Copyright (c) 2026 Aetox Skills; portions adapted from cloudflare/security-audit-skill, Copyright (c) 2025-2026 Cloudflare, Inc. (LICENSE)
---

# Aetox Security Audit

Find vulnerabilities that cross a real trust boundary in code that already
exists, and give the owner the evidence, the priority and the smallest
fix. A candidate with no lower-trust principal, no boundary it crosses and
no result someone could observe is not a finding.

Writing or building something, with nobody asking for a review: this skill
is not yours. Close it and follow `aetox-security`.

## Which size

- **Small**, the default: a diff, a PR, a commit, a few files, "is this
  safe", or one finding to triage. You do it yourself and answer in the
  chat. No run folder, no fan-out.
- **Full**: the user asks to audit or pentest the project or the system,
  asks for a full or end-to-end security review, or asks for the report as
  a file. Run the loop below and write the run folder.

When the request could be either, do the small size, and end the answer
with one line offering the full audit and what it would cover.

## Safety, both sizes

- Reading source is read-only. The audit never edits the project's source;
  fixes are proposed.
- The project's code runs only to settle a claim: its own tests, or a
  small harness on a copy, with dummy users and dummy data, nothing
  leaving the machine. Never a deployed endpoint, a live or shared
  service, a real account or credential, a paid API, or load against
  anything running. Dependencies are not installed or fetched for a
  check; a scanner is installed only when the user says yes
  (`references/scanners.md`).
- Stop at the smallest result that settles the claim: a wrong return
  value, another dummy user's record, a refused request that was let
  through. Nothing past it.
- A fact the repository cannot show (a proxy, provider settings, deploy
  config, OS policy, how it is packaged) is neither present nor absent.
  It is `needs_validation`, with the exact fact and how the owner checks
  it.

## The threat model, first

Before hunting, write the threat model in about ten lines and show it:
what the system does, whom it trusts, its entry points (HTTP handlers, CLI
arguments, files read, messages consumed, tool calls the model can make,
uploads, local IPC, deep links), where outside data is first checked, its
privileged operations (shell, filesystem writes, network, database, money,
credentials), and where secrets are stored, loaded and logged. The user
can correct it; the hunt follows it.

Learn how the project already guards itself: its auth middleware, its
validation layer, its escaping, its secret store. A finding is a place
that skips the project's own guard, or the guard missing everywhere.

## Small: one change, or one question

**One change** (a diff, PR, commit): read the diff for anything that
touches auth, secrets, crypto, validation, a path, a shell, a query, the
network, or a permission check. Then one hop out: who calls the changed
code, and what reaches each new input. Use `codebase` with `action: impact`
for changed symbols before opening callers or tests. For every guard the
diff removes or loosens, `git log -S` on the removed line; a removed check
is often the fix for a bug that is about to come back. A refactor is high
risk until the callers show it changed nothing.

**A few files or a question**: walk each path from an entry point in the
threat model to a privileged operation. Read ranges around the evidence;
use `codebase` with `action: trace` when a path crosses generated, RPC or
frontend/backend code. The questions per category, with grep leads, are
`references/audit.md`; an app with an agent, tools or MCP servers is also
read against `references/agentic.md`.

- Trace, do not pattern-match. `exec` in a file is not a finding; input
  from outside reaching `exec` with no boundary between them is.
- Run what is there: the tests on the touched code, and a scanner if one
  is installed. A scanner's line is a lead; the code is read before it
  becomes a finding.
- Each Critical or High candidate goes through the gates below once,
  argued against, before it is reported.

## Full: the loop

The whole project, hunted by read-only helpers and checked by fresh ones.
Every step, brief and file shape is in `references/loop.md`; read it
before step 1.

Helpers are what make this size full. When `task` is in your tools,
helpers are on: the hunts in step 4, the critic in step 5 and the
verifications in step 6 are `task` starts, not reads of your own. A
whole-project audit you walk alone while `task` is there is a small audit,
and its report says so on its first line.

1. **Run folder.** `.aetox/security/run-<N>/` in the project, kept out of
   git. Read the latest earlier run first: what it confirmed, what it
   left open, which commit it read.
2. **Recon.** The threat model above, then `codebase` `action: map`. Cut
   the system into **units**: one entry surface × the boundary that
   guards it × the attack class that applies. Write them to `ledger.json`.
   Units whose source did not change since the last run's commit keep
   their result; changed units and open items become current work. Write
   the first `REPORT.md` now, marked incomplete (`loop.md` §11): a run
   that stops early still leaves a report.
3. **Say the plan.** Units, how many hunters, critics and verifiers you
   will start, and the profile (`light`, `standard`, `deep`). On a
   project's first run, ask for the user's yes with `ask_user` and keep
   working in the same turn: never end the turn on the question. When it
   comes back unanswered, go on with the plan as said. A later run says
   the plan and goes on.
4. **Hunt.** Declare the phases with `task` `action: plan`, then start
   `explore` helpers, a wave at a time, each with a hunter brief from
   `loop.md`. Each brief carries the threat model and the class blocks
   for its units pasted whole: the helper cannot open this skill, so a
   block's name alone gives it nothing.
5. **Critic.** After each wave, one fresh helper reads the ledger and the
   map and names units nobody covered. Its gaps are the next wave.
6. **Verify.** Every candidate goes to a fresh helper whose brief begins
   "You did not write this candidate. Try to refute it". It gets the claim
   and the trace, never the hunter's reasoning or another verifier's
   verdict.
7. **Run the check.** You run the decisive check for each confirmed
   Critical or High, under the safety rules above. Helpers have no shell.
8. **Write.** `findings.json`, then `REPORT.md` from it. Prose never
   changes a verdict. The report is rewritten after every wave, not only
   here: this step makes it final.

The task tool's rule "a list is one job" is about repeated work. Here the
split is the method: hunters own different boundaries, and a verifier that
has read the hunter's reasoning is no longer fresh. Up to four helpers run
at once; start a whole wave, then collect it together.

Only when `task` is not in your tools at all (the user switched helpers
off) do you walk the same loop yourself; `run.json` then says
`"helpers": false` and the report's first line says the verification was
not independent.

## The gates a finding passes

A candidate stays a finding only when all five hold:

- **Boundary**: it names the lower-trust principal, what they send or do,
  the control that should stop it, the boundary crossed, and who or what
  is hurt. A user harming only their own data with their own authority
  crosses nothing.
- **Reach**: the principal the threat model names can send the input.
- **Flow**: source to sink with no step that removes the danger.
- **Guard**: the project's guard is absent on this path, not just
  elsewhere.
- **Impact**: the harm claimed follows from this code, not from the
  category's name.

A candidate that fails one moves to "worth a look" with the one it
failed. A finding that holds is then looked for elsewhere: the same sink,
reached the same way, without the guard. Each variant is its own finding.

## Verdicts and severity

- `confirmed`: the whole path traced, the gates passed, and the result
  either observed in a check or settled by source alone.
- `needs_validation`: a specific path blocked on one fact outside the
  source. It names that fact and how to check it, and carries **no
  severity**. It is never a place to park a guess.
- `rejected`: disproved, with the gate that failed and the line that
  shows it. Kept so the next run does not raise it again.

Severity goes on `confirmed` only, and never above what was shown:

- **Critical**: someone unauthenticated gets code execution, the whole
  data store, or any account they choose.
- **High**: an explicit control is fully defeated for an action with real
  consequences: auth bypass, another tenant's data read or written,
  stored script that runs for other users, code execution behind a login,
  a remote stop of a shared service.
- **Medium**: a real boundary crossed, with a small blast radius, unusual
  preconditions, or a narrow set of resources.
- **Low**: internals that are not secret, or an effect that costs the
  attacker more than it gains.

High or Medium: does the result fully defeat the control, or only weaken
it? If you cannot say the concrete damage, it is lower than it feels.

## What is kept

Nothing seen is dropped. A weakness with no path from an input you can
name, a missing header on an internal tool, a slow path with no shared
victim: these go under "worth a look", labelled as advice, with what would
make each one a finding. Availability is a finding only when
`references/exhaustion.md`'s rules hold. A secret in the platform's own
store (keychain, DPAPI, a secrets manager) is not a leak; one in source or
history is. `[redacted]` in what a tool shows you is Aetox hiding a value
shaped like a secret before it reaches you: in a source or config file,
that is a secret in source, reported by its location and kind.

## The shape of a finding

- Evidence: `file:line`, and the path the input walks, from where to
  where.
- Attack: what the principal sends or does, and what they get. One
  concrete scenario, in the target's own interface (a request, a CLI
  call, a file, an IPC message), not a category name.
- Verdict and, when confirmed, severity.
- Fix: the invariant the code must hold, enforced at the last trusted
  decision point, in the project's own idiom, with the test that would
  have caught it. Proposed, not applied, unless the user asks. A fix is
  checked by running the attack that worked and seeing it fail, and a
  normal input still giving what it gave before.

A found secret is named by its location and kind, never quoted.
`None identified` is a complete answer and beats an invented one.

## Report

Ordered for someone about to press a button:

1. One line: **ship** · **fix first** · **needs a deeper audit**, and why.
2. What this run covered: size, profile, scope, the commit read, what
   came from the last run, what was deferred. A light, scoped or
   unfinished run says it is partial.
3. The threat model, as corrected.
4. Critical and High findings, in the shape above.
5. Medium and Low, one line each.
6. Needs validation: the trace, the missing fact, how to check it.
7. Worth a look: advice, labelled as advice.
8. What was read, what was run and what it said, what was not covered.

## Do not

1. Report a checklist deviation as a vulnerability.
2. Report defense-in-depth advice as a finding when no boundary is
   crossed.
3. Guess how the proxy, provider, browser or deployment behaves.
4. Call a user's own authority, used on their own data, a privilege gain.
5. Grow a crash into code execution, or ordinary load into an outage.
6. Accept a helper's prose in place of its JSON.
7. Put severity on `needs_validation`.
8. Write the report before verification, or let it disagree with
   `findings.json`.

## Hand-offs

- Writing the fix: `aetox-security` for the defense, after the user says
  to fix.
- A query's shape, an index or a migration: `aetox-database`; the
  injection question stays here.
- A system with no code yet: not this skill; name the risks in the design
  itself.
