Onboard `$ARGUMENTS` as a new `ubx` provider, end to end: from confirming
a real schema source exists through published SDK packages.

## Before starting

Read `../TRAPS.md` and `../MANIFEST.md` in full (this repo's root) if
you have not already this session.

Check for an existing manifest at
`sdk/providers/.onboarding/$ARGUMENTS.json` in a fresh, freshly-fetched
checkout of `ubiquex`. If one exists, report every hop's own recorded
status before doing anything else, and resume from the first hop that
is not `done`. Do not trust a `done` status blindly for the hop you are
about to resume from or past -- re-verify it against real, current state
(the trap this whole mechanism exists to avoid: MANIFEST.md's own
"records history, never the resume point").

If no manifest exists, create one now with `runbook: "onboard-provider"`,
`target: "$ARGUMENTS"`, and an empty `hops` array, and commit it as the
first real step (a manifest that only appears once the run is finished
defeats its own purpose).

After every hop below, append or update that hop's own entry in the
manifest and commit it in the same commit as the hop's own real work
where possible.

## Hop 1: spec discovery

Confirm `$ARGUMENTS` publishes a real, public, machine-readable schema
source -- OpenAPI, a discovery-doc format, or another format
`ubx-provider-dynamic` already parses (`internal/openapi`,
`internal/discoverydoc`, `internal/cloudformation`, `internal/smithy` --
see `ubiquex-internals`'s [Provider System](https://ubiquex.mintlify.app/provider-system)
for what each covers). A URL returning 200 is not the bar -- see
`../TRAPS.md#a-spec-existing-is-not-the-same-as-it-loading`. This hop is
not done until you have actually run the real loader against the real
spec and it produced real resource/data-source counts, not until the
spec merely fetched.

If `$ARGUMENTS` is one of several unconfirmed candidates, the survey
method already used for 17 real candidates is `sdk/providers/docs/candidate-provider-survey.md`
in `ubiquex` -- fetch the real spec, run the real
`internal/resourcemap.Discover`/`DiscoverDataSources` against it, record
real counts. A spec existing with no fetchable URL, or generated from a
private spec never published for external fetch, is a real "no" --
confirmed per-candidate, not assumed from a repo's README alone.

Mark this hop `done` with the real resource/data-source counts in
`note`, or `blocked` with the specific loader failure if the spec does
not load as published (continue to hop 2 either way -- a live entry with
a known loader gap is still useful, real progress).

## Hop 2: declare a live entry

Add `[dynamic_providers.$ARGUMENTS]` to `ubiquex`'s own
`sdk/providers/.ubx/config`, live (`schema_source`/`schema_url`, no
pinned `source`/`version` yet -- no `ubx-schema-$ARGUMENTS` repo exists
yet). This is a real, reviewable PR on its own (`ubiquex` is
direct-push by convention, but per UBI-216's own decision, every hop in
this chain opens a PR regardless). Real precedent:
`https://github.com/Ubiquex/ubiquex/pull/30`.

If hop 1 found the spec does not load as published, this entry's own
loading may need a provider-specific flag (`redocly_bundle`, or
whatever the real blocker turns out to need) before `ubx sdk gen` will
succeed against it at all -- see hop 3.

Mark `done` with the PR URL once merged.

## Hop 3: make the loader succeed

Run `ubx sdk gen --only $ARGUMENTS --dump-ir <dir> --out <dir>` against
the live entry. If this fails, the failure is real work, not a blocker
to work around silently:

- If it's a parser gap in `ubx-provider-dynamic` (a spec convention the
  existing parser doesn't handle), that's real, scoped Go work in that
  repo. Report the exact cost before building anything speculative --
  DigitalOcean's own real example: two Redocly-only `$ref` conventions
  needed either a Go bundler reimplementation (real, non-trivial, path
  rewriting during content hoisting) or shelling out to
  `npx @redocly/cli bundle` behind a new per-provider config flag
  (`redocly_bundle`), gated so only a provider that explicitly needs it
  pays the Node.js dependency cost. The second was built
  (`https://github.com/Ubiquex/ubx-provider-dynamic/pull/39`) after
  reporting that exact tradeoff and getting it confirmed, not before.
- If it's a config issue (wrong `base_url`, a wire-naming collision), fix
  the config entry directly.

This hop is not done until a real `ubx sdk gen --dump-ir` run against
`$ARGUMENTS` produces real output -- resource/data-source counts, field
counts, source-vs-inferred description coverage. Record those real
numbers in the manifest, not an estimate.

## Hop 4: create and push the schema repo

`ubx sdk gen --only $ARGUMENTS --dump-ir <dir>` output is what a
`ubx-schema-$ARGUMENTS` repo's own `v1.0.0` snapshot is generated from
(`manifest.json` + `members/*.json`, matching every other real
`ubx-schema-<provider>` repo's own shape). Create the repo (public,
matching every other real schema repo), commit the generated snapshot,
push.

**The first push to a brand new schema repo can be blocked by GitHub's
own secret-scanning push protection**, even when nothing live is in the
content -- see `../TRAPS.md#a-github-secret-scan-block-on-vendor-content-needs-a-human-click`.
DigitalOcean's own real example: a Slack Incoming Webhook URL PATTERN
in two vendor-sourced fields, confirmed by reading the flagged bytes
directly to be DigitalOcean's own real documentation placeholder text,
not a live credential. If this happens: do not scrub the content before
confirming it's genuinely not live, and do not report the push as
succeeded -- record the hop as `blocked`, name the exact unblock URL the
rejected push printed, and note that only a repo admin can act on it.
This is a real, expected stopping point for this runbook, not a
failure of it.

**Do not claim "published" until a fresh `git fetch` against the real
remote confirms the push landed** -- see
`../TRAPS.md#dont-trust-a-prior-sessions-own-published-claim`. A repo
existing with local commits ready to push is not the same as the push
having succeeded.

## Hop 5: cut the schema release, switch the pin

Once hop 4's push has genuinely landed (verified per above): cut a real
GitHub Release (`v1.0.0`, matching the committed snapshot), then switch
`ubiquex`'s own `[dynamic_providers.$ARGUMENTS]` entry to
`[providers.$ARGUMENTS]` with `source`/`version` pointing at the real
release -- never both a live entry and a pin for the same provider at
once. A real PR, verified merged by reading the config file's own
content after, not the merged flag.

## Hop 6: generate and publish SDK bindings

`ubx sdk gen --only $ARGUMENTS --lang go/py/ts --out <dir>` against the
now-pinned schema. Create `ubx-sdk-$ARGUMENTS` (public), commit the
generated bindings, push, open a PR. Once merged, dispatch that repo's
own `publish.yml`.

**Verify the publish by querying each registry for the specific
version** -- npm, PyPI, the Go module proxy, each queried for the exact
new version string, never `@latest` and never the workflow's own exit
status. See `../TRAPS.md#verify-a-publish-by-querying-each-registry-for-the-specific-version`
for why `@latest` specifically has lagged a real, already-tagged version
before.

Mark `done` with all three real, independently-queried version numbers.

## Hop 7: hand off to write-artifacts

Onboarding ends here -- the provider has a real, published schema and
real, published SDK bindings, with zero descriptions/intros/categories
written yet (a brand new provider has no existing artifacts to inherit,
unlike a `regen-schema` run against a provider that already has some).
Run `/write-artifacts $ARGUMENTS` next, as its own separate session or
sessions -- do not attempt artifact authoring inside this same session
just because context happens to remain; a hundred intros is real work
with its own real batching discipline, covered in that runbook.

Mark this hop `done` once `/write-artifacts` has been invoked at least
once for `$ARGUMENTS`, whether or not it finished in one pass.
