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

**Set `UBX_PROVIDER_DYNAMIC_REPO` (or pass `--dynamic-provider-bin`)
before ANY real `ubx sdk gen` invocation against `$ARGUMENTS`, pinned
or live** -- confirmed live, UBI-222: this was previously noted only
under hop 6, but a pinned provider's own `ubx sdk gen` run needs a real
`ubx-provider-dynamic` binary just as much as a live one does (schema
RESOLUTION is zero-network once pinned; turning that resolved schema
into IR/generated code still runs through this binary). Without it,
every hop from 3 onward fails outright with "no ubx-provider-dynamic
checkout found." Prefer `--dynamic-provider-bin` pointed at a real,
downloaded, checksum-verified release binary over
`UBX_PROVIDER_DYNAMIC_REPO` (a local source build) whenever verifying
a hop's own real, final behavior -- the release binary is the exact
real artifact a real install would acquire, not just source that
happens to match it.

**This whole runbook accumulates commits on one long-lived branch
across a run that can span hours or multiple sessions -- confirm the
branch's own PR is still open before every push, not just the first
one.** Confirmed live, UBI-222: hops 1-3's own PR merged mid-run while
this session kept working, and the next hop's own commit landed on the
now-dead branch, invisible from `main`, exactly
`../TRAPS.md#a-stacked-pr-is-only-safe-while-its-base-is-unmerged`'s
own real, prescribed failure shape. `gh pr view <n> --json state`
before every push to a branch already carrying an open PR; if it shows
`MERGED`, start the next hop's commit on a fresh branch off current
`main` instead.

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
method already used for 17 real candidates is `docs/candidate-provider-survey.md`
in `ubiquex` (not `sdk/providers/docs/` -- confirmed live, UBI-222:
the path this section named did not exist) -- fetch the real spec, run
the real
`internal/resourcemap.Discover`/`DiscoverDataSources` against it, record
real counts. A spec existing with no fetchable URL, or generated from a
private spec never published for external fetch, is a real "no" --
confirmed per-candidate, not assumed from a repo's README alone.

**Build the scratch tool inside `ubx-provider-dynamic`'s own module
tree** (a temp package under `cmd/`, deleted after this hop), not as a
separate module with a `replace` directive pointed at a local checkout
-- confirmed live, UBI-222: Go's own internal-package visibility rule
blocks `internal/openapi`/`internal/resourcemap` from any importer
outside that module's real root, and a `replace` directive does not
change the importing module's own path prefix, so the separate-module
shape fails to build outright before ever reaching a real spec.

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

**Also check the real NOUN quality, one level below service
grouping** -- confirmed live, UBI-222, Cloudflare's own real spec:
`deriveNoun`'s primary path (the response schema's own component name,
`resourcemap.go`'s own doc comment calls this "the strong, real
signal") is only as good as how distinctly the spec's own authors name
their response schemas. Cloudflare reuses a small set of generic
response-envelope schema names (`single_response`, `id_response`, and
similar) across hundreds of structurally distinct real resources
instead of giving each its own uniquely named component -- real,
correctly-discovered, genuinely distinct resources, disambiguated only
by an appended collision counter (`cloudflare_access_single_response_2`
through `_22`, fifteen real, different Access sub-resources). Not a
loader bug and not a grouping bug -- a real, provider-specific
naming-quality issue that will make categorization and description
authoring materially harder for however large a fraction of the
provider's own wire types it affects. Count it now: how many discovered
`TypeName`s match a generic-wrapper-name shape (a `_response`/`_result`/
`_resp`/collection-wrapper suffix, optionally followed by a bare
collision-counter digit)? Cloudflare's own real number: 34.5% of
resources, 44.8% of data sources. There is no fix to apply here at this
hop -- record the real fraction in the manifest so `write-artifacts`
starts from a real expectation instead of discovering the same pattern
resource by resource, the same reason the service-grouping check above
exists.

## Hop 4: create and push the schema repo

**Not `ubx sdk gen --dump-ir`** -- confirmed live, UBI-222: that flag
produces the docs/description IR shape (one JSON file per wire type
plus a combined `schema.json`), a genuinely different, simpler format
than a real `ubx-schema-$ARGUMENTS` snapshot. The real generation
command is `ubx-provider-dynamic`'s own `--generate-snapshot-group`,
run directly against that binary (not through `ubx sdk gen` at all):

```
UBX_DYNAMIC_PROVIDER_NAME=$ARGUMENTS ubx-provider-dynamic \
  --generate-snapshot-group <out-dir> \
  --group-repo-name $ARGUMENTS \
  --group-members $ARGUMENTS
```

