---
type: spec
topic: research-team
date: 2026-06-06
status: active
tags: [dev/tooling, research, multi-agent]
---

# Research Team — design spec

A generalist, reusable, parallel multi-agent research kit. Point it at any topic
via a freeform prompt; it dispatches parallel specialist agents, then synthesizes
their findings into a single polished, dated research document in the Obsidian
vault.

Generalization of the topic-specific kit at
`~/Documents/01-Dev/01-investment-advisor-app/research-team/`.

---

## 1. Goals & non-goals

### Goals
- One generalist interface — give it different tasks (Polymarket bots, trading
  strategies, free APIs, anything researchable).
- Freeform prompt invocation with an approval gate before spawning agents.
- Multiple specialist agents running **in parallel**, then 1 lead synthesizing.
- Output = a single polished `.md` file in a vault research zone, dated, with
  frontmatter, ranked choices (pros/cons), evidence, risks, implementation
  overview, sources.
- Compounds value over time; cheap to run for small lookups, scalable for big
  investigations.

### Non-goals
- Not an automated trading system. Decision-support research only.
- Not a code project / library (no persistent committed scripts in v1 — see §6).
- Does not touch the vault except to write the one final output doc.
- Does not manage tasks / daily notes / existing vault frontmatter.

---

## 2. Architecture — orchestrator-worker

Follows the Anthropic multi-agent research playbook (lead decomposes → parallel
specialist workers with isolated context → lead synthesizes). Validated against
the existing investment-advisor kit and Anthropic's engineering write-up.

### Roles

| Role | File | Model | Mode | Job |
|------|------|-------|------|-----|
| Lead | `agents/00-lead.md` | Opus | orchestrate + write | Expand prompt → brief → dispatch → synthesize → write doc |
| Scout | `agents/01-scout.md` | Sonnet | read-only (web + repo) | Broad enumeration of candidates/options; candidate matrix + shortlist |
| Deep Analyst | `agents/02-deep-analyst.md` | Opus | read-only | Pick shortlist apart: math, code, repos, methodology, claimed results |
| Empiricist | `agents/03-empiricist.md` | Opus | sandboxed code (`_scratch/`) | Actually run curl/python/gh to verify testable claims; evidence log |
| Devil's Advocate | `agents/04-devils-advocate.md` | Opus | read-only | Attack everything: bad assumptions, missed candidates, gotchas, longevity risk |

### Flow

```
freeform prompt
  → Lead reads team.md + output-template.md + brief-template.md
  → Lead drafts structured brief from prompt
  → [APPROVAL GATE] show brief to user; user says `go` or edits
  → Lead saves brief to briefs/YYYY-MM-DD-HHMM-<slug>.md
  → Lead dispatches N workers in parallel (one message, N Agent calls)
  → workers return reports as text (never write the final doc)
  → Lead synthesizes via 00-lead.md rules + output-template.md skeleton
  → Lead writes single dated doc to $VAULT/08-research/<filename>.md
  → Lead reports path + 1-line verdict to user
```

### Approval gate (key UX)
Lead pauses after drafting the brief, shows it terse, waits for `go`. Rationale:
multi-agent runs burn ~15× chat tokens; freeform prompts are ambiguous; a ~2k-token
gate prevents a ~60k-token wrong-direction run. User can edit any field
(`must_test`, `effort`, scope) at the gate; Lead re-shows until `go`.

---

## 3. Scaling knob (effort)

The brief carries `effort: light | standard | heavy`. Lead auto-suggests from
prompt complexity; user overrides at the gate. Effort-scaling is an explicit
Anthropic best practice (1 agent for fact-find, 2–4 for compare, 10+ for deep).

| Effort | Agents | Roster | When |
|--------|--------|--------|------|
| light | 2 | Scout + Devil's Advocate | Single-thing lookup ("best free RSS reader macOS") |
| standard | 4 | Scout + Deep Analyst + Empiricist + Devil's Advocate | Default. Both example tasks. |
| heavy | 6–8 | Lead **forks** Scout + Deep Analyst across sub-domains | 5+ angles ("social, repos, quants, math, results") |

