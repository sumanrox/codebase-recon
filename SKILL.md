---
name: codebase-recon
description: Deep read-only reconnaissance of an unfamiliar codebase, producing an evidence-backed architecture report (structure, execution flows, data/state model, concurrency, security boundaries, invariants, coupling, extension points) before any code is written. Use this whenever the user is taking over, inheriting, onboarding to, or preparing to extend an existing repository — including phrasings like "understand this codebase before we change it", "map the architecture", "do recon on this repo", "what would break if I add X", "I'm inheriting this project", "explain how this system actually works end to end", or any request that asks for a system-wide mental model rather than a single answer. Also use it when a user forbids modifications and asks for analysis first. Do not use it for single-file questions, debugging one failure, or reviewing a diff.
---

# Codebase Recon

Build an accurate mental model of a system as it actually exists, so that later feature work can be reasoned about: where a change belongs, which contracts it touches, what breaks, what the cleanest integration point is.

The value of this skill is not summary. It is **evidence**. A recon report that cannot be verified against `file:line` is a guess wearing a report's clothes, and it will mislead every decision built on it.

## The read-only contract

Recon happens because someone does not yet trust their own understanding enough to change the code. Editing during recon destroys the thing being measured.

Permitted writes: exactly one file — the report at `docs/recon/YYYY-MM-DD-architecture-recon.md` (create the directory if needed; if the repo has a different docs convention, follow it and say so).

Refuse, and say why, if asked mid-recon to: edit source, fix a bug found during recon, refactor, run formatters, `git add`, `git commit`, create branches, or install dependencies. Note the finding in the report instead. Read-only shell commands (`cat`, `grep`, `find`, `git log`, `git show`, `git blame`, dependency tree listings) are all fair game — reading history is reconnaissance, not modification.

If a build or test run would materially resolve an unclear behavior, ask first rather than assume; some repos have side-effecting test suites.

## Phase 0 — Orient yourself (do not delegate this)

Delegating before you understand the shape of the repo produces subagents that search the wrong places. Do this pass first, personally:

1. Repo tree to a useful depth, ignoring vendor/build noise. `git ls-files` beats `find` when the repo is git-tracked — it already excludes ignored files.
2. Dependency manifests and lockfiles; note runtime, language versions, frameworks.
3. All documentation paths (README, `docs/`, ADRs, `CONTRIBUTING`, design notes, inline architecture comments).
4. Entry points: `main`, CLI definitions, server bootstrap, worker/daemon starts, job registration, `Dockerfile`/`docker-compose` commands, CI entry commands, package `scripts`/`entry_points`.
5. Config and environment handling — the surface where behavior changes without code changing.
6. Rough size: file count and LOC per top-level directory. This decides the dispatch strategy below.

Announce what kind of system this is and how you intend to partition it before dispatching anything.

## Phase 1 — Partition and dispatch

Repos routinely exceed one context window. Parallel read-only agents solve that, but only if each has a bounded subsystem and returns findings rather than file contents — a subagent that pastes source back has just moved the context problem, not solved it.

Partition by subsystem, not by directory count: an API layer, a persistence layer, a worker/queue system, an auth module, an integration client set, the CLI, infrastructure/CI. Aim for 3–7 agents; more than that and synthesis quality drops faster than coverage rises.

Dispatch briefs live in `references/subagent-briefs.md` — read it before spawning, and give every agent the read-only constraint plus the evidence format. Use `Explore` for search-shaped mapping and `general-purpose` where an agent must run tools like `git log` or a dependency query. Spawn them in one turn so they run concurrently.

Small repos (roughly under ~150 source files) are usually faster and more accurate read directly. Say which mode you chose and why.

## Phase 2 — Own the cross-cutting passes yourself

Some conclusions cannot be assembled from independent reports, because they are precisely the things that span subsystem boundaries. Verify these first-hand, reading the actual implementation:

- **Invariants and state machines** — which transitions are legal, and whether that is enforced by code, by a database constraint, or only by convention. This distinction is the single highest-value output of recon; conventions break silently when new code arrives.
- **Concurrency and lifecycle** — worker ownership, lease/lock semantics, retries, idempotency, crash recovery, terminal states, transaction boundaries. Comments about concurrency are frequently stale. Read the implementation.
- **Security boundaries** — trust boundaries, authn/authz enforcement points, secrets handling, deserialization, command/process execution, user-controlled data reaching sinks, egress and filesystem access, resource limits.
- **Error propagation and shutdown** — what happens to in-flight work when something fails or the process stops.
- **Contradictions** — documentation vs implementation, tests vs implementation, config defaults vs actual behavior, comments vs code. Report the discrepancy; do not silently pick a winner.

`references/inspection-domains.md` holds the full probe checklists for every domain, including data model, tests, and history. Read the sections you are working on rather than the whole file at once.

## Phase 3 — Synthesize and write

Use the structure in `references/report-template.md` (17 sections). Write the full report to the report path, and give the same report in chat — the file survives the context window, the chat version is what the user actually reads now.

Three disciplines make the report trustworthy:

**Evidence.** Every significant claim carries `path/to/file.ext:123` or a symbol name. If you cannot point at it, you cannot claim it.

**Epistemic labels.** Mark each substantive conclusion as **Verified** (read it), **Interpretation** (inferred from structure — say from what), or **Unclear** (state exactly what evidence is missing and where it would live). Inflating interpretation into fact is the most damaging failure mode available here, because the user will build on it.

**Coverage ledger.** End with what you did *not* read, and why. Large repos will have such a list, and saying so is a feature — it tells the user where the remaining risk sits. Claiming complete coverage you did not achieve poisons every downstream decision.

Trace flows end to end rather than describing files: *request enters at `routes/order.ts:44` → validated by `schema/order.ts:12` → state transformed in `services/order.ts:88` → persisted via `repo/order.ts:31` → enqueues `jobs/fulfil.ts:19` → response shaped at `routes/order.ts:71`*. A per-file inventory is not architecture; the seams between files are.

## Phase 4 — Stop

The terminal state is the delivered report. Do not propose refactors, write implementation code, or chain into an implementation skill. Recon exists to inform a decision the user has not made yet; pre-empting it wastes the work.

Close by naming the two or three areas most dangerous to modify without further study, and wait.

## References

| File | Read when |
|------|-----------|
| `references/inspection-domains.md` | Working a specific domain (data model, concurrency, security, tests, history) and needing the probe checklist |
| `references/report-template.md` | Writing the final report |
| `references/subagent-briefs.md` | Before spawning parallel agents |
| `references/standalone-prompt.md` | The user wants the recon prompt as pasteable text for another tool or session |
