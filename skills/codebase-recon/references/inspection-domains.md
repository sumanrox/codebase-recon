# Inspection domains — probe checklists

Read only the sections you are actively working. Each section lists what to look for and, where it matters, how to tell a real finding from a plausible-sounding one.

## Contents
1. Repository surface
2. Documentation
3. Architecture and execution flows
4. Data and state model
5. Concurrency and lifecycle
6. Security boundaries
7. Tests
8. Architectural seams
9. Dependency and responsibility map
10. Invariants and contracts
11. Contradictions
12. History

---

## 1. Repository surface

Inspect, and follow imports to confirm what is actually reachable rather than trusting directory names:

top-level layout · source dirs · tests · configuration · scripts · docs · database schema and migrations · CI/CD · container and orchestration files · build config · dependency manifests and lockfiles · environment handling · generated files that are architecturally load-bearing · CLI entry points · services, workers, daemons · APIs, routes, controllers · internal libraries and packages · storage and persistence · authn/authz · logging and telemetry · error handling · external integrations · feature flags · background jobs and queues · caching · network boundaries · security components.

A directory that looks obvious (`utils/`, `common/`, `legacy/`) is exactly where load-bearing logic hides. Dead code is common too — if nothing imports it, say so; that is useful.

## 2. Documentation

Read documentation for content, not for filenames. Extract: project purpose · architectural principles · design decisions and their rationale · intended execution model · operational assumptions · security model · data model · stated invariants · explicit non-goals · known limitations · planned work · development workflow · testing philosophy · deployment model.

Documentation drifts. Where it disagrees with the code, record both and mark the discrepancy — do not quietly prefer one.

## 3. Architecture and execution flows

Map: entry points · bootstrap and initialization order · main execution paths · module responsibilities · inter-module dependencies · data flow · control flow · persistence flow · network flow · async execution · worker lifecycle · error propagation · retry behavior · shutdown and cleanup.

For each major path, trace it across layers with file and line evidence at each hop. The unit of understanding is the flow, not the file.

Initialization order deserves specific attention: what must exist before what, which globals or singletons get constructed, and what happens if that order is disturbed. New features break here often.

## 4. Data and state model

For each important entity, model, schema, or state object: where defined · who creates it · who mutates it · who consumes it · lifecycle · important fields · relationships · persistence representation · serialization and deserialization · validation · state transitions · invariants — and, crucially, whether each invariant is enforced by code, by a database constraint, or only by convention.

State machines deserve an explicit transition table. Note transitions that are reachable in code but not documented, and documented transitions that no code performs.

## 5. Concurrency and lifecycle

Where the system has workers, jobs, async execution, locks, leases, queues, schedulers, subprocesses, threads, or distributed execution, determine: ownership of work · worker lifecycle · lease and lock semantics · retry semantics · idempotency · race protections · crash recovery · partial-failure behavior · terminal states · cleanup · transaction boundaries · exactly-once vs at-least-once assumptions.

Verify from implementation. Comments and docstrings about concurrency are the least reliable text in most repositories, because the code changed under them and the tests did not fail. Look specifically for: a lease that is taken but never renewed, a retry that is not idempotent, a transaction that closes before a side effect fires, and a terminal state that some path can leave.

## 6. Security boundaries

Identify: trust boundaries · input boundaries · privilege boundaries · authentication · authorization · secrets handling · network boundaries · filesystem access · process and command execution · user-controlled data flow · SSRF-shaped boundaries · serialization boundaries · injection-sensitive operations · isolation mechanisms · sandboxing · resource limits · egress controls · sensitive data handling.

This is reconnaissance, not a pentest and not a fix list. Describe where the boundary is, what enforces it, and where enforcement is inconsistent across call paths — an authorization check present on three routes and absent on a fourth is an architecture finding, and it is exactly the kind of thing a new feature will replicate incorrectly.

## 7. Tests

Determine what the project treats as contractual: unit vs integration vs system tests · important fixtures · helpers · mocking strategy · setup and teardown · coverage of critical paths · invariants under test · untested areas · tests that encode architectural assumptions.

Counting tests says nothing. What matters is which behaviors are pinned — those are the contracts a future change must not break — and which critical paths have no pin at all, because those will break silently.

Tests that mock a boundary heavily are telling you where the seams are. Tests that construct elaborate fixtures are telling you where the coupling is.

## 8. Architectural seams

Look for where future work will touch the system: stable interfaces · public vs internal APIs · extension points · tight coupling · hidden coupling · god modules · duplicated logic · cross-layer leakage · circular dependencies · global state · implicit contracts · fragile assumptions · places where adding a feature would force edits across many files.

Unconventional is not the same as wrong. Only call something a problem when you can state the concrete consequence: "adding a third payment provider requires editing five files across three layers because the provider list is duplicated at `a.ts:12`, `b.ts:88`, ...".

## 9. Dependency and responsibility map

Produce, per component: responsibility → dependencies → dependents → important state → entry points.

Then name the highest-level layers and how they communicate. Use terminology that fits the system; borrow from control plane / data plane / persistence / execution / interface / infrastructure / isolation / observability only where those words genuinely describe what is there.

## 10. Invariants and contracts

Find the assumptions that must hold for the system to work: state transition rules · ownership rules · ordering requirements · content and hash integrity assumptions · idempotency assumptions · database consistency requirements · worker and lease invariants · filesystem invariants · network isolation requirements · API contracts · configuration invariants.

Separate them into three buckets and keep the buckets visible in the report: **documented**, **enforced in code or schema**, and **held only by convention**. The third bucket is where future changes break things, because nothing will stop the change from violating it.

## 11. Contradictions

Compare documentation vs implementation · tests vs implementation · configuration vs actual behavior · comments vs implementation · intended architecture vs what accumulated.

Report only contradictions that would change a decision. Padding the report with trivial staleness makes the real ones harder to see.

## 12. History

Where git history is available, use it selectively to explain unusual decisions: recent major architectural changes · refactors · security hardening · bug fixes that introduced an invariant · reverted approaches · files with repeated fixes (`git log --format=%H -- <path> | wc -l` across hot files) · TODOs that represent deliberate future work.

`git log -S'<symbol>'` finds when a behavior appeared. `git blame` on a strange guard clause usually explains it in one commit message, and that guard is frequently an undocumented invariant.

Never rewrite history and never create commits.
