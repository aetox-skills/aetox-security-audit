# The full loop: run folder, ledger, briefs, records

Read this before step 1 of a full audit. Small audits never need it.

## 1. Run folder

- `.aetox/security/run-<N>/` under the project root, `<N>` the next unused
  number. Before the first write, check git ignores it
  (`git check-ignore -q .aetox/security/probe`). When it does not, write
  `.aetox/security/.gitignore` holding one line, `*`, so no report
  reaches a commit.
- Only you write in it. Helpers only read.
- Files, in the order they are written:
  - `run.json`: `run`, `started`, `commit` (`git rev-parse HEAD`),
    `dirty` (true when the worktree has changes; those files are read as
    they are now), `profile`, `scope` (paths, or `"all"`), `prior` (the
    run folder read, or null), `helpers` (true whenever `task` is in
    your tools; false only when it is not), `status` (`in_progress`,
    `complete`, or `incomplete` with `reason`).
  - `threat-model.md`: the threat model as the user corrected it, the
    map's starting paths, the boundaries and the strongest control on
    each, under about 800 words. Every hunter brief carries it whole.
  - `ledger.json`: the units.
  - `findings.json`: the records.
  - `REPORT.md`: first written right after the ledger, then rewritten
    as the run goes (§11).

## 2. Earlier runs

Read the latest earlier run's `ledger.json` and `findings.json` before
cutting units. Compare by source, not by run date: `git diff
<prior commit> -- <paths>` over each unit's paths, plus anything dirty.

- A `confirmed` record whose paths did not change is carried: same
  fingerprint, linked to a planned unit, left off the hunters' list, and
  still sent to one fresh verifier this run.
- A `confirmed` record whose paths changed becomes a planned unit to
  check again. It is not on any exclusion list.
- `needs_validation`, `deferred`, `blocked` and `out_of_scope` units, and
  any unit whose source changed, are current work.
- A `rejected` record with unchanged source stops only that same claim,
  never the unit around it.
- A light or scoped earlier run says nothing about what it did not reach.

No earlier run: say so in the report.

## 3. Units and the ledger

A unit is one entry surface × the boundary that guards it × one attack
class. The classes are the blocks in `classes.md`, plus the domain files
when recon found their boundary: `agentic.md`, `desktop-ipc.md`,
`exhaustion.md`, `isolation.md`. Choose a domain file because the
boundary its "When to use" names is present, never because a library's
name appears.

`ledger.json` is an array, one object per unit:

```json
{
  "id": "internal/files/serve.go#Serve::internal/auth/mw.go#RequireUser::classes.md#Resource and file handling",
  "surface": "GET /files/{name}",
  "boundary": "RequireUser, then the root check in serve.go",
  "class": "classes.md#Resource and file handling",
  "blocks": ["classes.md#Resource and file handling"],
  "skipped": [{"block": "exhaustion.md", "reason": "files are served from disk, nothing expands"}],
  "paths": ["internal/files/serve.go", "internal/auth/mw.go"],
  "prior": "new",
  "wave": 1,
  "status": "planned",
  "by": null,
  "read": [],
  "fingerprints": [],
  "unresolved": []
}
```

- `id` is built from source references (a file and the route or symbol),
  never from line numbers, wave or verdict, so the same unit has the same
  id next run. Sort by id.
- `status` and what it needs:

| status | `by` | `read` | `fingerprints` | `unresolved` |
|---|---|---|---|---|
| `planned` | null | empty | empty | empty |
| `in_progress` | task id | empty | empty | empty |
| `covered` | task id | the files read | empty | empty |
| `candidate` | task id | the files read | one or more | optional |
| `blocked` | task id | the files read | empty | the fact that stopped it |
| `deferred`, `out_of_scope` | null | empty | empty | the reason |

The ledger is the coverage claim. "Auth reviewed" in prose is not
coverage; a unit closed with the files its helper read is.

## 4. Profiles and the count

- `light`: units coarsened to surface × class; one hunter wave, one
  critic, one verifier per candidate. The critic's gaps are written as
  `deferred`, not hunted. A small project, a re-run, a first look.
- `standard`: hunter waves until a critic adds nothing. The default.
- `deep`: units split per subsystem and per lifecycle (create, update,
  delete, restore, migrate); after the critic adds nothing, one more
  fresh critic must also add nothing; carried records are hunted again.

Profiles change breadth, never the bar: every profile keeps the gates,
the verifier for every candidate and the safety rules.

The count you say before wave 1: hunters (one per unit; one hunter may
own up to three units of the same subsystem and boundary), one critic per
wave, and verifiers, about one per expected candidate (reserve a third of
the total for them). When the user set a cap, it is never passed in
silence: when it would be, stop hunting, verify what was found in
fingerprint order, mark the rest `deferred` and the run `incomplete`.

