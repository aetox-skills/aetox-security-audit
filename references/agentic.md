# Agentic questions: apps with an agent, tools or MCP servers

## When to use this file

A language model takes part in a decision that matters for trust: an
assistant, retrieval over documents, memory that persists, a loop that
calls tools, MCP servers or clients, code that builds prompts from outside
input, or code that acts on model output. The flow to follow is content
from outside → the model or its memory → a capability, an authority or a
sink. Read alongside `audit.md` and `classes.md`: transport, access
control, queries, files and rendering are still ordinary boundaries.

## Core discipline

```text
- Prompt injection alone is not a finding. A finding needs a failure in
  code: content reaches another principal's context, uses authority the
  requester lacks, discloses data they cannot read, or drives a sink they
  cannot reach directly.
- Model output, memory, tool descriptions and MCP responses are input
  from outside. Point to the code that grants authority, trusts the
  output, writes durable state, or feeds a sink.
- An instruction in the system prompt is not a security boundary. Only
  deterministic checks, authorization per resource, isolation, binding
  and narrowed credentials count.
- Name the attacker, the principal hurt, the identity the action runs as,
  the resource, the exact action, the authority used and what is seen. A
  user asking the model to use the user's own authority is not a
  delegation flaw because a model carried it out.
- Authorization and action binding are separate. Content from an attacker
  that makes the agent act under the user's valid authority, on something
  the user did not ask for or approve, is an action-binding failure.
```

## The questions (OWASP Top 10 for Agentic Applications 2026)

| ID | Name | The question |
|---|---|---|
| ASI01 | Agent goal hijack | Can content the model reads (a tool result, a fetched page, a file, an email, a comment) change what it does next? Is that content marked as data when it is put back into the prompt? Is there any place where instructions found in content are followed without the user? |
| ASI02 | Tool misuse | Can a tool be called with arguments outside what its approval covered: a `read` that reads a secret store, a `write` outside the project, a `shell` that runs something the user did not see? Is the approval prompt shown the real arguments? Does a tool schema accept aliases, extra fields or coercions the handler reads differently from the check? |
| ASI03 | Identity and privilege abuse | Does a subagent, a worker or a scheduled job carry the parent's full rights? Does a tool run under a broad service credential without checking the requester may touch that resource? Can a tool filter be widened from inside a session? Is an API key usable for more than the one provider it was given for? |
| ASI04 | Agentic supply chain | Is an MCP server, a skill or a plugin installed from a name alone, with no pin, no hash and no reading of what it declares? Does an installed skill get to rewrite other skills? Is MCP metadata (a tool description, a schema) trusted as policy? |
| ASI05 | Unexpected code execution | Is there a road from model output to `eval`, a shell, or a file that is then executed, that skips the gate every other tool passes through? A calculator that can reach the filesystem is this. |
| ASI06 | Memory and context poisoning | Can content the model read end up in a memory file, a persona file or a skill that every later session loads? Who approves the write, and is the source shown? Are history, embeddings or prompt caches keyed so one session or tenant reads another's? |
| ASI07 | Insecure inter-agent communication | Do agents pass each other text that is then treated as instructions? Is a remote engine socket authenticated, and bound to localhost or a key? Can one MCP server's reply satisfy another server's pending call, or two servers claim the same tool name? |
| ASI08 | Cascading failures | Can one bad tool result make a loop (retry, re-plan, re-delegate) that never stops, or that widens what it tries? Where is the ceiling, and who pays for the loop? |
| ASI09 | Human trust exploitation | Can the model make an approval prompt look safer than it is: a summary that hides the dangerous argument, an action taken as "read" that writes? Does the user see the argument, not the model's description of it? Does an approval bind the exact tool, arguments, target and expiry, so a retry or a resumed session cannot reuse it for a changed call? |
| ASI10 | Rogue agents | Is there a kill switch a person can reach while a run is going? Does a background job, a scheduled task or a desktop companion keep rights after the session that made it is gone? |

## Universal moves

- Draw four maps first: each identity actions run as, each capability,
  every context or memory source someone can write, and each place output
  goes. Then connect the principal at the start to the authority at the
  end.
- Start at tools with side effects and work back through the dispatcher,
  schema, confirmation, model context, retrieval and ingestion. Start at
  memory reads and trace every writer.
- Compare direct, queued, retried, resumed, batched and delegated paths
  for the same action. The strongest gate applies after the arguments are
  final and before every side effect.

## Validation rules

1. Name the boundary crossed and what is seen: attacker, principal hurt,
   identity used, target, and the unauthorized or unrequested action or
   disclosure.
2. Confused deputy: show the tool lacks a check of requester and resource,
   and that the attacker cannot do the same through the normal product.
   Action binding: show attacker content caused an action under the
   user's authority that the user did not request or approve.
3. Memory or retrieval: cite both the attacker's write and the later read
   by another principal or a privileged decision.
4. MCP identity: check the authenticated connection, request
   correlation, tool namespace and credential. External server identity
   or routing that decides it is needs_validation.

Source: OWASP Top 10 for Agentic Applications 2026 (ASI01 to ASI10),
phrased as the question to ask of an app like this one; the discipline,
moves and rules adapted from cloudflare/security-audit-skill
`AI-AND-LLM.md` (MIT).
