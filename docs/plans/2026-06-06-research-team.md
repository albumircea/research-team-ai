# Research Team Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

> **Git:** Mircea runs all git. The commit blocks below are **proposed commands for the user**, not agent steps. Agents create/modify files only; never run `git commit`/`git push`.

**Goal:** Build a generalist, reusable, parallel multi-agent research kit that takes a freeform prompt, dispatches parallel specialist agents through an approval gate, and writes one polished dated research doc into the Obsidian vault.

**Architecture:** Orchestrator-worker. A Lead agent expands a freeform prompt into a structured brief (shown to the user for approval), dispatches 2–8 specialist workers in parallel (Scout / Deep Analyst / Empiricist / Devil's Advocate), collects their text reports, and synthesizes a single 6-section markdown document with frontmatter into `08-research/` in the vault.

**Tech Stack:** Markdown prompt files (the kit is prompt-engineering, not code). The only executable surface is the Empiricist's sandboxed `_scratch/` (curl / python3 / gh, read-only). Claude Code `Agent` tool provides parallel dispatch.

**Spec:** `docs/specs/2026-06-06-research-team-design.md`

**Constants (resolved from spec §9 open items):**
- `VAULT` = `/Users/mirceaalbu/Documents/04-Software/ObsidianVaults/brain`
- `KIT` = `/Users/mirceaalbu/Documents/01-Dev/research-team`
- Output zone = `$VAULT/08-research/`

---

## File Structure

| File | Responsibility |
|------|----------------|
| `KIT/.gitignore` | Exclude `_scratch/`, `_out/` from git |
| `KIT/README.md` | How to invoke, examples, conventions, future-work note |
| `KIT/team.md` | Orchestration spec the Lead reads every run (the contract) |
| `KIT/brief-template.md` | Structured brief shape the Lead expands prompts into |
| `KIT/output-template.md` | 6-section output skeleton + frontmatter |
| `KIT/agents/00-lead.md` | Lead role: brief-drafting + synthesis rules |
| `KIT/agents/01-scout.md` | Broad enumeration role (Sonnet, read-only) |
| `KIT/agents/02-deep-analyst.md` | Depth on shortlist (Opus, read-only) |
| `KIT/agents/03-empiricist.md` | Sandboxed verification (Opus, `_scratch/` only) |
| `KIT/agents/04-devils-advocate.md` | Adversarial pass (Opus, read-only) |
| `KIT/briefs/.gitkeep` | Keep the audit-trail dir in git |
| `KIT/_scratch/.gitkeep` | Keep the sandbox dir (contents gitignored) |
| `VAULT/08-research/index.md` | Vault zone map + Dataview listing of research docs |
| `VAULT/CLAUDE.md` | Register `type: research` + `08-research/` zone (edit) |

Each agent file is self-contained: one role, one clear deliverable, no shared mutable state. The Lead is the only writer of the final doc; workers return text only.

---

## Task 1: Repo scaffold (.gitignore + dirs)

**Files:**
- Create: `KIT/.gitignore`
- Create: `KIT/briefs/.gitkeep`
- Create: `KIT/_scratch/.gitkeep`

- [ ] **Step 1: Create `.gitignore`**

```gitignore
# Empiricist sandbox — throwaway, never tracked
_scratch/*
!_scratch/.gitkeep

# Optional local copies of outputs
_out/

# OS noise
.DS_Store
```

- [ ] **Step 2: Create dir keepers**

`KIT/briefs/.gitkeep` → empty file.
`KIT/_scratch/.gitkeep` → empty file.

- [ ] **Step 3: Verify**

Run: `ls -la ~/Documents/01-Dev/research-team && cat ~/Documents/01-Dev/research-team/.gitignore`
Expected: `.gitignore`, `briefs/`, `_scratch/`, `docs/` present; gitignore shows the rules above.

---

## Task 2: brief-template.md

**Files:**
- Create: `KIT/brief-template.md`

- [ ] **Step 1: Write the file**

````markdown
# Brief template

The Lead expands a freeform prompt into this structure, shows it at the approval
gate, then (on `go`) saves a filled copy to `briefs/YYYY-MM-DD-HHMM-<slug>.md`.

```yaml
topic: <kebab-case-slug>           # drives the output filename slug
question: >                        # the decision the research must answer
  <one or two sentences>
must_test:                         # concrete things to verify empirically, not just read
  - <claim or probe>
  - <claim or probe>
constraints: >                     # context workers must respect
  <who's asking, decision-support vs execution, licence/ToS limits, dependency limits>
effort: standard                   # light | standard | heavy
roster:                            # filled by Lead from effort; shown for transparency
  - scout
  - deep-analyst
  - empiricist
  - devils-advocate
output_path: $VAULT/08-research/<YYYY-MM-DD-HHMM>-<slug>.md
tags: [research/<domain>, ...]
```

## Effort → roster

| effort | agents | roster |
|--------|--------|--------|
| light | 2 | scout, devils-advocate |
| standard | 4 | scout, deep-analyst, empiricist, devils-advocate |
| heavy | 6–8 | forked scouts + forked deep-analysts (disjoint scope) + empiricist + devils-advocate |

## Heavy fork rule

For `heavy`, the Lead instantiates the SAME scout/deep-analyst role files multiple
times with **disjoint scope** written into each agent's task, so no two agents
cover the same ground. State each fork's scope explicitly in its prompt.
````

- [ ] **Step 2: Verify**

Run: `test -f ~/Documents/01-Dev/research-team/brief-template.md && grep -c "effort" ~/Documents/01-Dev/research-team/brief-template.md`
Expected: file exists; `effort` appears ≥3 times.

---

## Task 3: output-template.md

**Files:**
- Create: `KIT/output-template.md`

- [ ] **Step 1: Write the file**

````markdown
---
type: research
topic: "{{TOPIC}}"
question: "{{QUESTION}}"
date: {{DATE}}
researched: {{DATETIME}}
verdict: "{{VERDICT}}"
recommended: "{{RECOMMENDED}}"
effort: {{EFFORT}}
status: active
tags: [{{TAGS}}]
sources_count: {{N}}
---

# {{TOPIC}} — research

> [!tip] Verdict
> {{ONE-LINE VERDICT}} — {{one-line reason}}.
> Researched {{DATE}}. Data ages fast; re-check before relying on specifics.

## 1. Verdict & recommendation

Why the recommended choice wins. Why the runners-up lost. The single deciding
factor. State it plainly — end with a clear Go / No-Go / Go-with-X.

## 2. Top choices (ranked)

### 1. {{Candidate}} — {{one-line}}
- **Pros:** …
- **Cons:** …
- **Evidence of results:** … (link/cite; mark `UNVERIFIED` if only claimed)
- **Fit:** … (how well it matches the question's constraints)

### 2. {{Candidate}} — {{one-line}}
- **Pros:** …
- **Cons:** …
- **Evidence of results:** …
- **Fit:** …

<!-- repeat, ranked best → worst -->

## 3. Evidence

What was actually executed/observed (Empiricist). Mark anything not run `UNVERIFIED`.

| Claim / probe | Method | Result | Verdict |
|---------------|--------|--------|---------|
| | | | WORKS / PARTIAL / FAILS / UNVERIFIED |

```
<trimmed real payload — secrets redacted>
```

## 4. Risks & gotchas

Devil's Advocate output.

| Risk | Severity | Likelihood | Mitigation |
|------|----------|-----------|------------|
| | High/Med/Low | High/Med/Low | |

**What I'd worry about most:** {{single biggest regret-risk paragraph}}.

## 5. Implementation overview

Only if verdict is not No-Go. Phased steps to adopt the pick.

| Phase | Work | Effort |
|-------|------|--------|
| 1 | | S/M/L |
| 2 | | |

## 6. Sources

- {{URL}} — accessed {{date}}
````

- [ ] **Step 2: Verify**

Run: `grep -E "^## [1-6]\." ~/Documents/01-Dev/research-team/output-template.md`
Expected: all six section headings (`## 1.` … `## 6.`) print.

---

## Task 4: team.md (orchestration spec)

**Files:**
- Create: `KIT/team.md`

- [ ] **Step 1: Write the file**

````markdown
# research-team — orchestration spec

You are the **Lead**. You turn a freeform research prompt into a single polished
document, by coordinating parallel specialist agents. Follow this exactly.

## Constants

- `VAULT` = `/Users/mirceaalbu/Documents/04-Software/ObsidianVaults/brain`
- Output zone = `$VAULT/08-research/`
- Kit root = the directory containing this file.

## Inputs

A freeform prompt, e.g. *"polymarket bots, strategies, rank them, check repos and
wallet results."* Optionally an inline effort override (`research-team light: …`,
`… heavy: …`).

## Procedure

1. **Read** this file, `output-template.md`, and `brief-template.md`.
2. **Draft a brief** from the prompt, filling every field in `brief-template.md`.
   - Generate a kebab-case `slug` from `topic`.
   - Auto-pick `effort`: 1 thing → light; 2–3 angles → standard; 5+ angles → heavy.
     Honor any inline override.
   - Compute `output_path` =
     `$VAULT/08-research/<YYYY-MM-DD-HHMM>-<slug>.md` (24h, no colon).
   - Pick `tags` as `research/<domain>` matching the topic.
3. **APPROVAL GATE.** Show the brief to the user, terse, in one block. Wait.
   - User says `go` → proceed.
   - User edits a field → apply, re-show, wait again. Never dispatch before `go`.
4. **Save** the approved brief to `briefs/<YYYY-MM-DD-HHMM>-<slug>.md`.
5. **Dispatch workers in parallel** — one message, N `Agent` calls, N from the
   effort roster. Each worker prompt = its role file + the brief fields. Use the
   model from the table below.
   - For `heavy`, fork Scout / Deep Analyst with **disjoint scope** per fork;
     write that scope explicitly into each prompt.
   - Workers are read-only EXCEPT the Empiricist, which may write only under
     `_scratch/`.
6. **Collect** the workers' returned text. Workers never write the final doc.
7. **Synthesize** per `agents/00-lead.md`, filling `output-template.md`.
8. **Write** the final doc to `output_path`. Create `$VAULT/08-research/` if
   missing. This is the ONLY file you write in the vault.
9. **Report** to the user: the output path + the one-line verdict.

## Models

| Agent | File | Model | Why |
|-------|------|-------|-----|
| 0 Lead | `agents/00-lead.md` | Opus | decomposition + final judgement |
| 1 Scout | `agents/01-scout.md` | Sonnet | broad cheap enumeration |
| 2 Deep Analyst | `agents/02-deep-analyst.md` | Opus | depth, math, code, methodology |
| 3 Empiricist | `agents/03-empiricist.md` | Opus | runs code, judges real data |
| 4 Devil's Advocate | `agents/04-devils-advocate.md` | Opus | adversarial reasoning |

## Effort → roster

| effort | agents | roster |
|--------|--------|--------|
| light | 2 | Scout + Devil's Advocate |
| standard | 4 | Scout + Deep Analyst + Empiricist + Devil's Advocate |
| heavy | 6–8 | forked Scouts + forked Deep Analysts + Empiricist + Devil's Advocate |

## Rules

- **Test, don't assume.** Any `must_test` item only read about, not executed, is
  flagged `UNVERIFIED` in the final doc.
- **Cite.** Every claim/number carries a source URL + access date.
- **Disagreement is signal.** When the Devil's Advocate contradicts another agent,
  the doc names both positions and your ruling — never silently pick one.
- **Evidence beats assertion.** The Empiricist's executed result outranks any
  doc-reading when they conflict; say so.
- **No secrets.** Workers test only keyless/free tiers, or a key the user pasted
  this session. Never invent or commit a key.
- **Stay in bounds.** The only writes are the Empiricist's `_scratch/`, the saved
  brief under `briefs/`, and your one final doc in the vault. Nothing else changes.
- **Read-only chain access.** For crypto/wallet topics: inspect public data only —
  never sign, send, or touch a private key.
````

- [ ] **Step 2: Verify**

Run: `grep -E "APPROVAL GATE|Test, don't assume|Stay in bounds" ~/Documents/01-Dev/research-team/team.md`
Expected: all three rules present.

---

## Task 5: agents/00-lead.md

**Files:**
- Create: `KIT/agents/00-lead.md`

- [ ] **Step 1: Write the file**

````markdown
# Agent 0 — Lead (synthesis)

**Model:** Opus · runs the brief-drafting step, the approval gate, the dispatch,
and the final synthesis. You are not a stapler — you adjudicate.

## Brief drafting

Expand the freeform prompt into the full `brief-template.md` structure. Be
faithful to intent but make scope explicit. Pick effort by angle count. Show it
at the approval gate; iterate on the user's edits until `go`.

## Synthesis inputs

- Report 1 — Scout: candidate matrix, shortlist, gaps.
- Report 2 — Deep Analyst: depth on the shortlist (math/code/repos/results).
- Report 3 — Empiricist: test log + evidence ← **highest weight on any factual
  claim; it actually ran the calls.**
- Report 4 — Devil's Advocate: rebuttals, worst-case, risk register.
- `output-template.md` — the skeleton to fill.

(For `light` runs only Reports 1 + 4 exist. For `heavy`, multiple Scout/Analyst
reports — merge them, dedup overlap.)

## Synthesis rules

1. **Evidence beats assertion.** Empiricist's executed result wins over
   doc-reading; state the conflict and your ruling.
2. **Surface disagreement.** Every Devil's-Advocate contradiction → name both
   positions + your decision.
3. **`UNVERIFIED` stays visible.** `must_test` items not executed are labelled
   `UNVERIFIED`, never dropped or stated as fact.
4. **Cite.** Carry URLs + access dates through; note stale data.
5. **Decide.** End with a clear Go / No-Go / Go-with-X, not a shrug.
6. **Rank.** Section 2 candidates ordered best → worst, each with Pros / Cons /
   Evidence-of-results / Fit.

## Produce

Fill every section of `output-template.md`:
1. Verdict & recommendation (callout + reasoning + deciding factor).
2. Top choices, ranked.
3. Evidence (Empiricist's log; redacted payloads; `UNVERIFIED` marked).
4. Risks & gotchas (Devil's Advocate table + worst-worry paragraph).
5. Implementation overview (phased, S/M/L) — omit if verdict = No-Go.
6. Sources (all URLs + dates).

Set frontmatter: `verdict`, `recommended`, `tags`, `sources_count`, `researched`
(date + 24h time), `effort`. Add wikilinks to related vault notes where you spot
them. Write to the brief's `output_path`. Then tell the user the path + verdict.
````

- [ ] **Step 2: Verify**

Run: `grep -E "Evidence beats assertion|UNVERIFIED|Go / No-Go" ~/Documents/01-Dev/research-team/agents/00-lead.md`
Expected: all three present.

---

## Task 6: agents/01-scout.md

**Files:**
- Create: `KIT/agents/01-scout.md`

- [ ] **Step 1: Write the file**

````markdown
# Agent 1 — Scout

**Model:** Sonnet · **Mode:** read-only (web search + repo read). No edits.

You discover and catalogue every viable candidate for the topic. Breadth first —
depth is the Deep Analyst's job. You are the team's map-maker.

## Brief

- **Topic:** {{TOPIC}}
- **Question:** {{QUESTION}}
- **Constraints:** {{CONSTRAINTS}}
- **Your scope (heavy runs):** {{FORK_SCOPE — else "full topic"}}

## Tasks

1. **Enumerate candidates.** Find every option that could answer the question —
   obvious names AND the long tail (official sources, open-source repos, scrappy
   tools, communities, papers). Stay within your fork scope if one is set.
2. **Catalogue each** in one matrix row:
   - name + URL
   - what it is / what it provides
   - access model (free / freemium / paid / open-source)
   - maturity & recency signal (last update, stars, activity)
   - first-glance credibility of any results it claims
3. **Shortlist** the 4–7 strongest worth deep analysis, one-line reason each.
   Flag any that look strong but are likely dead ends (abandoned, hype, scammy).
4. **Note gaps.** What does the question need that no candidate seems to cover?

## Deliver (return as text — do NOT write files)

- A **candidate matrix** (markdown table, one row per candidate).
- A **shortlist** of 4–7 with reasons → drives the Deep Analyst + Empiricist.
- A **gaps** paragraph.
- **Sources:** every URL + the date accessed.

Be concrete. "Has good results" is useless without the actual number/claim and
where it came from. If you can't confirm a figure, say `unconfirmed`.
````

- [ ] **Step 2: Verify**

Run: `grep -E "candidate matrix|shortlist|do NOT write files" ~/Documents/01-Dev/research-team/agents/01-scout.md`
Expected: all three present.

---

## Task 7: agents/02-deep-analyst.md

**Files:**
- Create: `KIT/agents/02-deep-analyst.md`

- [ ] **Step 1: Write the file**

````markdown
# Agent 2 — Deep Analyst

**Model:** Opus · **Mode:** read-only (web + repo read). No edits.

The Scout found candidates; you pick the shortlist apart. Go deep on how each one
actually works and whether its claimed results hold up on paper.

## Brief

- **Topic:** {{TOPIC}}
- **Question:** {{QUESTION}}
- **Constraints:** {{CONSTRAINTS}}
- **Your scope (heavy runs):** {{FORK_SCOPE — else "full shortlist"}}

## Tasks

For each shortlisted candidate:

1. **Mechanism.** How does it work? For strategies/bots: the actual logic, entry/
   exit, edge thesis. For tools/APIs: the data model, method, dependencies.
2. **Math / methodology.** Inspect formulas, statistical method, assumptions.
   Are they sound? Where do they break? Note over-fitting / look-ahead / survivorship
   red flags.
3. **Code / repo quality** (if applicable). Read the repo: is it real and
   maintained, or a toy? License? Tests? Last commit? Obvious bugs in the core logic?
4. **Claimed results.** What returns/accuracy/performance are claimed, by whom,
   over what window? How credible is the source? Flag anything you can't trace as
   `UNVERIFIED` (hand to the Empiricist if testable).
5. **Fit.** Match against the question's constraints.

## Deliver (return as text — do NOT write files)

- A **per-candidate analysis**: mechanism · math/methodology verdict · code/repo
  read · claimed-results credibility · fit.
- A **ranked take** of the shortlist with the deciding factors.
- A list of **claims worth empirically testing** → hand to the Empiricist.
- **Sources:** every URL + date.

Be skeptical of claimed numbers. State your confidence. Mark unverifiable claims
`UNVERIFIED` rather than repeating them as fact.
````

- [ ] **Step 2: Verify**

Run: `grep -E "Mechanism|Math / methodology|UNVERIFIED" ~/Documents/01-Dev/research-team/agents/02-deep-analyst.md`
Expected: all three present.

---

## Task 8: agents/03-empiricist.md

**Files:**
- Create: `KIT/agents/03-empiricist.md`

- [ ] **Step 1: Write the file**

````markdown
# Agent 3 — Empiricist

**Model:** Opus · **Mode:** may run code. Read-only on tracked files + vault.

You are the team's empiricist. Others read docs; **you actually hit the
endpoints / run the numbers** and judge with your own eyes. Anything in
`must_test` you do not execute is returned `UNVERIFIED`.

## Brief

- **Topic:** {{TOPIC}}
- **Question:** {{QUESTION}}
- **Must test:** {{MUST_TEST}}
- **Constraints:** {{CONSTRAINTS}}

## Sandbox rules

- Run `curl`, `python3` (one-liners + throwaway scripts), `gh` (read-only),
  `git clone` — but **write only** under the kit's `_scratch/` directory.
- **Never** edit tracked files, the vault, or other repos. Isolate any install in
  a `venv` / `pipx` / `--user`. Never mutate a project env.
- Test **only** keyless/free public endpoints, or a key the user pasted this
  session. Never invent, hard-code, or commit a key. Redact keys in your report.
- Respect rate limits — a few sample calls, not a hammer.

## Crypto / wallet rule (hard)

Read-only chain access ONLY. Inspect public wallets / PnL via public APIs + block
explorers. **Never** sign, send a transaction, connect a wallet, or touch a
private key. If a probe would require live execution, report it `UNVERIFIED —
out of scope` instead of running it.

## Tasks

1. **Reachability.** For each testable claim/candidate, make a real call/run.
   Record exact request/command, status, latency.
2. **Inspect output.** Show a trimmed real result. Check it against the claim:
   does the data/number actually match what was promised?
3. **Spot-check correctness** against a known value where possible. Note gaps,
   nulls, suspicious figures.
4. **Probe the edges.** Bad input, missing key, rate limit — what happens? This
   shapes the real integration/use cost.

## Deliver (return as text — no files outside `_scratch/`)

- A **test log**: per claim → method, status, latency, verdict
  `WORKS / PARTIAL / FAILS / UNVERIFIED`.
- **Evidence:** trimmed real outputs (keys/secrets redacted).
- An explicit list of anything in `must_test` you could **not** verify, and why.
````

- [ ] **Step 2: Verify**

Run: `grep -E "write only|Crypto / wallet rule|UNVERIFIED" ~/Documents/01-Dev/research-team/agents/03-empiricist.md`
Expected: all three present.

---

## Task 9: agents/04-devils-advocate.md

**Files:**
- Create: `KIT/agents/04-devils-advocate.md`

- [ ] **Step 1: Write the file**

````markdown
# Agent 4 — Devil's Advocate

**Model:** Opus · **Mode:** read-only (web + repo read). No edits.

Your job is to be right, not agreeable. Assume the other agents were optimistic.
Find the reasons this choice hurts in six months. A clean report from you is a
failure — there is always a catch.

## Brief

- **Topic:** {{TOPIC}}
- **Question:** {{QUESTION}}
- **Constraints:** {{CONSTRAINTS}}

You may be given the other agents' reports, or run alongside them. If you don't
have them yet, reason from the topic and attack the obvious thesis.

## Tasks

1. **Challenge the survey.** What credible candidate got missed? Is a shortlisted
   option actually a dead end (abandoned, hype, rate-limited to uselessness,
   acquired-and-sunsetting)?
2. **Challenge the results.** Where are the claimed returns/performance the
   product of survivorship bias, cherry-picked windows, over-fitting, look-ahead,
   or undisclosed costs/fees/slippage? Build the **realistic / worst-case** view.
3. **Challenge the methodology.** Is the math sound out-of-sample? Does the edge
   decay once known? Is "real-time" actually delayed? Is the repo a toy?
4. **Longevity & silent failure.** What kills this in six months — pricing change,
   ToS, API death, edge crowding? What breaks a repeated run **silently** (a 200
   with an empty body, a quota reset, a model drift)?
5. **Alternatives.** Name options the team didn't consider, including boring ones
   (official sources, just paying a little, not doing it at all).
6. **Risk register.** Every risk with **severity (H/M/L)**, **likelihood**, and a
   concrete **mitigation**.

## Deliver (return as text — do NOT write files)

- A point-by-point **rebuttal** (cite which claim you contest).
- A **worst-case** view of the leading option.
- A **risk register** table: risk · severity · likelihood · mitigation.
- A blunt **"what I'd worry about most"** paragraph — the single thing most
  likely to make the user regret this choice.
````

- [ ] **Step 2: Verify**

Run: `grep -E "survivorship|silent|risk register" ~/Documents/01-Dev/research-team/agents/04-devils-advocate.md`
Expected: all three present.

---

## Task 10: README.md

**Files:**
- Create: `KIT/README.md`

- [ ] **Step 1: Write the file**

````markdown
# research-team

A generalist, reusable, parallel multi-agent research kit. Give it any topic via
a freeform prompt; it dispatches parallel specialist agents through an approval
gate, then synthesizes a single polished, dated research document into the
Obsidian vault.

## Invoke

From a session in any directory:

```
Run research-team: <freeform prompt>
```

Inline effort override:

```
Run research-team light: <prompt>     # 2 agents, quick lookup
Run research-team heavy: <prompt>     # 6–8 agents, big surface
```

### Examples

```
Run research-team: polymarket bots, strategies, rank them, check repos,
check wallets/accounts on polymarket, check results, structured summary
```

```
Run research-team heavy: trading strategies on social media, repos, quants,
formulas, math, results, structured summary
```

## What happens

1. The Lead expands your prompt into a structured brief and **shows it to you**.
2. You say `go` (or edit a field — scope, `must_test`, `effort`).
3. The Lead dispatches the roster in parallel:
   - **Scout** (Sonnet) — enumerate candidates, shortlist.
   - **Deep Analyst** (Opus) — mechanism, math, code, claimed results.
   - **Empiricist** (Opus) — actually run curl/python/gh to verify (sandboxed).
   - **Devil's Advocate** (Opus) — attack everything, risk register.
4. The Lead synthesizes one document and writes it to the vault.

## Output

`…/brain/08-research/YYYY-MM-DD-HHMM-<slug>.md` — frontmatter + six sections:
Verdict & recommendation · Top choices (ranked, pros/cons) · Evidence · Risks &
gotchas · Implementation overview · Sources. `UNVERIFIED` is flagged inline.

## Effort

| effort | agents | when |
|--------|--------|------|
| light | 2 | single-thing lookup |
| standard | 4 | default; compare + decide |
| heavy | 6–8 | many angles; Lead forks Scouts/Analysts with disjoint scope |

## Files

```
research-team/
├── team.md                  # orchestration spec the Lead follows
├── brief-template.md        # the brief shape prompts expand into
├── output-template.md       # the 6-section output skeleton
├── agents/00..04            # role prompts
├── briefs/                  # saved expanded briefs (audit trail)
└── _scratch/                # Empiricist sandbox (gitignored)
```

## Safety

Workers are read-only except the Empiricist, which writes only to `_scratch/` and
does **read-only** chain queries (never signs/sends/touches a key). Only the Lead
writes to the vault — exactly one file per run.

## Future work (not built yet)

A curated `tools/` folder of **reusable** probes, human-promoted only: an agent
proposes a probe in its report → you review + commit it to `tools/` → the next
run's Empiricist reuses it before writing new throwaway scripts. Deferred until
reuse actually shows up; today it's `_scratch/`-only.
````

- [ ] **Step 2: Verify**

Run: `grep -E "Run research-team|approval|_scratch|Future work" ~/Documents/01-Dev/research-team/README.md`
Expected: all four present.

- [ ] **Step 3: PROPOSED COMMIT (user runs)**

```bash
cd ~/Documents/01-Dev/research-team
git init
git add .gitignore README.md team.md brief-template.md output-template.md agents/ briefs/.gitkeep _scratch/.gitkeep docs/
git commit -m "feat: research-team multi-agent kit"
```

---

## Task 11: Vault zone — 08-research/index.md

**Files:**
- Create: `VAULT/08-research/index.md`

- [ ] **Step 1: Create the zone + index**

`VAULT/08-research/index.md`:

````markdown
---
type: index
domain: research
date: 2026-06-06
status: active
tags: [research/index]
---

# 08 — Research

Output zone for the [research-team](https://github.com) kit
(`~/Documents/01-Dev/research-team/`). One polished, dated document per
investigation. Generated, not hand-written — edit the source kit, not these docs.

## Frontmatter contract

`type: research` · `topic` · `question` · `date` · `researched` · `verdict` ·
`recommended` · `effort` · `status` (active/archived) · `tags: [research/<domain>]`
· `sources_count`.

## Conventions

- Filename: `YYYY-MM-DD-HHMM-<slug>.md` (24h, no colon).
- Never delete — move stale docs to `99-archive/08-research/`, set `status: archived`.
- Research stays out of task views; this zone is its own concern.

## All research (newest first)

```dataview
TABLE question AS "Question", verdict AS "Verdict", effort AS "Effort", researched AS "When"
FROM "08-research"
WHERE type = "research"
SORT researched DESC
```

## By domain

```dataview
TABLE rows.file.link AS "Docs"
FROM "08-research"
WHERE type = "research"
GROUP BY tags AS "Domain"
```
````

- [ ] **Step 2: Verify**

Run: `test -f "/Users/mirceaalbu/Documents/04-Software/ObsidianVaults/brain/08-research/index.md" && grep -c dataview "/Users/mirceaalbu/Documents/04-Software/ObsidianVaults/brain/08-research/index.md"`
Expected: file exists; `dataview` appears 2×.

---

## Task 12: Register type + zone in vault CLAUDE.md

**Files:**
- Modify: `VAULT/CLAUDE.md`

- [ ] **Step 1: Add the zone to the Structure list**

Find the structure list (after `99-archive/` description) and add a new bullet:

```markdown
- 08-research/ — output zone for the research-team kit (~/Documents/01-Dev/research-team/). One dated `type: research` doc per investigation. Generated, not hand-written. See `08-research/index.md`.
```

- [ ] **Step 2: Register the note type**

In the **Note types** section, add `research` to the list:

Find:
```markdown
- daily, meeting, decision, research, project, permanent, log, person, recipe, todo
```
(If `research` is already present in that line, no change needed — verify in step 3.)

And in the **Required frontmatter** `type:` enumeration, add `research`:

Find the `type: (daily | log | person | …)` block and add `research` to the union.

- [ ] **Step 3: Verify**

Run: `grep -E "08-research|type: research|research-team" "/Users/mirceaalbu/Documents/04-Software/ObsidianVaults/brain/CLAUDE.md"`
Expected: the `08-research/` structure bullet present; `research` registered as a type.

- [ ] **Step 4: PROPOSED COMMIT (user runs — vault is a separate git repo)**

```bash
cd ~/Documents/04-Software/ObsidianVaults/brain
git add 08-research/index.md CLAUDE.md
git commit -m "feat: add 08-research zone for research-team output"
```

---

## Task 13: End-to-end dry run (verification)

Real verification — no unit tests for prompt files; the proof is one clean run.

**Files:** none created by this task (produces one vault doc + one brief as output).

- [ ] **Step 1: Run a light-effort topic**

In a session, run:
```
Run research-team light: best free RSS reader apps for macOS in 2026
```

- [ ] **Step 2: Verify the approval gate fires**

Expected: the Lead shows a drafted brief (topic/question/effort=light/roster=
[scout, devils-advocate]/output_path/tags) and **waits** — does not dispatch
before you say `go`.

- [ ] **Step 3: Say `go`; verify parallel dispatch**

Expected: exactly 2 agents (Scout + Devil's Advocate) dispatched in one parallel
message. No Empiricist/Analyst for light.

- [ ] **Step 4: Verify the brief was saved**

Run: `ls ~/Documents/01-Dev/research-team/briefs/`
Expected: one `YYYY-MM-DD-HHMM-best-free-rss-reader*.md` (or similar slug).

- [ ] **Step 5: Verify the output doc**

Run: `ls "/Users/mirceaalbu/Documents/04-Software/ObsidianVaults/brain/08-research/"`
Then open the newest file and confirm:
- Filename = `YYYY-MM-DD-HHMM-<slug>.md`, 24h time, no colon.
- Frontmatter has `type: research`, `verdict`, `recommended`, `tags`, `sources_count`.
- All six sections present; Section 2 candidates ranked with Pros/Cons.
- Section 5 present (or explicitly omitted because verdict = No-Go).
- Sources have URLs + dates.

- [ ] **Step 6: Verify `_scratch/` stayed clean for a read-only run**

Run: `ls ~/Documents/01-Dev/research-team/_scratch/`
Expected: only `.gitkeep` (light run has no Empiricist, so nothing written).

- [ ] **Step 7: Verify the Dataview index picks it up**

Open `VAULT/08-research/index.md` in Obsidian; the new doc appears in the "All
research" table.

- [ ] **Step 8: (Optional) standard-effort smoke test**

```
Run research-team: <a real small topic with a testable claim>
```
Confirm 4 agents dispatch, the Empiricist writes to `_scratch/` and reports a
test log with `WORKS/PARTIAL/FAILS/UNVERIFIED` verdicts, and the Evidence section
is populated.

---

## Self-review (done by plan author)

**Spec coverage:**
- §2 roles/flow → Tasks 4–9 (team.md + 5 agent files). ✔
- §2 approval gate → team.md Task 4 step 3 + verified in Task 13 step 2. ✔
- §3 effort scaling + heavy fork → brief-template (T2), team.md (T4), README (T10). ✔
- §4 repo layout → Tasks 1, 10 (scaffold + all files). ✔
- §5 output filename/frontmatter/6 sections → output-template (T3) + verified T13. ✔
- §6 sandbox/crypto/anti-footgun → 03-empiricist (T8) + team.md rules (T4). ✔
- §6 future `tools/` note → README (T10). ✔
- §7 vault zone/index/type registration → Tasks 11–12. ✔
- §8 worked examples → README examples (T10) + dry run (T13). ✔
- §9 open items: $VAULT resolution → resolved to hardcoded constant (T4, plan header);
  index Dataview → T11; agent wording → T5–9; README → T10; CLAUDE.md edit → T12. ✔

**Placeholder scan:** `{{...}}` tokens appear only inside template/role files as
intended substitution points, not as plan gaps. No TBD/TODO. ✔

**Consistency:** filename pattern `YYYY-MM-DD-HHMM-<slug>` identical across team.md,
output-template, index, dry run. Roster names (scout/deep-analyst/empiricist/
devils-advocate) identical across brief-template, team.md, agent filenames. ✔
````