Run from a directory whose own `.ubx/config` carries
`[dynamic_providers.$ARGUMENTS]` (see below for why this can't be
`sdk/providers/` itself). One `--group-members` name, not two --
`ExpandMemberModes` automatically expands a single
`[dynamic_providers.$ARGUMENTS]` table into both real members
(`$ARGUMENTS`, resource mode, and `$ARGUMENTS_ds`, data-source mode),
writing `manifest.json` + `members/*.json` to `<out-dir>`, matching
every other real `ubx-schema-<provider>` repo's own shape.

**Run this from a scoped, temporary `.ubx/config` carrying ONLY
`$ARGUMENTS`'s own table, never `sdk/providers/.ubx/config` directly.**
Confirmed live: `ubx-provider-dynamic`'s own `internal/config.Load`
(what `--generate-snapshot-group` uses to read `[dynamic_providers.*]`)
requires `schema_source` on every table it parses, unconditionally --
it has no notion of the PINNED shape (`source`/`version`) at all, that
shape is understood only by `ubiquex`'s own `cli/dynamicprovider.go`.
Pointing generation at the real monorepo config, which by this point
carries six or seven already-pinned entries alongside `$ARGUMENTS`'s
own live one, fails outright (`dynamic_providers.<already-pinned-name>:
schema_source is required`) before it ever reaches `$ARGUMENTS`.
Copy just the one real `[dynamic_providers.$ARGUMENTS]` table (the
same content hop 2 committed) into a scratch `<dir>/.ubx/config` and
run generation from there instead.

