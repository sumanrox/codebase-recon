# Report template

Write to `docs/recon/YYYY-MM-DD-architecture-recon.md` and deliver the same content in chat.

Scale each section to what the system actually warrants — a 40-file CLI does not need a concurrency chapter, and padding one in makes the report less useful, not more. Drop a section with a one-line "not applicable: no background execution in this system" rather than inventing content for it.

Keep `file:line` evidence inline throughout, and keep the **Verified / Interpretation / Unclear** labels attached to substantive claims rather than collected in a footnote — a reader scanning one section should be able to tell what is solid without cross-referencing.

---

```markdown
# Architecture Recon — <project> — <YYYY-MM-DD>

Commit: <sha> · Branch: <name> · Scope of this pass: <what was read>

## 1. Executive understanding
What this system is, what it does, who runs it, and the three things a new contributor most needs to know.

## 2. Repository structure
Layout, what each top-level area holds, what is generated, what is dead.

## 3. System architecture
Layers and how they communicate. A diagram in text or mermaid where it clarifies more than prose.

## 4. Major components and responsibilities
Per component: responsibility, entry points, key files.

## 5. Major execution flows
Each flow traced end to end with evidence at every hop.

## 6. Data and state model
Entities, lifecycles, persistence, validation, state transitions.

## 7. Concurrency and lifecycle model
Ownership, leases, retries, idempotency, crash recovery, terminal states, transaction boundaries.

## 8. Security architecture
Trust boundaries and what enforces each, plus inconsistencies in enforcement.

## 9. Testing architecture
What is pinned as contractual, what is not covered, what the tests reveal about the design.

## 10. Dependency and coupling map
Component → responsibility → dependencies → dependents → state → entry points.

## 11. Invariants and contracts
Three tables: documented · enforced in code or schema · convention only.

## 12. Documentation-vs-code discrepancies
Each with both sides cited.

## 13. Architectural strengths
What is genuinely well-built, and why it works.

## 14. Weaknesses and technical debt
Each with its concrete consequence, not a style objection.

## 15. Likely extension points
Where new work fits cleanly, and what it would touch.

## 16. Areas requiring caution before modification
Ranked. What breaks, how it breaks, and what to read first.

## 17. Mental model
A narrative of how the whole system works, written so someone can hold it in their head.

## Coverage ledger
Read in full · read partially · not read, and why. Open questions with the evidence that would settle each.
```
