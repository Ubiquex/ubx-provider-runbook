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
opens a PR itself -- never merges. Check `gh pr list --label <whatever
hash-watch tags its own PRs, confirm from the workflow file>` for an
already-open one before triggering anything by hand
(`gh workflow run hash-watch.yml`).

Mark `trigger` `done` noting whether this run was `hash-watch`'s own PR
or a manual `workflow_dispatch`, with the PR URL either way.

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
report the real scope before merging -- do not assume it's noise.

Mark `done` with the real diff scope (member count changed, resource
count delta) once reviewed.

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

`sdk/providers/.ubx/config`'s own `[providers.$ARGUMENTS]` entry,
`version` bumped to the new release. Real PR (`ubiquex` is direct-push
by convention, but every hop in this chain still opens one). Verify
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