**Fork mechanic:** heavy is NOT new role files. Lead instantiates the *same*
Scout/Analyst role files multiple times, writing **disjoint scope** into each
agent's brief so they don't overlap (avoids the vague-brief → duplicate-work
failure mode). Example heavy split for "trading strategies":
- Scout-A: social media / influencer strategies
- Scout-B: GitHub repos / open-source quant libs
- Deep Analyst-A: math / formula validation
- Deep Analyst-B: backtest results / claimed-PnL credibility
- Empiricist: run testable code
- Devil's Advocate: attack all

---

## 4. Repo layout

Kit lives at `~/Documents/01-Dev/research-team/` as its own git repo (reusable
from any session/dir; keeps the vault pure knowledge).

```
research-team/
├── README.md                 # how to use, examples, conventions, future-work notes
├── team.md                   # orchestration spec the Lead reads every run
├── output-template.md        # 6-section skeleton with frontmatter
├── brief-template.md         # structured brief shape Lead expands prompts into
├── .gitignore                # ignore _scratch/, _out/
├── docs/specs/               # this spec + future design docs
├── agents/
│   ├── 00-lead.md
│   ├── 01-scout.md
│   ├── 02-deep-analyst.md
│   ├── 03-empiricist.md
│   └── 04-devils-advocate.md
├── briefs/                   # auto-saved expanded briefs per run (audit trail)
│   └── YYYY-MM-DD-HHMM-<slug>.md
├── _scratch/                 # Empiricist sandbox (gitignored)
└── _out/                     # optional local copy of outputs (gitignored, default off)
```

No `topics/` folder (old kit had one) — briefs are auto-generated from freeform
prompts, not hand-authored. `briefs/` keeps the expanded brief as an audit trail
("what was actually researched" vs the freeform prompt) for cheap reproducibility.

---

## 5. Output document

### Filename
`$VAULT/08-research/YYYY-MM-DD-HHMM-<slug>.md`
- 24h time, **no colon** (`:` breaks on some sync targets) → `HHMM`.
- Example: `2026-06-06-1430-polymarket-bots-strategies.md`
- `<slug>` = kebab-case of the brief `topic` field.

### Frontmatter
```yaml
---
type: research
topic: polymarket-bots-strategies
question: "Which Polymarket bots/strategies are documented, profitable, copyable?"
date: 2026-06-06
researched: 2026-06-06 14:30
verdict: "Go with copy-trading top 3 wallets; build-your-own = No-Go for now"
recommended: "Wallet-mirror strategy via Polymarket CLOB API"
effort: standard
status: active
tags: [research/trading, research/crypto, polymarket]
sources_count: 14
---
```

### Six sections
1. **Verdict & Recommendation** — `> [!tip]` callout 1-line verdict; why winner
   wins, why runners-up lost, single deciding factor.
2. **Top Choices (ranked)** — each candidate as H3 with **Pros** / **Cons** /
   **Evidence of results** / **Fit** bullets. Ranked best → worst.
3. **Evidence** — what the Empiricist actually ran/saw. Table: claim · method ·
   result · `WORKS/PARTIAL/FAILS/UNVERIFIED`. Trimmed real payloads (keys redacted).
4. **Risks & Gotchas** — Devil's Advocate output. Risk table: risk · severity ·
   likelihood · mitigation. Plus a "what I'd worry about most" paragraph.
5. **Implementation Overview** — if adopting the pick: phased steps + effort
   (S/M/L). Skipped when verdict = No-Go.
6. **Sources** — every URL + access date. `UNVERIFIED` flagged inline throughout.

Obsidian niceties: verdict callout, wikilinks to related vault notes where the
Lead spots them, `#research/<domain>` tags routing into Dataview.

---

## 6. Sandbox & safety (Empiricist)

The Empiricist runs real code. Hard walls so it never touches repos/vault/secrets.

- Writes **only** under `research-team/_scratch/`. Never tracked files, never the
  vault, never other repos.
- May run: `curl`, `python3` (one-liners + throwaway scripts), `gh` (read-only
  API), `git clone` into `_scratch/` to inspect a repo.
