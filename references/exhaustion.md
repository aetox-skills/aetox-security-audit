# Resource exhaustion and availability

## When to use this file

Work from outside (a request, a message, a file, a tenant's data, an
agent's task) can use up CPU, memory, disk, connections, worker slots,
queue capacity, a paid API or model tokens, or can crash or deadlock a
shared process. This file separates an availability flaw from a slow
path. Never check it by loading a live or shared service.

## Core discipline

```text
- A finding needs a path from input to cost, a missing bound that works,
  and harm to another user, a shared service, or money the operator pays.
  Work that only slows the requester's own process is not a finding.
- A missing rate limit alone is not enough. Check size caps on bodies,
  messages and files, concurrency, queues, deadlines, database limits,
  upstream gateways and per-tenant quotas before calling a path unbounded.
- No stress tests. Use the complexity, small growth points in a local
  fixture with hard limits, mocked paid calls, and cancellation tests.
- Say the attacker's cost, the work the service does, whether it
  persists, who shares it, and how it recovers. One small input with a
  superlinear or lasting shared effect differs from plain volume.
- Upstream caps, autoscaling, paid quotas and recovery that live outside
  the repository make the candidate needs_validation.
```

## Work that grows faster than the input

**Superlinear parsing or matching.** A short accepted input drives
regex backtracking, nested parsing, recursive validation, template
expansion, graph walks, or adversarial hash and sort behaviour. Work out
the accepted depth and size, then the growth.

**Expansion.** Compressed, nested, aliased or encoded input expands far
past the size that was checked: archives, images, documents, YAML or XML
aliases. Check the limit after every expansion stage.

**Query and fan-out.** A small request makes broad scans, joins with no
bound, unbounded sort or aggregation, or many downstream calls because
depth, page size or expansion fields have no cap.

## Things that pile up

**Buffers and cardinality.** Bodies, streams, uploads, sessions, unique
cache keys, metric labels, subscriptions or pending jobs accumulate with
no per-item and no total limit. Find the cleanup on disconnect, timeout,
cancel and half-parsed input.

**Leaks.** Malformed or cancelled work skips cleanup of sockets, files,
cursors, timers, child processes or temp files from a shared pool.

**Work after cancel.** A client timeout, disconnect, cancelled job or
failed authorization returns, but database, model, network or worker work
keeps running. Trace the cancel through every layer.

## Quotas, scheduling and spend

**Work before authentication.** Expensive parsing, decompression, crypto,
lookups or outbound calls happen before the login check and before the
first size or rate gate.

**Quota keyed wrong.** Accounting by a value the attacker picks (IP,
route, key prefix, task id) lets one principal escape its budget or spend
another's. Check overflow, races, retries, reconnects and account
switching.

**Operator spend.** One request or one message can start model calls,
paid API calls, sends or tool loops with no budget per request, no step
ceiling and no cancel. The person paying is not the person sending.

**Starvation.** Attacker jobs hold shared locks, workers, pools or event
loop turns past their deadline, or skip the queue's fairness.

## Failure and recovery

**Reachable crash or deadlock.** Input from outside reaches a panic, an
uncaught exception, a fatal assert, a process exit, a lock cycle or an
infinite loop in a shared process. Say whether one worker or the whole
service goes down; a supervisor restart lowers impact, it does not remove
the flaw.

**Retry storms.** Timeouts or failures trigger retries with no jitter,
ceiling, circuit breaker or dedupe, so one bad source makes lasting work.

**Poison record.** One malformed record fails at the head of a shared
queue, startup scan or migration again and again, and blocks everyone
behind it.

**Recovery that repeats the failure.** Restart, restore or fallback
rebuilds unbounded state, ignores today's quotas, or reloads the input
that crashed it.

## Universal moves

- Build an input-to-resource table: the earliest accepted size, work done
  before auth, fan-out downstream, what persists, the shared pool, who
  owns the limit and the cleanup, how it recovers.
- Compare total limits with per-item ones: ten thousand valid tiny items
  can pass a per-message cap and still exhaust the process.
- Check only in an isolated fixture with hard limits and small growth
  points; mock paid calls; stop once the missing bound is visible.

## Validation rules

1. Name the input from outside, the requester's work, the service's
   amplified or retained resource, who shares it, and how it recovers.
   A missing limit with no shared harm is advice.
2. Confirm no upstream, parser, queue, tenant or framework bound in the
   source stops it. Deployed controls nobody can see are
   needs_validation.
3. Superlinear: show the accepted complexity and a small growth curve.
   Leak: show retention repeating after cleanup should have run. Crash:
   show what the process boundary takes down.
4. Rank by low attacker cost, no login, other tenants hurt, lasting
   effect and poor recovery. Never prove it by causing an outage.
