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
