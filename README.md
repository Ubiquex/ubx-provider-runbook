# ubx-provider-runbook

Executable runbooks for the four recurring operations against a `ubx`
provider, as Claude Code slash commands (`.claude/commands/`, invoked
`/onboard-provider <name>`, `/regen-schema <name>`,
`/write-artifacts <name>`, `/regen-mintlify-docs <name>`).

This repo carries the procedure -- real commands, real verification,
real traps. `ubiquex-internals` carries the explanation of why the
process is shaped this way (`ubiquex-internals`'s own [Provider
Runbooks](https://ubiquex-internals.mintlify.app/provider-runbooks) page).

## Why this exists

The process for onboarding a provider, regenerating a schema, and
writing artifacts used to live only in conversations -- reconstructed
each time, prone to skipping a step or a trap a prior session already
paid to learn. This is a checked-in, versioned, reviewable version of
that process instead.

## The runbooks

| Command | What it does |
| --- | --- |
| `/onboard-provider <name>` | Spec discovery through publishing on `ubx-docs-providers`, for a provider that has never been onboarded. |
| `/regen-schema <name>` | Cut a fresh pinned snapshot when a provider's own upstream spec has drifted. |
| `/write-artifacts <name>` | Write the intros, categories, and depth-0 field descriptions a provider's own pages need, in its own `ubx-sdk-<name>` repo. |
| `/regen-mintlify-docs <name>` | Mintlify only (docs.ubiquex.io) -- regenerate Resource Reference pages against whatever's currently pinned. Not part of a new provider's own default path. |

`onboard-provider` is the real, end-to-end path for a brand new
provider (UBI-240): it ends live on `ubx-docs-providers`
(providers.ubiquex.io) with zero descriptions authored, `write-artifacts`
an optional, additive follow-up for richer pages, never a blocker.
`regen-schema` -> (SDK regen, which also refreshes the docs release) ->
`write-artifacts` (for anything genuinely new or renamed) is the same
shape for an existing provider whose upstream changed.
`regen-mintlify-docs` only enters either path when a provider is also,
separately, meant to carry Mintlify coverage -- the seven providers
already dual-tracked there are the current, transitional case, not a
pattern to extend.

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
