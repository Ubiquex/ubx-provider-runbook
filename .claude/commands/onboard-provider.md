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

**Before moving on, look at the real service/namespace grouping this
run produced, not just the resource count.** A provider whose own wire
names don't cleanly split into `<provider>_<service>_<noun>` will
mechanically fragment into near-meaningless single-resource "services"
that nothing downstream catches automatically -- DigitalOcean's own
real example: nearly all 59 resources landed in their own
single-resource service (`load`, `floating`, `reserved`, `db`,
`database`, `cluster`, `pool`, `replica`, `config`...) instead of its
real product taxonomy, discovered only by a human actually reading the
coverage report's own resource list during `write-artifacts`, several
hops later than it should have been. Check this now: does the spec
carry genuine, human-authored operation tags or an equivalent grouping
signal (`internal/snapshot`'s own `namespace_from_tags` config flag
reads exactly this for OpenAPI sources, opt-in per provider, gated so
it never risks regressing an already-correct provider) that the
mechanical wire-name split is missing? If the grouping already looks
wrong here, fixing it now is far cheaper than fixing it after
artifact-authoring has already been batched against the wrong
services.

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
`ubiquex`'s own `[dynamic_providers.$ARGUMENTS]` entry from its live
shape (`schema_source`/`schema_url`, plus any provider-specific flag
hop 3 needed, e.g. `redocly_bundle`) to its pinned shape (`source`/
`version` pointing at the real release) -- **within that SAME table,
never a separate `[providers.$ARGUMENTS]` table.** The real, current
config only ever carries one `[dynamic_providers.<name>]` entry per
provider; a bare `[providers.<name>]` table is a genuinely different,
newer mechanism (`cli/config.go`'s own `Providers` field, for the `ubx
resolve`/`ship` execution path) that `ubx sdk gen` never reads at all.
This section previously said otherwise and cost a real session real
time tracing the mismatch before the config file's own comment (already
correct) settled it -- see DigitalOcean's own onboarding manifest,
`release-cut-and-pin` hop. Real PR, verified merged by reading the
config file's own content after, not the merged flag.

## Hop 6: generate and publish SDK bindings

Set `UBX_PROVIDER_DYNAMIC_REPO` locally before running `ubx sdk gen`
against a dynamic provider -- CI sets this via `env:`, but a session
running this by hand has no equivalent instruction anywhere else in
this runbook, and hits a real, confusing failure without it.

`ubx sdk gen --only $ARGUMENTS --lang go/py/ts --out <dir>` against the
now-pinned schema does NOT, by itself, produce a complete, publishable
repo -- confirmed live, DigitalOcean's own real onboarding, twice (a
first missing `deno.json` broke a real `deno check`, a first missing
`build-npm.mjs` broke a real `publish.yml` dispatch with
`MODULE_NOT_FOUND`) before this was fixed at the tool level rather
than left as a checklist a future session could miss the same way:

- **`ubx sdk gen`'s own output**: the three language SDKs, plus
  `sdk/go/go.mod`, `sdk/typescript/package.json`, and
  `sdk/python/pyproject.toml` as placeholder stubs carrying version
  `0.0.0`. `sdk/typescript/deno.json` is now ALSO real, per-provider
  generated output (its own "exports" map, derived from the same file
  tree as everything else) -- no longer a hand-copied file, this was
  the first of the two real DigitalOcean gaps.
- **`ubx sdk init-repo --out <dir>/$ARGUMENTS --short-name $ARGUMENTS
  --provider-display "<Display Name>" --source-note "<one real,
  honest sentence>"`, run once, right after the `--lang go/py/ts`
  calls above.** Writes everything else a new repo needs and `ubx sdk
  gen` still doesn't produce: `LICENSE`, `.github/scripts/build-npm.mjs`,
  `.github/workflows/publish.yml`, `CLAUDE.md`, `README.md`, `STATE.md`,
  `HISTORY.md`, and a real `sdk/go/go.sum` via an actual `go mod tidy`
  subprocess run. Never overwrites a file that already exists (safe to
  re-run). `--source-note` is the one real judgment call this command
  can't make for you -- one honest sentence on this provider's own
  schema source and format (e.g. `"OpenAPI-sourced via
  \`ubx-provider-dynamic\`"`; Datadog's own real v1/v2 API merge is the
  confirmed example of a provider needing something more specific than
  the generic default).
- **`.github/workflows/hash-watch.yml` is the one file this command
  deliberately does NOT write** -- it needs real, provider-specific
  content (which upstream URL to poll, how to hash it) that differs
  genuinely per provider, not mechanical substitution. Author it by
  hand, using an existing provider repo's own copy as a structural
  reference, not a copy-paste source.
- **A real, deliberate first version, chosen before the first PR, not
  left at `0.0.0`.** Every one of the six pre-existing providers
  inherited its own starting version from a prior Terraform-provider
  migration (kubernetes started at `0.2.0`, not `0.0.0` or `1.0.0`) --
  that is real history specific to those repos, not a convention to
  copy. A genuinely from-scratch provider has no such history; match
  the schema pin's own real starting version (`1.0.0` for
  DigitalOcean) as the deliberate choice, and bump
  `sdk/python/pyproject.toml`, `sdk/typescript/deno.json`, and
  `sdk/typescript/package.json` together (Go needs no file bump --
  `publish.yml`'s own design publishes it via a pushed tag).
- **`NPM_TOKEN` and `PYPI_TOKEN` as real, dedicated repo secrets**
  (Settings -> Secrets and variables -> Actions), scoped to this one
  new repo -- `publish.yml`'s own doc comment already says these are
  "dedicated CI tokens, scoped to this repo/package," never shared or
  inherited from another repo. A brand new provider genuinely needs
  its own newly-created tokens; only the real npm/PyPI account owner
  can create and add them. The first `publish.yml` dispatch against a
  repo missing these fails with a real `ENEEDAUTH` (a genuine auth
  failure, distinct from `MODULE_NOT_FOUND` above) -- confirm this
  step actually happened before dispatching, since `ubx sdk init-repo`
  cannot do this part; it needs a real credential only a human has. If
  it fails with `ENEEDAUTH` even after the secrets are added, check
  whether they were added as ORG-level secrets with a repository
  access list that doesn't include this new repo yet -- confirmed real
  precedent, `ubx-sdk-typescript`'s own earlier onboarding hit exactly
  this.

Create `ubx-sdk-$ARGUMENTS` (public), commit the generated bindings
plus every item above, push, open a PR. Once merged, dispatch that
repo's own `publish.yml`.

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