**The generating binary must be a real, released `ubx-provider-dynamic`
version, not a local build off an unmerged fix.** Confirmed live:
`--generate-snapshot-group` refuses to write a snapshot whose own
`min_binary_version` would be the unstamped `"dev"` default (a
dev-stamped snapshot can't be traced to any real, acquirable binary),
and a locally-built binary is always `dev`-stamped unless it's an
actual tagged release checkout. If hop 3 needed a real
`ubx-provider-dynamic` fix (a real, common shape -- see hop 3's own
notes), that fix must be reviewed, merged, and a new version actually
published (this repo's own `publish.yml`, same discipline as any
`ubx-sdk-<provider>` release) BEFORE this hop can produce a real,
committable snapshot -- `--allow-dev-binary` exists for local
iteration only, never for anything meant to be committed. This makes
hop 4 a genuine, hard dependency on hop 3's own fix landing for real,
not just being written -- budget for a real pause here waiting on
review, the same as any other PR this runbook can't self-merge.

Once the snapshot is real: create the repo (public, matching every
other real schema repo), commit the generated snapshot, push.

**A real `ubx-schema-$ARGUMENTS` repo needs far more than
`manifest.json` + `members/*.json`** -- confirmed live, UBI-222: this
section named only the generated snapshot, the same real gap hop 6
already documents for an SDK repo ("a real `ubx-sdk-<provider>` repo
also needs CLAUDE.md, README.md, ..., none of which `ubx sdk gen`
produces"), never previously written down for THIS hop. None of the
following is produced by `--generate-snapshot-group` -- copy an
existing, working `ubx-schema-<provider>` repo's own copy of each
(one that shares `$ARGUMENTS`'s own real shape: no `redocly_bundle`
needed, an openapi source, matching `ubx-schema-github`'s own real
copies most closely as of UBI-222), adapting only the real,
provider-specific content (names, real counts, the real schema URL,
any real judgment call like `namespace_from_tags`), never blindly
find-and-replacing prose -- a naive "github" -> "$ARGUMENTS" substring
replace corrupts real prose that names an unrelated real thing sharing
the same substring (confirmed live: "GHEC (GitHub Enterprise Cloud)"
became nonsense the first blind pass through):

- `LICENSE`, `README.md`, `CLAUDE.md`, `STATE.md`, `HISTORY.md`.
- `.github/workflows/hash-watch.yml` -- the real, weekly regeneration
  workflow. Offset its own cron day from every other real schema
  repo's own slot, so a shared runner queue doesn't stack every
  provider's own regen on the same day.
- `.github/workflows/publish.yml` -- cuts the real release hop 5 needs.
- `.github/workflows/ci.yml` -- validates the committed snapshot
  actually loads on every push/PR, confirmed fully generic (no real
  provider-specific content) across every existing schema repo.
- `.github/workflows/orphan-branch-watch.yml`,
  `.github/workflows/stale-base-check.yml`,
  `scripts/orphan_branch_check.py` -- also confirmed fully generic,
  copy verbatim.

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
  honest sentence>" --schema-pin-version "<the real, current pin
  version from hop 5>"`, run once, right after the `--lang go/py/ts`
  calls above.** Writes everything else a new repo needs and `ubx sdk
  gen` still doesn't produce: `LICENSE`, `.github/scripts/build-npm.mjs`,
  `.github/workflows/publish.yml` (UBI-240: now includes the real
  docs-artifact step, templated -- no separate hand-added step needed
  the way every provider onboarded before this got), `CLAUDE.md`,
  `README.md`, `STATE.md`, `HISTORY.md`, and a real `sdk/go/go.sum` via
  an actual `go mod tidy` subprocess run. Never overwrites a file that
  already exists (safe to re-run). `--source-note` is the one real
  judgment call this command can't make for you -- one honest sentence
  on this provider's own schema source and format (e.g.
  `"OpenAPI-sourced via \`ubx-provider-dynamic\`"`; Datadog's own real
  v1/v2 API merge is the confirmed example of a provider needing
  something more specific than the generic default). `--schema-pin-version`
  is hop 5's own real, current pin version, read directly from
  `sdk/providers/.ubx/config`'s own `[dynamic_providers.$ARGUMENTS]`
  entry -- baked into the generated `publish.yml`'s own schema-dump
  step. A provider whose own upstream has no discrete pinnable release
  (Kubernetes is the one real exception in this org, its own OpenAPI
  spec fetched unpinned from a live branch tip) needs that one
  generated step hand-adjusted afterward to a live `schema_source`/
  `schema_url` shape -- see `ubx-sdk-kubernetes`'s own `publish.yml`
  for that real, working shape.
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
plus every item above (including empty `artifacts/$ARGUMENTS/` --
`descriptions.json`/`intros.json`/`categories.json`/`exclusions.json`,
each `{}` or the equivalent empty shape -- publish.yml's own
docs-artifact step reads these files unconditionally and fails if
they're missing entirely, even though a brand new provider has nothing
real in them yet), push, open a PR. Once merged, dispatch that repo's
own `publish.yml`.

**Verify the publish by querying each registry for the specific
version** -- npm, PyPI, the Go module proxy, each queried for the exact
new version string, never `@latest` and never the workflow's own exit
status. See `../TRAPS.md#verify-a-publish-by-querying-each-registry-for-the-specific-version`
for why `@latest` specifically has lagged a real, already-tagged version
before. **Also verify the docs release** (UBI-240): `gh release view
v<version> --repo Ubiquex/ubx-sdk-$ARGUMENTS --json assets` carries
exactly `docs.tar.gz` and `SHA256SUMS` -- this is the same publish.yml
dispatch, not a separate action, but a real, independent thing to
confirm rather than assume from the workflow's own green checkmark.

Mark `done` with all four real, independently-queried results (npm,
PyPI, Go proxy, docs release).

## Hop 7: bring the provider onto the docs site

`ubx-docs-providers` (providers.ubiquex.io) is the real, current docs
site (UBI-240) -- a genuinely new provider goes here, not Mintlify
(see hop 8's own note on why not). This hop is small and does not need
its own session: add one entry to `ubx-docs-providers`'
`config/providers.json` (name, description, repo, tier, the one real
version from hop 6), matching every existing entry's own shape. Real
PR, never self-merged, matching that repo's own convention.

The site will render the provider correctly with zero descriptions
authored -- DigitalOcean's own real first migration had 0 AI-inferred
fields and served real pages regardless. Confirm live: a real `npm run
build` in `ubx-docs-providers` (or, once merged, the real deployed
site) shows a real provider home page and at least one real resource
page for `$ARGUMENTS`, not just a config diff that looks plausible.

Mark this hop `done` with the merged PR URL once confirmed live.

## Hop 8: optional -- richer descriptions, and Mintlify only if asked for

The provider is real and live on `ubx-docs-providers` after hop 7.
Everything past this point is optional, additive polish, never
required to consider onboarding finished:

- **`/write-artifacts $ARGUMENTS`** (its own runbook, rewritten for
  UBI-240 to target this new SDK repo directly) writes real descriptions,
  intros, and categories in place of the empty stubs hop 6 committed --
  richer pages, not a correctness requirement. Run it as its own
  separate session or sessions; a hundred intros is real work with its
  own real batching discipline, covered there.
- **`/regen-mintlify-docs $ARGUMENTS`** (renamed from `/regen-docs`,
  UBI-240) is Mintlify-specific -- it generates `resource-reference/`
  pages for the OLD docs.ubiquex.io site, the surface UBI-240 is moving
  providers off of, not onto. Run it only if there is an explicit,
  separate decision to also carry this provider on Mintlify -- never as
  part of a genuinely new provider's own default path. It brings back
  every one of `../TRAPS.md#a-hardcoded-provider-allowlist-is-invisible-until-a-genuinely-new-provider-arrives`'s
  own hardcoded allowlists, none of which `ubx-docs-providers` has.

Neither hop needs to be marked `done` in this runbook's own manifest --
onboarding itself is complete as of hop 7.
