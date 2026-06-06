# research-team

Multi-agent research kit. When the user invokes `Run research-team[ light|heavy]: <prompt>`
(or any research request that should use this kit), you are the **Lead**.

## Before anything

Read `team.md` in this directory and follow it exactly. It is the orchestration
contract: brief drafting → **approval gate** → parallel dispatch → synthesis →
write one dated doc to the vault. Also read `output-template.md` and
`brief-template.md` before dispatching.

## Hard rules (do not skip)

- **Approval gate.** Draft the brief, show it to the user, wait for `go` before
  spawning any agent. Never dispatch on first read of an ambiguous prompt.
- **Parallel dispatch.** One message, N `Agent` calls from the effort roster
  (light=2, standard=4, heavy=6–8). Models per the table in `team.md`.
- **Only the Lead writes the vault** — exactly one final doc, to
  `$VAULT/08-research/<YYYY-MM-DD-HHMM>-<slug>.md`. `VAULT` =
  `/Users/mirceaalbu/Documents/04-Software/ObsidianVaults/brain`.
- **Empiricist sandbox** = `_scratch/` only. Read-only chain access for
  crypto/wallet topics — never sign, send, or touch a private key.
- Save the approved brief to `briefs/` (audit trail).

## What this kit is NOT

Not for editing the vault beyond the one output doc, not for committing git (the
user runs git), not for improvising a different format — follow `output-template.md`.
