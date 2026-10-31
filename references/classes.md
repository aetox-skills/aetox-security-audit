# Attack classes: what one hunter looks for

Each `##` block below is one class, written to be pasted whole into a
hunter brief (`loop.md` §6). A unit names its class as `classes.md#<the
heading>`. Pick the classes that fit what recon found; add one of your own
when the system has a class none of these name.

When recon finds one of these boundaries, the domain file's blocks go into
the brief too:

- a model, agent, tools, memory or MCP: `agentic.md`
- a desktop app, a webview bridge, deep links, a local socket or pipe, an
  updater or a privileged helper: `desktop-ipc.md`
- work from outside that can use up shared CPU, memory, disk, workers,
  queues, quotas or money: `exhaustion.md`
- several users or tenants in one store, caches, search, exports,
  backups, deletion: `isolation.md`

`audit.md` holds grep leads per category; paste the rows that fit a unit.

## Injection

Trace input from outside to a sink that interprets it. The sink depends on
the system: a query, HTML, a shell or process argument, a template, a file
path, a redirect, a deserializer, a log line, a regular expression, an
LDAP or XPath query, a header.

Do not stop at the direct paths. Look for a value stored safely and later
read back into a dangerous context by other code. Look for injection
through keys, field names, headers and metadata, not only values. Look for
injection into the systems around the app: logs, caches, search indexes,
analytics.

## Access control

Check that a caller cannot do what its authority does not cover. Not
whether a check exists: whether it checks the right permission, on the
right resource, by the right means.

- Is there another path to the same state change that checks a weaker
  permission?
- Can a field in the request body override what the permission meant to
  restrict?
- Is there a route that requires a login and then never asks whether this
  user may touch this object?
- Does one resource have several access paths with different checks?
- Do bulk, batch, export and import check each item?

## Resource and file handling

- A path from outside reaching read, write, delete or serve, through `..`,
  encoded sequences, symlinks, absolute paths or Windows drive and UNC
  forms.
- A URL from outside that the server fetches, including after a redirect,
  through DNS that changes, and through two URL parsers that disagree.
- Unsafe deserialization, archive extraction that writes outside its
  target, temporary files with guessable names.
- A file checked, then opened by name again (check-then-use).

## Crypto and secrets

- Weak randomness for tokens, keys, nonces and ids that guard something.
- Secrets in source, logs, error messages, URLs, or responses the client
  sees.
- Broken key derivation, a MAC never checked, a reused nonce, ECB, a fixed
  IV, encryption with no integrity.
- A secret compared with `==` where timing shows how much matched.
- What happens when a crypto call fails: does the error path fall back to
  no crypto?

## Business logic

Scanners cannot find these, and they are often the costly ones. For each
main workflow:

- Can a step be skipped, repeated, done backwards, or replayed after it
  finished? If step 2 of 3 fails, is step 1 undone?
- Two requests at once: double spend, double approve, a lost update.
  Look for check-then-act that is not atomic.
- Negative, zero, huge, rounded or string-typed quantities.
- A restriction on one operation that another operation reaching the same
  effect does not apply.
- Data trusted because "it was validated on the way in", when another
  path could have written it.
- Expiry, schedules, rate windows, clock skew, time zones between
  components, the exact boundary moment.
- What the system does when config is missing, a flag is off, a
  dependency is down, or a migration is half done.

## Feature abuse and data leakage

A feature used for what it was not meant for. Look for the flaw in the
design as well as the code.

- Export or backup that includes data above the requester's access,
  other users' data, deleted or draft content.
- Import or restore that overwrites, skips validation, or writes where the
  requester cannot.
- Search, filter or sort that reveals what the requester cannot open,
  through results, counts or ordering.
- Different answers for "does not exist" and "no access": the message, the
  status, the time, the size. User enumeration through reset, invite or
  sign-up.
- Preview or share links that unlock more than one item, drafts reachable
  through search or listings, private content cached as public.
- A callback, webhook or notification URL the user sets and the server
  fetches.

## Chained flaws

Behaviour that is allowed or contained on its own can become a flaw when
another part relies on a stronger promise.

- Map what a low-trust principal can read, write, call and keep, then
  connect only concrete outputs to later trust decisions. Confirm each
  step; do not assume the next one.
- Component A checks a value and hands it to B. Compare exactly what A
  guarantees with what B assumes: truncation, type, normalization, tenant.
- A value safe where it was stored becomes dangerous later: a field name
  used as a path, escaped text rendered raw, a stored string used as a
  URL, a pattern or a policy.
- A token, key, plugin or tool scope that grows after delegation,
  refresh, caching or a role change.
- Setup, migration, soft delete, revoke and cache expiry: is stale state
  accepted in the window?
- Undelete, restore, rollback and cancel: do they apply today's owner and
  rules?

## Wildcard

You are not given a class. Find what the other classes miss, with the
same bar: a concrete boundary and a result.

- The strangest code in the project: why does it exist, and what happens
  if it is abused?
- Features that look half finished, experimental or bolted on.
- Calls the API accepts that the UI never makes.
- Routes, parameters, headers or flags in the source that no doc names.
- Features combined that were not designed together.
- Git history: a reverted security fix, a commented-out check, a secret
  committed then removed.
- Actions one user takes that change other users, shared state, or what
  the operator pays.
- Irreversible actions: is approval bound to the final principal, action
  and resource?
- What the code assumes about its environment: the clock, DNS, a local
  database, a case-sensitive filesystem.
- What the tests do not test.

When something looks strange, read it until you can say whether it is
safe. When a comment says why something is safe, check the reason. Code
named `temp`, `hack` or `legacy` gets a close read.

## Obvious things

Others hunt the subtle flaws. This class checks the plain exposures
everyone assumes someone else checked. Be literal and check every item.

- Passwords, keys, tokens or private keys in source (`password`,
  `secret`, `apikey`, `token`, `Bearer`, `-----BEGIN`).
- TODO, FIXME, HACK or XXX that mention auth, validation or permission.
- Debug or dev mode that can be switched on in production by an
  environment variable, a query parameter or a header.
- Test, example or seed credentials that would work in production.
- `/debug`, `/admin`, `/test`, `/metrics`, `/env`, `/config` or `pprof`
  reachable without a login, or on a listener bound beyond localhost.
- `.env`, `credentials.json`, `*.pem`, `*.key` committed; a `.gitignore`
  that misses secrets, uploads or local config.
- Unpinned dependencies, a lockfile missing or unused in CI.
- `eval`, `exec`, `child_process`, `Function()`, dynamic import with input.
- CORS `*`, or any origin reflected, with credentials allowed.
- Cookies that carry a session without `HttpOnly`, `Secure` or `SameSite`.
- Open redirects through `redirect`, `return`, `next`, `url`, `goto`.
- Plain HTTP where TLS was meant; stack traces, paths or SQL errors in
  responses.

A flag is not a finding. Trace each one: a cookie without `HttpOnly` that
holds nothing sensitive, or that the page must read, is advice; an error
that names a field that never holds anything sensitive is advice.
