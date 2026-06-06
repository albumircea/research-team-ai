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
# research-team-ai
