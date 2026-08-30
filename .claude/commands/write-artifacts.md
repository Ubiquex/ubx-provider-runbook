Write the artifacts `$ARGUMENTS`'s own resource and data-source pages
need before docs regeneration will ship them: an intro, a category, and
a depth-0 field description for every field that doesn't already have
one from the vendor's own spec.

## Before starting

Read `../TRAPS.md` and `../MANIFEST.md` if you have not already this
session.

## What "mandatory" actually covers

`ubiquex-docs`'s own `coverage_check.py` (`check_gaps`) is the real
gate `regen-docs` refuses to ship past -- read it directly
(`scripts/resource-reference-gen/coverage_check.py` in `ubiquex-docs`)
rather than assuming its scope. It checks exactly three things per wire,
never more:

1. **An intro** -- one real paragraph, `artifacts/$ARGUMENTS/intros.json`.
2. **A category** -- a service-level label, either an explicit override
   in `artifacts/$ARGUMENTS/categories.json` or a valid default
   derivation from the wire's own service name.
3. **A depth-0 field description for every top-level field** --
   `artifacts/$ARGUMENTS/descriptions.json`, depth-0 ONLY. A nested
   field with no description is not a gap this check enforces.

Field descriptions are frequently already covered for free: `inject_description`
(`build_regen_schema.py`) fills any empty depth-0 field description
directly from the vendor's own spec text before this check ever runs,
whenever the source tag isn't already `vendor-spec` (meaning the vendor
text is already baked into the schema dump itself). DigitalOcean's own
real numbers at onboarding: 1,627 of 10,609 total fields already
provider-sourced (15%) with zero hand-authoring needed for those. The
real, batchable, expensive work is **intros** -- one per wire, no
vendor text to fall back on, and the reason this runbook exists at all.

## Check for an existing manifest

`scripts/resource-reference-gen/artifact-manifests/$ARGUMENTS.json` in
`ubiquex-docs` (fresh `git fetch` first). If one exists, read its own
batch history for context (what was tried, when, how many wires per
batch) -- then immediately recompute the real remaining set live (next
section). Never trust the manifest's own last count as current; see
`../MANIFEST.md#the-manifest-records-history-never-the-resume-point`
for why, with a real, confirmed example of this exact mistake.

If no manifest exists, create one now (`runbook: "write-artifacts"`,
`target: "$ARGUMENTS"`, `hops: []`).

## Recompute the real remaining set, every session

```
python3 scripts/resource-reference-gen/coverage_check.py \
  --dump-root <a fresh --dump-ir of $ARGUMENTS> \
  --only $ARGUMENTS
```

Read its own report directly -- `missing_intro`, `missing_category`,
`missing_field_description` are the real, current, authoritative work
list. This is the same check `regen_pages.py`/`gen_all_data_source_pages.py`
run against their own freshly-regenerated batch; running it standalone
here, against the FULL current corpus (not just a batch already
written), is what makes it the correct resume-point source instead of
the manifest's own count.

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
   `artifacts/$ARGUMENTS/*.json` files.
2. Commit with the real running total in the message
   (`artifacts/$ARGUMENTS: intros batch N -- X/Y wires`).
3. Update the manifest's own `hops` entry for this runbook's single
   `write-intros` hop -- append a batch record (`batch`, `wires_written`,
   `commit`), do not overwrite prior batches' own records.
4. Push. `ubiquex-docs` is direct-push by convention, but per UBI-216's
   own decision every hop in this chain opens a PR regardless -- open
   one per batch, or accumulate a few batches into one PR if the
   founder's own review cadence prefers that; either is fine as long as
   nothing merges itself.

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
in the manifest, with the final real wire count. `/regen-docs $ARGUMENTS`
is unblocked at that point -- it does not need to be triggered from
this session; it runs on its own schedule against whatever's currently
committed.
