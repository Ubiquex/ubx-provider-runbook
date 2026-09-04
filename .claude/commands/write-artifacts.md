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
   derivation from the wire's own service name. Not authored one wire
   at a time -- see "Deriving a real category" below for the real
   method (UBI-245).
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

   **Any manual `ubx sdk gen` invocation must pass this same
   `--descriptions-dir` flag, pointed at this checkout's own
   `artifacts/$ARGUMENTS/`.** Confirmed live, UBI-249: a full session
   of hand regenerations across seven providers omitted the flag every
   time, so real, individually-authored description text kept serving
   the docs site correctly (it reads `descriptions.json` directly) while
   staying entirely absent from the generated `.go`/`.ts`/`.py` code
   comments -- the exact corpus this runbook exists to write, silently
   never reaching the SDK it was written for. This is the same class of
   gap as `PROVENANCE.json` never being copied during a manual
   regeneration (see `hash-watch.yml`'s own `cp .../PROVENANCE.json`
   step): both come from working outside the automated path, which
   passes both correctly and a hand regeneration has to reproduce by
   hand. Before treating any manual regeneration as complete, confirm
   the flag was actually passed -- do not assume it from a clean build
   alone, since a build with the flag omitted also succeeds cleanly and
   looks identical until someone diffs the generated comments against
   `descriptions.json`.
3. Commit both files together, with the real running total in the
   message (`artifacts/$ARGUMENTS: intros batch N -- X/Y wires`).
4. Update the manifest's own `hops` entry for this runbook's single
   `write-intros` hop -- append a batch record (`batch`, `wires_written`,
   `commit`), do not overwrite prior batches' own records.
5. Push. `ubx-sdk-$ARGUMENTS` is PR-only, matching every SDK repo in
   this org -- open one per batch, or accumulate a few batches into one
   PR if the founder's own review cadence prefers that; either is fine
   as long as nothing merges itself.

**A batch that touches only `artifacts/$ARGUMENTS/*.json` needs a
version bump in the same PR, or the next real publish silently no-ops
or fails outright.** `publish.yml`'s own automatic version-bump logic
diffs only `sdk/go/`, `sdk/typescript/`, `sdk/python/` against the last
published tag -- an artifacts-only change is invisible to that diff, so
the workflow treats the committed version as already published and
unchanged, and `gh release create` then fails on a tag that already
exists (confirmed live, UBI-245: this is exactly what happened dispatching
`publish.yml` for six of seven providers after an artifacts-only PR).
Bump the PATCH version in `sdk/typescript/package.json`,
`sdk/typescript/deno.json`, and `sdk/python/pyproject.toml` (all three,
kept in sync -- `publish.yml` fails outright if `package.json` and
`pyproject.toml` disagree) as part of the same commit, with no other
`sdk/go`/`sdk/typescript`/`sdk/python` content changed. The one real
exception: if the committed version is already ahead of what's live on
PyPI/npm (a prior code regen bumped it but never got published), no
bump is needed here -- but confirm that by querying the live registries
directly, not by reading the committed version file alone; a registry
can advance between when you check and when `publish.yml` actually
runs, in which case it applies its own routine PATCH bump on top
regardless, and the version that actually ships may not be the one you
predicted. Either way, verify the real published version and content
after publish -- `../TRAPS.md#verify-a-publish-by-querying-each-registry-for-the-specific-version`
applies to the docs release exactly as it does to PyPI/npm/Go.

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

## Deriving a real category, not authoring from scratch (UBI-245)

A category is not authored per wire, and it is not authored from
nothing. It is derived per **service token** -- the wire's own
`service` field -- and a resource shares its token's real label with
its own same-named data source (a data source's category key is the
`data_` prefix stripped before lookup; see `lib/docs.ts` in
`ubx-docs-providers` for the real, live version of this exact rule).

For every distinct service token in the target's real schema, gather
every real, already-existing label across ALL of that token's own wire
types (resources and data sources together), then classify:

1. **Clean** -- every covered wire type agrees on exactly one real
   label. Inherits automatically, including an uncovered sibling wire
   type of the same token. No authoring needed. This is most tokens --
   real, measured range across all seven providers already onboarded,
   at the wire-type level: 74% (AWS, Datadog) to 100% (Google) resolve
   this way with zero hand-authoring.
2. **Ambiguous** -- more than one distinct real label across the
   token's own covered wire types. The token genuinely covers more
   than one real product (AWS's own `ec2` token really does span
   Amazon EC2, Amazon VPC, AWS Transit Gateway, and AWS Verified
   Access) -- split it by what each wire's own schema fields actually
   describe: real, verified keyword/substring rules or a lookup table,
   built and checked against every already-covered wire type of that
   token until it matches 100%, never guessed and left unverified. A
   token that only LOOKS ambiguous -- the same real service spelled two
   ways by two different generators (CFN vs. Smithy, for one real
   example) -- is not this case; canonicalize first (strip
   underscores, compare) and merge before splitting anything, or a
   spelling variant gets treated as a second product it was never
   meant to be. Real, measured total across all seven providers already
   onboarded: 25 ambiguous tokens, combined.
3. **No-signal** -- zero real labels anywhere on the token. Author a
   real product name: check this target's own `categories.json` for an
   existing label that already covers the same real product first
   (reuse it, don't invent a near-duplicate), and verify against the
   vendor's own current documentation for anything genuinely
   unfamiliar rather than guessing at a plausible-sounding name -- a
   flagged low-confidence entry is cheap to correct later, a
   confidently wrong product name is not. This bucket skews heavily
   toward read-only, data-source-only services (AWS's own no-signal
   bucket alone was 1,308 of 6,241 real wire types, almost entirely
   data sources with no resource sibling to inherit a label from) --
   for most of them the real category is simply the product the
   read-only API reads from (AWS's own `access_analyzer` is IAM Access
   Analyzer).

Batch and commit the no-signal authoring the same way intros are
batched above -- a manifest, real running totals per commit, low and
medium confidence entries flagged with their own reasoning rather than
committed at silent full confidence.

**Verify 100% coverage before calling this done.** A live recompute
(the same shape as intro coverage: diff the real, current schema's own
service tokens against the real, currently-committed
`categories.json`) must report zero wire types with no real label --
not "most," not "the batches that were planned." `ubx-docs-providers`'
own `build-sidebar-index.mjs` stopped inferring a label for an
uncovered wire type once this method made 100% coverage real (UBI-245)
-- an uncovered wire type there now surfaces honestly as
`Uncategorized` rather than being silently guessed into a plausible-
looking group, which means an incomplete categorization pass is no
longer invisible on the real site, it is a visible regression the
moment it ships.

## Done

This runbook's own `write-intros` hop is `done` once a live recompute
(same command as above) reports zero gaps for `$ARGUMENTS`. Mark it so
in the manifest, with the final real wire count.

This is optional polish (`onboard-provider.md`'s own hop 8), not a
blocker -- the provider has already been live and correctly rendering
on `ubx-docs-providers` since that runbook's own hop 7, zero
descriptions or not.
