Regenerate `$ARGUMENTS`'s own Resource Reference pages against whatever
schema is currently pinned, using the same real mechanism
`resource-reference-regen.yml` already runs in CI (UBI-137) -- this
runbook is the manual, scoped path through it, not a separate
implementation.

## Before starting

Read `../TRAPS.md` and `../MANIFEST.md` if you have not already this
session.

## No persistent manifest here, deliberately

The other three runbooks resume across sessions because each hop
produces real, committed, cumulative state (a schema pin, a batch of
artifacts). Docs regeneration doesn't work that way -- it's re-derived
fresh from whatever's currently pinned and currently committed in
`artifacts/`, every time, never resumed against a stale prior run. The
real manifest here is `regen_all.py`'s own JSON report, written fresh
on every invocation to `/tmp/regen-scratch/$ARGUMENTS_regen_result.json`
and `$ARGUMENTS_datasource_regen_result.json` -- read it after running,
don't look for a committed one from a prior session.

## Hop 1: confirm this needs to run at all

`resource-reference-regen.yml` already runs on push to `main` touching
`artifacts/**`, plus a weekly schedule. If `/write-artifacts $ARGUMENTS`
just finished and pushed real commits to `main`, that push alone
already triggers a real CI run -- check
`gh run list --workflow resource-reference-regen.yml` for one in
progress or recently finished before starting a manual one. This
runbook exists for: a provider not yet covered by the schedule's own
next window, a scoped one-provider run instead of waiting for all six,
or investigating a CI run that reported blocked.

## Hop 2: build and generate, same real steps CI takes

```
go build -C <ubiquex checkout> -o ubx ./cmd/ubx
./ubx version   # confirm this matches the commit you meant to build -- TRAPS.md
                 # "build from current source, not whatever binary is in the path"

./ubx sdk gen --only $ARGUMENTS --dump-ir <dump-dir> --out <throwaway>
./ubx sdk gen --only $ARGUMENTS --lang go --out <local-sdk-root>
./ubx sdk gen --only $ARGUMENTS --lang py --out <local-sdk-root>
./ubx sdk gen --only $ARGUMENTS --lang ts --out <local-sdk-root>
```

`$ARGUMENTS` here is the real published SDK repo's own short name --
`gcp`'s own docs-internal key is `google` for this exact command
(`sdk/providers/.ubx/config`'s own real naming; every other provider's
two names already match). Confirm which before running if in doubt.

## Hop 3: run the real orchestrator, not the generators by hand

```
python3 scripts/resource-reference-gen/regen_all.py \
  --dump-root <dump-dir> \
  --local-sdk-root <local-sdk-root> \
  --docs-root <ubiquex-docs checkout> \
  --only $ARGUMENTS
```

This is the exact script CI runs -- it excludes any resource or data
source missing an intro, category, or depth-0 field description from
the batch it writes (never ships a gap), and refuses outright if
`UBX_DOCS_ALLOW_COVERAGE_GAPS` is set in the environment. **Never set
that variable for this runbook.** It exists for a human running a
deliberate, one-off experiment against a single page -- never for a
real regeneration meant to ship, since it would silently reintroduce
the exact templated-page/blank-field failure UBI-216's own artifact
mandate exists to prevent.

Read the script's own stdout report -- per-wire coverage results, pages
kept, pages excluded and why. If it reports gaps: that's real, expected
output naming exactly which wires still need `/write-artifacts`, not a
tooling failure.

## Hop 4: verify, then open a PR

```
mint validate
```

in `ubiquex-docs`, clean, before committing anything. Then
`git status --porcelain` -- if nothing changed, there is nothing to
commit, and that's a normal, complete outcome (report it as such, not
as though nothing happened). If something changed: commit, push a
branch, open a PR. Never self-merge, matching every other hop in this
chain.

## A note on trusting this script's own output

`stage_gap_free.py` (part of this same chain) once promised pure JSON
on its own stdout while a function it called printed real log lines
onto that same stream first -- looked correct to a human, broke the
first time an actual caller parsed it as JSON. If you're modifying any
part of this chain rather than just running it, verify the literal
output contract, not just that it looks right --
`../TRAPS.md#a-scripts-stdout-looking-right-to-a-human-is-not-the-same-as-its-being-parseable`.
