# Subagent briefs

Parallel agents pay off only when each returns *findings with evidence* rather than file contents. An agent that pastes source back has relocated the context problem instead of solving it. Say so in the brief — subagents follow an explicit output contract far more reliably than an implied one.

Spawn all agents in a single turn so they run concurrently. Use `Explore` for search-and-map work; use `general-purpose` when the agent must run tools such as `git log`, dependency queries, or schema dumps.

## Brief template

Fill the bracketed parts and send verbatim:

```
Read-only reconnaissance of the [SUBSYSTEM] subsystem in [REPO PATH].

Do not modify, create, or delete any file. Do not run git write commands, formatters,
installers, or builds. Reading (cat/grep/find/git log/git blame/git show) is expected.

Scope: [DIRS AND FILES]. Follow imports outward when they explain this subsystem's
behavior, but do not map neighbouring subsystems in depth — other agents own those.

Determine:
- Entry points into this subsystem and who calls them.
- Its responsibilities, and what it explicitly does not do.
- Major execution paths traced across layers, with file:line at every hop.
- State it owns, creates, mutates, or consumes; where that state is defined and persisted.
- Its dependencies (what it imports) and dependents (what imports it).
- Invariants it relies on, and for each whether it is enforced by code, by a database
  constraint, or only by convention. This distinction matters more than the invariant itself.
- Error handling, retries, and failure behavior.
- Anything security-relevant: authz checks, user-controlled data reaching a sink, secrets,
  process execution, deserialization, network egress.
- Contradictions between comments, docs, tests, and the implementation.

Return, in under [N] words:
1. Findings as terse claims, each carrying file:line or a symbol name.
2. Each claim labeled Verified (you read it), Interpretation (inferred — say from what), or
   Unclear (say what evidence is missing and where it would live).
3. The interfaces this subsystem exposes and consumes, since the main agent needs these to
   stitch subsystems together.
4. A coverage ledger: what you read fully, partially, and not at all, and why.

Do not paste file contents. Do not summarize files one by one — the seams between files are
what matters. Do not propose fixes or refactors; this is reconnaissance only.
```

## Partitioning guidance

Partition by subsystem rather than by directory count — 3 to 7 agents. Past roughly seven, synthesis quality falls faster than coverage rises, because the main agent spends its context reconciling overlapping reports.

Typical partitions: interface layer (API/CLI/UI) · domain and services · persistence and migrations · async execution (workers, queues, schedulers) · auth and security · external integrations · infrastructure, CI, and deployment.

Give overlapping boundaries to exactly one agent and say so in both briefs ("`auth/` is owned by another agent; note the call sites and move on"), otherwise two agents produce two partial and subtly conflicting accounts of the same code.

## After the agents return

Reconcile before writing. Where two agents disagree about a boundary, read that code yourself — a disagreement at a seam usually means the seam is genuinely ambiguous in the code, which is itself a finding worth reporting.

Never promote a subagent's Interpretation to Verified in the final report. If it matters enough to state as fact, read it yourself.