Order the units: no-login surfaces before logged-in ones; boundaries over
credentials, other tenants' data, code execution or release before the
rest; changed source and open items from the last run before re-passes;
ties by id.

## 5. Starting helpers

- `task` `action: plan` first, phases named for the waves (`Hunt 1`,
  `Critic 1`, `Verify`), then `start` with `agent: "explore"` and a
  `phase` on every start.
- Start the whole wave, then `collect` its ids together. Four run at once;
  the rest wait their turn.
- Before a unit is handed out, set it `in_progress` with the task id.
- A reply that is not the JSON asked for is not repaired: the unit goes
  back to `planned` and the next wave gives it to a fresh helper.

## 6. The hunter brief

Fill the `<...>` parts. The method and gate sections go across unchanged.

```text
You are hunting for security flaws in one part of the project at <root>.
Your goal is to find places where a lower-trust caller gets past a control
meant to stop it, with source evidence for each. For this job your whole
report is one JSON object in the shape at the end of this brief, with no
prose around it; it replaces the path list you usually return.

## The system
<threat-model.md, whole>

## Your units
<for each unit: id, surface, boundary, class, starting paths>
Owned by other helpers, do not cover: <their unit ids>
Already known, do not report again: <fingerprint and title, one per line>

## What to look for
<each block in the unit's `blocks`, pasted whole: the class block from
classes.md, and from a domain file its Core discipline, the chosen class
sections and its Validation rules>
Not yours this time: <each skipped block and its reason>

## Method
READ THE CODE AT DEPTH. Follow each input through parsing, identity,
authorization, normalization, storage, the copies made of it, and the
final sink. Read the sibling, legacy, batch, retry, cancel, migration and
error paths that reach the same effect. Where two paths guard the same
thing, check the guards are equal, not only that both exist. Compare what
one component promises with what the next one assumes.

WORK FROM A CONCRETE INVARIANT:
1. Name the lower-trust principal and what it can already do.
2. Name the value, action, state change or resource selector it sends.
3. Find the control that should refuse, bind, isolate, limit or revoke it.
4. Trace the exact path after that decision.
5. Stop at the smallest effect: a wrong return value, another dummy
   user's record, a request let through that should be refused.
6. Say the source change that enforces the invariant, and the test that
   would catch it.

DEPTH BOUND. Trace only paths that reach your units' boundaries or that
those boundaries rely on. Stop a line as soon as the invariant is settled
either way, write it down, and move on.

TEST SAD PATHS AND DISAGREEMENTS. Absent, empty, zero, negative, maximum,
over the limit, duplicated, mixed encoding, stale, revoked, out of order,
concurrent, half-migrated, a dependency failing, a rollback: wherever the
interface accepts them. Compare how each parser and each policy reads the
same value at every handoff.

VARIANTS. When a High or Critical candidate shows a root cause that could
repeat, search your own units for the same sink reached the same way.
Each variant needs its own conditions. A variant outside your units goes
in "uncovered".

YOU READ, YOU DO NOT RUN. You have no shell. When only running code would
settle a claim, return it as needs_validation with the exact input and the
result that would settle it; the main agent runs it.

A fact outside the repository (deploy config, proxy, provider, OS policy,
packaging) is not proof either way. When it decides the claim, return
needs_validation naming that fact.

## Candidate gate
1. A candidate has a full source trace with file:line from entry to sink,
   and names the strongest control on the path.
2. It names the lower-trust principal, what they send, the control that
   should stop it, the boundary crossed, and who or what is hurt.
3. Do not grow a crash into code execution, ordinary work into an outage,
   or a user's own action on their own data into a privilege gain.
4. A missing best practice with nobody hurt goes in "hardening", not in
   candidates. A claim the source disproves is dropped, not
   needs_validation.
5. Use one fingerprint per root cause: the bug class and the sink, like
   `path-traversal:internal/files/serve.go#Serve`. No line numbers in it.
6. Return "candidates": [] when nothing passes this gate. That is a
   complete answer.

## Report
{
  "units": [
    {"id": "<one of your unit ids>",
     "status": "covered | candidate | blocked",
     "read": ["path/you/read.go"],
     "checked": "the invariant you checked and what the source showed",
     "fingerprints": [],
     "unresolved": []}
  ],
  "candidates": [
    {"fingerprint": "...", "proposed": "confirmed | needs_validation",
     "title": "...", "principal": "...", "control": "...", "boundary": "...",
     "trace": [{"step": "entry | through | sink", "at": "file:line", "what": "..."}],
     "result": "what the principal gets",
     "severity": "critical | high | medium | low (confirmed only)",
     "fix": "...",
     "missing": "the fact outside the source (needs_validation only)",
     "how_to_check": "the input and the result that settles it (needs_validation only)"}
  ],
  "hardening": ["advice, with nobody hurt yet"],
  "uncovered": [{"surface": "...", "boundary": "...", "class": "...", "paths": ["..."], "reason": "..."}]
}
Every unit id you were given appears once in "units".
```

Read each reply: a unit maps to exactly one id you assigned, `covered`
and `candidate` have `read`, `blocked` has `unresolved`. Update only those
units. Keep `hardening` lines for the report. `uncovered` entries become
new units for the critic to weigh.

## 7. The critic brief

```text
You check whether a security audit missed anything. You do not look for
bugs; you look for parts of the project nobody covered. For this job your
whole report is one JSON object in the shape at the end, with no prose
around it.

