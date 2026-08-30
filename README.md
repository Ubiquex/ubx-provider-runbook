# ubx-provider-runbook

Executable runbooks for the four recurring operations against a `ubx`
provider, as Claude Code slash commands (`.claude/commands/`, invoked
`/onboard-provider <name>`, `/regen-schema <name>`,
`/write-artifacts <name>`, `/regen-docs <name>`).

This repo carries the procedure -- real commands, real verification,
real traps. `ubiquex-internals` carries the explanation of why the
process is shaped this way (`ubiquex-internals`'s own [Provider
Runbooks](https://ubiquex.mintlify.app/provider-runbooks) page).

## Why this exists

The process for onboarding a provider, regenerating a schema, and
writing artifacts used to live only in conversations -- reconstructed
each time, prone to skipping a step or a trap a prior session already
paid to learn. This is a checked-in, versioned, reviewable version of
that process instead.

## The runbooks

| Command | What it does |
| --- | --- |
| `/onboard-provider <name>` | Spec discovery through published SDK packages, for a provider that has never been onboarded. |
| `/regen-schema <name>` | Cut a fresh pinned snapshot when a provider's own upstream spec has drifted. |
| `/write-artifacts <name>` | Write the intros, categories, and depth-0 field descriptions a provider's own pages need before they can ship. |
| `/regen-docs <name>` | Regenerate Resource Reference pages against whatever's currently pinned. |

`onboard-provider` -> `write-artifacts` -> `regen-docs` is the real,
end-to-end path for a brand new provider. `regen-schema` -> (SDK regen)
-> `write-artifacts` (for anything genuinely new or renamed) ->
`regen-docs` is the same shape for an existing provider whose upstream
changed.

## Two things every runbook holds to

**They carry the traps, not just the happy path.** `TRAPS.md` is real,
confirmed failures from this project's own history, most more than
once. Every runbook links into it at the exact step where a trap
applies.

**They resume, they don't restart.** `MANIFEST.md` is the shared
convention: each runbook records which hop it reached in a small,
committed JSON file, so a session that runs out of context partway
through picks up from real, current state instead of starting over --
and never trusts its own manifest's last count over a fresh, live
recheck.

## Proving case

DigitalOcean. Onboarded manually first, real intervention points
recorded from what actually happened (not guessed) before any of this
was written: the spec needed a new bundling flag and a Node dependency
before it would load at all; the first push to the new schema repo was
blocked by a GitHub secret scan on vendor placeholder content, needing
a founder click; a stale `STATE.md` claimed a release that had never
actually landed. All three are in `TRAPS.md`.

## Never self-merge

Every runbook opens a PR for every hop of real work. None of them merge
anything themselves.
