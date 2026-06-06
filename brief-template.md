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