## The system
<threat-model.md, whole>

## The ledger
<ledger.json, whole>

## What to check
- Entry points in the source that no unit names.
- A second path to the same effect that no unit traced.
- Create, update, delete, restore or migrate paths missing for a boundary
  that has them.
- A domain file whose boundary is in the source but in no unit's blocks,
  or a skipped block whose reason the source contradicts.
- A unit marked covered whose "read" list could not have reached its
  boundary.
- Open items from the last run that no unit carries.

## Report
{"missing": [{"surface": "...", "boundary": "...", "class": "...",
              "paths": ["..."], "reason": "the source that shows the gap"}],
 "reassign": ["unit id whose result does not hold up, with the reason"],
 "stop": false}
"stop" is true only when "missing" and "reassign" are both empty.
```

Add the accepted gaps as planned units. A reassigned unit keeps its old
result in an `attempts` list and goes to a fresh helper. In `light`, both
become `deferred` with reason `light profile`.

## 8. The verifier brief

One verifier per candidate. Group up to three candidates from the same
subsystem in one brief only when there are more than eight candidates.

```text
You did not write this candidate. Try to refute it from the project's
source at <root>. For this job your whole report is one JSON object in
the shape at the end, with no prose around it.

## The system
<the threat model lines this path needs>

## The candidate
<the candidate JSON: claim, trace, result, proposed severity>

## The rules for this kind of flaw
<the Validation rules of its domain file, or its class block>

## Steps
1. Open every file:line in the trace. Is the first a real entry a
   lower-trust principal can reach, and the last the claimed sink?
2. Find the strongest control on the path: the project's own guard, a
   framework default, a type, a config in the repository. Look on the
   sibling paths too.
3. Check five things: the boundary (a lower-trust principal, a control, a
   crossing, someone hurt), reach, flow, guard, impact.
4. A proposed severity must match only what the evidence shows.
5. For needs_validation: if the source answers the missing fact, decide
   it. Keep needs_validation only when the fact is truly outside the
   source, and make the check exact and harmless.
6. Keep the fingerprint unless the root cause is a different one.

## Report
{"decision": "confirmed | needs_validation | rejected",
 "record": <the candidate, corrected; for rejected add "reason": the check
 that failed and the line that shows it>}
Your corrected record replaces the hunter's wording.
```

Give the verifier the candidate only. Never the hunter's reasoning, never
another verifier's decision.

## 9. Running the check

For each confirmed Critical or High, you run the smallest check that
shows the result, under SKILL.md's safety rules: an existing test, or a
small harness on a copy with dummy data. Record the command, the input and
what came back. When nothing can run, the record stays `confirmed` only
if the source alone settles it, with `"proof": "source"`; when a runtime
fact decides it, it becomes `needs_validation`.

## 10. findings.json

An array sorted by fingerprint. Every record has `fingerprint`, `verdict`,
`title`, `principal`, `boundary`, `trace`, and paths relative to the
project root.

- `confirmed` adds `result`, `severity`, `proof` (`run` or `source`),
  `check` (command, input, output) when run, and `fix`.
- `needs_validation` adds `missing` and `how_to_check`. No severity, no
  fix.
- `rejected` adds `reason`.

Before writing `REPORT.md`, read `findings.json` back and check each
record has the fields of its verdict and nothing from another verdict.
Then write the report from the records and the ledger, in SKILL.md's
order. The run ends one of two ways: `status: complete` with every
candidate decided, or `status: incomplete` with the reason, and the report
says which units are left.

## 11. The report as the run goes

A turn can end before step 8: the provider refuses a request, the
connection drops, the user presses Stop. Whatever is on disk then is
what the user gets, so the report is never left for the end.

- **After the ledger:** write `REPORT.md` with this first line,
  `Status: incomplete (in progress)`, then the scope, the threat model's
  boundaries and the unit table, every unit `planned`.
- **After each wave and after the verifications:** write
  `findings.json` with the records decided so far, then rewrite
  `REPORT.md` from it and the ledger: the unit table with each status
  now, the confirmed records, and the open candidates listed by title
  as `not yet verified`. Keep the first line.
- **At the end:** the full report in SKILL.md's order. Set the first
  line and `run.json` together, to `Status: complete` or `Status:
  incomplete: <reason>`.

A record that is not verified yet never appears under confirmed, in any
version of the report.
