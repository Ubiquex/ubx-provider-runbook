Regenerate `$ARGUMENTS`'s pinned schema snapshot against its own real,
current upstream, when upstream has drifted since the last pin.

## Before starting

Read `../TRAPS.md` and `../MANIFEST.md` if you have not already this
session.

Check for an existing manifest at `.regen-manifest.json` in
`ubx-schema-$ARGUMENTS`'s own repo root (fresh `git fetch` first). Resume
from the first hop not `done`, re-verifying rather than trusting the
hop you're resuming from or past.

If no manifest exists, create one now (`runbook: "regen-schema"`,
`target: "$ARGUMENTS"`, `hops: []`) and commit it as the first step.

## This is usually already half-automated -- check first

Every real `ubx-schema-<provider>` repo carries its own `hash-watch.yml`
(weekly schedule + `workflow_dispatch`), which builds
`ubx-provider-dynamic` from its own latest real, tagged release and runs
`--generate-snapshot-group` against `$ARGUMENTS`'s own real, current
upstream. If the regenerated content differs from what's committed, it
opens a PR itself -- never merges, and never labeled (checked directly
against the real workflow file -- its own `gh pr create` call carries no
`--label`). Check `gh pr list --state open` for a branch named
`snapshot-regen/$ARGUMENTS-<version>` before triggering anything by hand
(`gh workflow run hash-watch.yml`).

Mark `trigger` `done` noting whether this run was `hash-watch`'s own PR
or a manual `workflow_dispatch`, with the PR URL either way.

**A version bump alone does not mean the schema itself changed.**
Confirmed live: dispatching this against Kubernetes with its own real
upstream spec unchanged for days still opened a real PR, version 3.0.1
-> 3.0.2, both real members individually logged `own change level: none`.
The only line that changed was `manifest.json`'s own `min_binary_version`
-- `ubx-provider-dynamic` had itself published a new release
(1.0.1 -> 1.0.2) between the two runs, and every snapshot this tool
builds stamps its own current version into that field regardless of
whether the schema content moved at all (confirmed against that
release's own real GitHub release notes). This is real, intentional,
documented behavior, not a bug -- but it means "hash-watch opened a PR"
is not by itself evidence the upstream schema drifted. Hop 1 below
(read the real diff) is what actually answers that question; do not
skip it because a PR exists.

## Hop 1: review the real diff, not the PR's own summary

`hash-watch`'s own PR carries the regenerated `manifest.json` and
`members/*.json` content, diffed against what's committed. Read the
real diff -- not just the change-level `hash-watch` itself computed
(`NoChange`/`Patch`/`Minor`/`Major`, from `AssembleGroup`'s own highest
member-level change) -- before trusting it. A "real diff, not assumed
small" check has caught unrelated upstream drift hiding inside what
looked like a scoped fix more than once in this project (a new AWS
service, real service-directory renames, field-level changes alongside
an unrelated fix -- `ubiquex` `STATE.md`, UBI-201's own entry).

If the diff is larger than expected for what triggered this regen,
report the real scope before merging -- do not assume it's noise. If
the diff is ONLY `manifest.json`'s own `min_binary_version` field with
no `members/*.json` change at all, that's the real no-op case above --
still worth merging (the stamped version should reflect what actually
built it), but report it as exactly that, not as a schema update.

Mark `done` with the real diff scope (member count changed, resource
count delta, or "toolchain version only, zero schema content change")
once reviewed.

## Hop 2: merge, cut the release

Never self-merge (this repo, like every schema repo, is PR-only).
Once the founder merges: cut a real GitHub Release for the new version
(`.github/workflows/publish.yml`, manual dispatch only in every real
schema repo -- confirm the exact trigger from that repo's own workflow
file rather than assuming). Verify the release via
`gh api repos/Ubiquex/ubx-schema-$ARGUMENTS/releases/tags/v<version>`
directly -- never the workflow's own exit status
(`../TRAPS.md#verify-a-publish-by-querying-each-registry-for-the-specific-version`
applies to a GitHub Release the same as it does to npm/PyPI/the Go
proxy).

## Hop 3: bump the pin in ubiquex

`sdk/providers/.ubx/config`'s own `[dynamic_providers.$ARGUMENTS]`
entry (never a separate `[providers.$ARGUMENTS]` table -- that is a
genuinely different, newer mechanism `ubx sdk gen` never reads; see
`onboard-provider.md`'s own hop 5 for the real incident this note
comes from), `version` bumped to the new release. Real PR (`ubiquex` is
direct-push by convention, but every hop in this chain still opens
one). Verify
`ubx sdk gen --only $ARGUMENTS --dump-ir <dir>` resolves the new pin
cleanly (zero-network on a cache hit is expected; confirm it's actually
reading the new version, not a stale local cache, by checking the
resolved version in the command's own output) before merging.

## Hop 4: cascade -- SDK regen, then docs

A schema regen is not the end state on its own. Once the pin bump
merges:

- Run `ubx sdk gen --only $ARGUMENTS --lang go/py/ts` and open PRs
  against `ubx-sdk-$ARGUMENTS`, matching `onboard-provider`'s own hop 6
  (this is the identical mechanism, just against an existing repo
  instead of a new one). Publish and verify per registry, same as
  there.
- Once published: any resource whose fields changed shape needs its
  own artifacts re-checked -- a description authored against the OLD
  field set is not automatically wrong for the new one, but a
  genuinely new or renamed field has no description at all until
  `/write-artifacts $ARGUMENTS` runs again.
- Docs regeneration (`/regen-docs $ARGUMENTS`) already runs on its own
  schedule against whatever's currently pinned (UBI-137's own
  `resource-reference-regen.yml`) -- this hop does not need to trigger
  it manually unless the new content needs to appear before that
  schedule's own next run.

Mark this hop `done` once the SDK side is published and verified; note
whether artifacts/docs were run as part of the same session or left
for their own.
