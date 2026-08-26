# Codebase Recon

[![skills.sh](https://skills.sh/b/sumanrox/codebase-recon)](https://skills.sh/sumanrox/codebase-recon)

An agent skill for deep, read-only reconnaissance of an unfamiliar codebase. It produces an evidence-backed architecture report — structure, execution flows, data and state model, concurrency, security boundaries, invariants, coupling, extension points — before any code is written.

The value of this skill is not summary. It is **evidence**. A recon report that cannot be verified against `file:line` is a guess wearing a report's clothes, and it will mislead every decision built on it.

## When to use it

Reach for this when you are taking over, inheriting, onboarding to, or preparing to extend a repository you do not yet understand:

- "Map the architecture before we change anything."
- "I'm inheriting this project — explain how it actually works end to end."
- "What would break if I add X?"
- "Analyze this, but do not modify a single file."

It is deliberately the wrong tool for single-file questions, debugging one failure, or reviewing a diff.

## Install

### With the `skills` CLI (any supported agent)

```bash
npx skills add sumanrox/codebase-recon
```

Installs into the current project. Add `-g` for a global install available across all projects:

```bash
npx skills add sumanrox/codebase-recon -g
```

Target one agent explicitly and skip the prompts:

```bash
npx skills add sumanrox/codebase-recon -g -a claude-code -y
```

The CLI supports Claude Code, Codex, Cursor, OpenCode, and 70+ other agents. See [the `skills` docs](https://github.com/vercel-labs/skills) for the full list and for private-repo authentication.

### As a Claude Code plugin

```
/plugin marketplace add sumanrox/codebase-recon
/plugin install codebase-recon@codebase-recon
```

### Manually

Clone the repository and copy or symlink the skill directory into your agent's skills folder — `~/.claude/skills/` for Claude Code globally, or `.claude/skills/` inside a project:

```bash
git clone https://github.com/sumanrox/codebase-recon.git
ln -s "$PWD/codebase-recon/skills/codebase-recon" ~/.claude/skills/codebase-recon
```

### Without installing anything

Run it once against a repository, no install:

```bash
npx skills use sumanrox/codebase-recon@codebase-recon --agent claude-code
```

Or pipe the generated prompt into an agent of your choice:

```bash
npx skills use sumanrox/codebase-recon@codebase-recon | claude
```

## Usage

Once installed, the skill activates on intent — you do not need to name it. Any of these will trigger it:

```
Do recon on this repo before we touch anything.
I'm inheriting this codebase. Map the architecture.
Explain how this system actually works end to end.
What would break if I added a second payment provider?
```

To invoke it explicitly in Claude Code:

```
/codebase-recon
```

The skill runs four phases:

| Phase | What happens |
|-------|--------------|
| **0 — Orient** | The main agent personally maps the tree, manifests, docs, entry points, config, and size. Never delegated — delegating before you know the shape of a repo produces subagents that search the wrong places. |
| **1 — Partition** | Repos too large for one context window are split across 3–7 read-only subagents, one per subsystem, spawned concurrently. Small repos (roughly under 150 source files) are read directly instead. |
| **2 — Cross-cutting passes** | The main agent verifies first-hand what cannot be assembled from independent reports: invariants and state machines, concurrency and lifecycle, security boundaries, error propagation and shutdown, and contradictions between docs, tests, config, and code. |
| **3 — Synthesize** | A 17-section report, written to `docs/recon/YYYY-MM-DD-architecture-recon.md` and delivered in chat. |
| **4 — Stop** | The terminal state is the report. No refactor proposals, no implementation, no chaining into a build workflow. |

## What you get

A report covering executive understanding, repository structure, system architecture, components, execution flows traced end to end, data and state model, concurrency and lifecycle, security architecture, testing architecture, a dependency and coupling map, invariants and contracts, documentation-vs-code discrepancies, strengths, weaknesses, extension points, areas requiring caution, and a narrative mental model — closing with a coverage ledger.

Three disciplines make it trustworthy:

- **Evidence.** Every significant claim carries `path/to/file.ext:123` or a symbol name. If you cannot point at it, you cannot claim it.
- **Epistemic labels.** Each substantive conclusion is marked **Verified** (read it), **Interpretation** (inferred — from what), or **Unclear** (what evidence is missing and where it would live). Inflating interpretation into fact is the most damaging failure mode available, because you will build on it.
- **Coverage ledger.** What was *not* read, and why. Claimed coverage that was not achieved poisons every downstream decision.

Flows are traced across seams rather than described file by file:

> request enters at `routes/order.ts:44` → validated by `schema/order.ts:12` → state transformed in `services/order.ts:88` → persisted via `repo/order.ts:31` → enqueues `jobs/fulfil.ts:19` → response shaped at `routes/order.ts:71`

A per-file inventory is not architecture. The seams between files are.

## The read-only contract

Recon happens because someone does not yet trust their own understanding enough to change the code. Editing during recon destroys the thing being measured.

The skill permits exactly one write: the report file. It refuses, and says why, if asked mid-recon to edit source, fix a bug it found, refactor, run formatters, `git add`, `git commit`, create branches, or install dependencies — the finding goes in the report instead. Read-only shell commands (`cat`, `grep`, `find`, `git log`, `git show`, `git blame`, dependency listings) are all fair game; reading history is reconnaissance, not modification.

If a build or test run would materially resolve an unclear behavior, it asks first — some repos have side-effecting test suites.

## Repository layout

```
skills/codebase-recon/
├── SKILL.md                          # workflow: phases, contracts, disciplines
└── references/
    ├── inspection-domains.md         # 12 probe checklists, loaded per domain
    ├── report-template.md            # the 17-section report structure
    ├── subagent-briefs.md            # verbatim dispatch brief + partitioning rules
    └── standalone-prompt.md          # the whole recon as one pasteable prompt
```

Reference files are loaded on demand rather than up front, so a session that never spawns subagents never pays for the dispatch brief.

## Using it outside an agent that supports skills

`skills/codebase-recon/references/standalone-prompt.md` contains the entire workflow as a single pasteable prompt for any agentic tool with filesystem and shell access. Replace `<REPO PATH>` before sending.

That prompt grants real system access. Review its scope locks, forbidden actions, and stop conditions, and confirm the path matches the intended project, before you paste it.

## License

MIT — see [LICENSE](LICENSE).