- Install isolation: `venv` / `pipx` / `--user`. Never mutate a project env.
- **No secrets invented.** Tests only keyless/free public endpoints, or a key the
  user pasted into the session. Redacts any key in its report.
- Respects rate limits — a few sample calls, not a hammer.
- `_scratch/` is gitignored + safe to wipe; Lead may clear it between runs.

### Crypto / wallet-specific (matters for Polymarket-type tasks)
- **Read-only chain queries only.** Inspect public wallets / PnL via public APIs
  + block explorers. Never sign, never send a tx, never touch a private key.
- If a brief implies "test the strategy live," the Empiricist reports it as
  out-of-scope `UNVERIFIED` rather than executing.
- No wallet connections, no trade-scope API keys.

### Anti-footgun
- Network calls are fine (it's research). Destructive shell outside `_scratch/`
  (`rm -rf`, writes outside the sandbox) is forbidden by the role prompt.
- Anything in `must_test` the Empiricist can't safely run → returned
  `UNVERIFIED` with a reason, never faked.

### Trust boundary
Kit lives outside the vault, so a misbehaving agent writing to `_scratch/` can't
corrupt vault knowledge. Only the Lead writes to the vault, and only the one
final doc.

### Future work (NOT in v1 — YAGNI)
A curated `tools/` folder of reusable probes, **human-promoted only** (agent
proposes in its report → user reviews + commits → next run's Empiricist reads
`tools/` before writing new throwaway probes). Deferred until reuse actually
shows up. v1 is `_scratch/`-only. README documents the promotion path so it
exists when wanted.

---

## 7. Vault integration

### New zone
`08-research/` at vault root:
```
08-research/
├── index.md      # domain map: purpose, frontmatter contract, Dataview listing
└── YYYY-MM-DD-HHMM-<slug>.md
```

### Frontmatter contract
- `type: research` is **new** → register it in the vault `CLAUDE.md` note-types
  list (currently daily, log, person, recipe, project, permanent, todo, index,
  rules, …).
- `status: active` (allowed: active/paused/completed/archived). Flips `archived`
  when stale/superseded. Never deleted → move to `99-archive/08-research/`.

### Tags & Dataview
- `tags: [research/<domain>, …]` matches the existing `#domain/subcategory`
  convention.
- Research stays **out** of task views (`tasks.md`, daily-note queries) — research
  ≠ tasks, separate concern.
- `08-research/index.md` carries its own Dataview listing all research docs by
  date + domain.

### Cross-linking
Lead wikilinks each output to related vault notes when spotted (honors the
"new note links to ≥1 existing note" rule).

### Untouched
Daily notes (Obsidian-managed), existing frontmatter keys, the task pipeline.

---

## 8. Worked examples

### Example A — Polymarket bots
```
Run research-team: polymarket bots, strategies, rank them, check repos, check
wallets/accounts on polymarket, check results, structured summary
```
- Effort: standard (4 agents).
- Scout: enumerate known bots, strategies, public repos, cited profitable wallets.
- Deep Analyst: strategy logic, repo quality, claimed-vs-plausible returns.
- Empiricist: hit Polymarket CLOB API for sample market data; verify 2–3 cited
  wallet PnLs on-chain (read-only); clone top repo, inspect logic.
- Devil's Advocate: survivorship bias in "profitable" wallets, ToS, API
  longevity, edge decay.
- Output: `08-research/2026-06-06-HHMM-polymarket-bots-strategies.md`.

### Example B — Trading strategies
```
Run research-team: trading strategies on social media, repos, quants, formulas,
math, results, structured summary
```
- Effort: heavy (6–8 agents) — 5 angles named.
- Forked Scouts (social vs repos), forked Analysts (math vs results), Empiricist,
  Devil's Advocate.
- Output: `08-research/2026-06-06-HHMM-trading-strategies.md`.

---

## 9. Open items for implementation plan
- Exact `$VAULT` resolution (env var vs hardcoded vault path in `team.md`).
- `index.md` Dataview query for `08-research/`.
- Wording of each agent role file (generalize from old kit's API-shaped prompts).
- README quick-start + invocation phrasing + effort-override syntax.
- One vault `CLAUDE.md` edit to register `type: research` + the `08-research/`
  zone in the structure list.
