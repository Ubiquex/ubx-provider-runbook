Write the artifacts `$ARGUMENTS`'s own resource and data-source pages
need before docs regeneration will ship them: an intro, a category, and
a depth-0 field description for every field that doesn't already have
one from the vendor's own spec.

## Before starting

Read `../TRAPS.md` and `../MANIFEST.md` if you have not already this
session.

**UBI-240: the target is `ubx-sdk-$ARGUMENTS`'s own `artifacts/$ARGUMENTS/`
directory, not `ubiquex-docs`.** Every provider's own canonical corpus
now lives in its own SDK repo (see `ubx-sdk-kubernetes`'s own
`artifacts/kubernetes/README.md` for the real account of why). A
handful of already-migrated providers still carry a second, no-longer-
canonical copy in `ubiquex-docs` too, transitionally, only because
live Mintlify pages still read it -- never write there for a session
that starts after this rewrite; write the SDK repo's own copy, and if
that provider's Mintlify pages also need the update, that's
`/regen-mintlify-docs`'s own separate, optional job (see
`onboard-provider.md`'s own hop 8), not this runbook's.

## What "mandatory" actually covers

`coverage_check.py` (`check_gaps`, in `ubiquex-docs`) is the real gate
this runbook exists to satisfy -- read it directly
(`scripts/resource-reference-gen/coverage_check.py` in `ubiquex-docs`)
rather than assuming its scope. Against an SDK-repo target it checks
exactly three things per wire, never more (the fourth and fifth --
on-disk page reachability in both directions -- only apply against
`ubiquex-docs`' own `resource-reference/` tree, and are skipped
automatically by `--artifacts-root` below, since an SDK-repo-only
target has no such tree):

1. **An intro** -- one real paragraph, `artifacts/$ARGUMENTS/intros.json`.
2. **A category** -- a service-level label, either an explicit override
   in `artifacts/$ARGUMENTS/categories.json` or a valid default
   derivation from the wire's own service name.
3. **A depth-0 field description for every top-level field** --
   `artifacts/$ARGUMENTS/descriptions.json`, depth-0 ONLY. A nested
   field with no description is not a gap this check enforces.

Field descriptions are frequently already covered for free: the real
vendor schema itself fills any empty depth-0 field description
directly, whenever the source tag isn't already `vendor-spec` (meaning
the vendor text is already baked into the schema dump itself).
DigitalOcean's own real numbers at onboarding: 1,627 of 10,609 total
fields already provider-sourced (15%) with zero hand-authoring needed
for those. The real, batchable, expensive work is **intros** -- one
per wire, no vendor text to fall back on, and the reason this runbook
exists at all.

**No hardcoded provider allowlist to hit here anymore.**
`coverage_check.py`'s own `DUMP_DIR`/`providers.py` registry --
DigitalOcean's own real "unknown provider" failure, the hop this
runbook's own core command used to fail outright on for any genuinely
new provider -- is not consulted at all when `--artifacts-root` is
given (see the command below): a provider need not be a real
`providers.py` key to be checked this way. `TRAPS.md`'s own
"hardcoded provider allowlist" entry is real, confirmed history, but
it does not apply to this path.

## Check for an existing manifest

`.artifact-manifest.json` in `ubx-sdk-$ARGUMENTS`'s own repo root
(fresh `git fetch` first). If one exists, read its own batch history
for context (what was tried, when, how many wires per batch) -- then
immediately recompute the real remaining set live (next section).
Never trust the manifest's own last count as current; see
`../MANIFEST.md#the-manifest-records-history-never-the-resume-point`
for why, with a real, confirmed example of this exact mistake.

If no manifest exists, create one now (`runbook: "write-artifacts"`,
`target: "$ARGUMENTS"`, `hops: []`).

## Recompute the real remaining set, every session

```
python3 scripts/resource-reference-gen/coverage_check.py \
  --dump-root <a fresh --dump-ir of $ARGUMENTS> \
  --only $ARGUMENTS \
  --artifacts-root <path to the ubx-sdk-$ARGUMENTS checkout>
```

Run from a `ubiquex-docs` checkout (the script itself still lives
there -- the authoring toolchain stays in one place, pointed at
whichever repo's own artifacts it's checking, matching
`extract_idents.py`'s own existing precedent). Read its own report
directly -- `missing_intro`, `missing_category`,
`missing_field_description` are the real, current, authoritative work
list.

## Batch, write, commit -- one real batch per commit

Matching the real precedent this project already has (AWS's own intro
work across thirteen commits, Azure across fifteen, each one committed
with the real running total in its own message, e.g. "batch 8 --
587/1030 new resource types"): write intros in batches sized to what
fits comfortably in the session's own remaining context, not a fixed
number -- a hundred intros will not fit in one session, and guessing a
batch size wrong either wastes a session stopping early or risks
running out mid-batch with nothing committed.

After each batch:

1. Write the batch's own intros/descriptions/categories into the real
   `artifacts/$ARGUMENTS/*.json` files in the `ubx-sdk-$ARGUMENTS`
   checkout.
2. Regenerate the codegen-ready export in the same checkout:
   ```
   python3 scripts/resource-reference-gen/export_raw_descriptions.py \
     $ARGUMENTS "<Display Name>" \
     --dump-root <the same fresh --dump-ir used above> \
     --descriptions-path <ubx-sdk-$ARGUMENTS checkout>/artifacts/$ARGUMENTS/descriptions.json \
     --nested-out <ubx-sdk-$ARGUMENTS checkout>/artifacts/$ARGUMENTS/$ARGUMENTS.json
   ```
   This is what `hash-watch.yml`'s own `--descriptions-dir` actually
   reads (UBI-240) -- a batch that updates `descriptions.json` without
   also refreshing this file leaves the real, checked-in codegen
   corpus stale until the next session happens to notice.
3. Commit both files together, with the real running total in the
   message (`artifacts/$ARGUMENTS: intros batch N -- X/Y wires`).
4. Update the manifest's own `hops` entry for this runbook's single
   `write-intros` hop -- append a batch record (`batch`, `wires_written`,
   `commit`), do not overwrite prior batches' own records.
5. Push. `ubx-sdk-$ARGUMENTS` is PR-only, matching every SDK repo in
   this org -- open one per batch, or accumulate a few batches into one
   PR if the founder's own review cadence prefers that; either is fine
   as long as nothing merges itself.

A session that runs out of context mid-batch: whatever was actually
committed stands (never leave written-but-uncommitted artifact edits --
`../TRAPS.md#run-git-status-before-claiming-work-is-committed` applies
here exactly). The next session's own live recompute picks up cleanly
from real committed state, manifest or no manifest.

## Writing a real intro, not template text

UBI-216's own decision exists because of a real, corrected failure:
4,200 pages once carried an identical templated sentence instead of a
real intro. An intro names what the resource actually is and does, in
the vendor's own real terms -- not a mechanical restatement of its own
name. Read the wire's own real fields and the vendor spec's own
description text (if any) before writing one; a wire with genuinely
opaque fields and no vendor text at all is worth flagging rather than
guessing at content that reads confidently but is wrong.

## Done

This runbook's own `write-intros` hop is `done` once a live recompute
(same command as above) reports zero gaps for `$ARGUMENTS`. Mark it so
in the manifest, with the final real wire count.

This is optional polish (`onboard-provider.md`'s own hop 8), not a
blocker -- the provider has already been live and correctly rendering
on `ubx-docs-providers` since that runbook's own hop 7, zero
descriptions or not.
