# Data isolation and lifecycle

## When to use this file

The target stores data for several users or tenants, keeps copies of it
(cache, search index, analytics, thumbnails, exports, backups), issues
links to objects, migrates schemas, or promises deletion, revocation and
retention. This file follows one record through every copy and every
state change. Endpoint-level permission checks are `classes.md#Access
control`.

## Core discipline

```text
- An owner or tenant field on a record is not isolation. Find the query,
  key, path or policy that enforces it on each read and write path.
- Follow the copies. Data clean in the primary store can leak through
  search, cache, analytics, exports, previews, logs, replicas and backups
  that have other access rules and other lifetimes.
- Deletion and revocation are promises about every copy: current,
  historical, cached, indexed, exported, restored and queued.
- A privacy preference is not a vulnerability by itself. A finding needs
  an explicit access boundary or deletion promise, and an unauthorized
  reader or a later operation that breaks it.
- External storage policy, CDN behaviour, replica lag and backup access
  the repository cannot show make the candidate needs_validation.
```

## Tenant and object isolation

**Owner not enforced.** A read, update, delete, list, count or bulk query
loads an object by id without binding it to the logged-in owner or
tenant, or takes that identity from the request body. Compare direct
lookups, nested relations, background jobs, admin, import and legacy
paths.

**Keys that collide.** Cache keys, object paths, unique constraints,
search document ids, temp files or dedupe keys leave out the tenant, so
two principals overwrite or read the same logical key.

**Policy and query disagree.** Row-level policy, ORM default scopes,
authorization filters and raw or service clients apply different
predicates. Check joins, aggregates, views, `unscoped` or bypass clients,
and paths where the request context is missing.

**Links that reach too far.** Object keys, attachment ids, share links or
signed URLs allow operations or objects beyond what the issuer could
reach, or keep working after the ACL changed. Bind the operation, the
exact object and version, the audience, the expiry and the tenant.

## Copies and disclosure

**Search, cache and index drift.** A record's ACL or lifecycle changes,
but its cached, indexed, embedded, thumbnail, feed or preview copy is not
updated. Filtering must happen when the copy is read, not only when it is
written.

**Logs, analytics and diagnostics as readers.** Private content or
credentials are written to systems with wider access, longer retention or
mixed tenants. Name the data and the realistic reader; a field name or a
public id is not enough.

**Oracles.** Counts, filters, ordering, errors, unique-constraint
failures, timings or notifications reveal a protected record's existence
or state. A finding names the confidential fact and the visible
difference.

## Export, import, backup, migration

**Export that carries too much.** An export, report, snapshot or backup
includes other tenants, hidden fields, deleted data, secrets or history
the requester cannot see. Check per-item authorization after selection,
and who may download the result.

**Import or restore that grants too much.** Restore or import skips
owner, schema, ACL, uniqueness or validation rules, overwrites existing
records, or writes into a tenant the requester cannot. Treat the archive
as untrusted.

**Migration defaults.** Old records lack tenant, ACL or lifecycle fields;
ids from two eras collide; during rollout old and new readers apply
different defaults. Read backfill, dual-write, rollback and resumed
migrations.

## Deletion and revocation

**Soft delete bypassed.** A direct lookup, search, relation, link,
background job or restore ignores the deleted flag. Can a deleted
identifier be registered again while references remain?

**Stale authority.** Removing a member, changing an ACL, withdrawing
consent, revoking a secret or downgrading a role leaves sessions, caches,
subscriptions, jobs or materialized data that still authorize.

**Queued work recreates data.** Deletion finishes in the primary store
while queued jobs, retries, exports or analytics recreate or keep it.

**Restore brings back what policy removed.** Undo, undelete or restore
returns data, credentials, memberships or permissions that today's policy
does not allow.

## Universal moves

- Pick one protected record and draw its write, query, cache, index,
  event, export, backup, delete and restore paths. Mark the principal and
  tenant at every edge.
- Compare two dummy tenants through the same local functions, then again
  after an ACL change, a delete, an account switch and a restore.
- Start where request identity is most often missing: bypass clients,
  background jobs, migrations, global unique keys, cache keys.

## Validation rules

1. Name the lower-trust principal, the protected data or state, its owner
   or tenant, the copy or operation used, and the unauthorized read or
   change.
2. Cite both the intended policy and the path that skips or contradicts
   it, and confirm no other layer enforces the same condition.
3. Prove it with dummy tenants and fixtures; stop at the first record or
   operation that crosses.
4. External cache, storage, replica, analytics, backup or retention
   policy that decides it makes it needs_validation, with the check the
   owner makes.
