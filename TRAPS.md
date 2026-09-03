# TRAPS.md -- failures that will happen again if skipped

Every trap below caused a real, confirmed failure in this project, most
of them more than once. They are listed here once and linked from the
exact runbook step where each applies, rather than repeated inline four
times.

## Verify a merge by reading real file content, not the merged flag

Four PRs showed `MERGED` in their own status while the fix they carried
was not actually present on `main` when read directly. `gh pr view`
reporting merged is a claim about git history, not about content -- read
the file, or grep the symbol, in a fresh checkout of the real branch
after the merge, before reporting anything as shipped.

## A stacked PR is only safe while its base is unmerged

Pushing more commits to a branch after its own base branch merged
strands those commits -- they land nowhere near `main`, silently, because
the branch's own diff base moved out from under it. Happened three
times in one session before this was written down.

**Freshly confirmed, this exact failure, unprompted**: a session working
on an unrelated ticket pushed a `STATE.md` update to a branch whose own
PR had merged hours earlier, in a different session, without ever
fetching first. `git status` looked completely normal -- clean tree, a
named branch, a real commit ready to push -- because a stale local
checkout cannot tell you a remote branch's PR has already merged. The
push succeeded, landed on a dead branch, and would have gone unnoticed
if the commit hadn't been the kind of thing someone would later look for
on `main` and not find. Caught only by fetching origin and running
`git merge-base --is-ancestor` after the fact, not before.

**Deleting a base branch closes dependent PRs rather than retargeting
them.** If a stack is `feature-a` -> `feature-b` (b's PR based on a),
deleting `feature-a` after its own PR merges does not rebase `feature-b`
onto `main` -- it closes `feature-b`'s PR outright, GitHub treating the
missing base as the PR no longer having anywhere to land. If a stack
must be cleaned up, retarget each dependent PR's base explicitly
(`gh pr edit <n> --base main`) before deleting anything upstream of it,
and verify the retarget landed (`gh pr view <n> --json baseRefName`)
before the delete.

**Check before any push to an existing branch**:
```
git fetch origin
gh pr list --head <branch> --state all --json number,state
```
An empty result or a `MERGED`/`CLOSED` state means stop -- start a new
branch from current `main` instead of pushing here.

## Run git status before claiming work is committed

A full batch sat uncommitted while its own summary said it shipped.
`git status --short` costs nothing and catches this every time; a
summary written from memory of intent, not from the real working tree,
does not.

## Fetch fresh before any branch operation

`git status`, `git log`, and a clean working tree all describe the
LOCAL view only. None of them can tell you a branch's remote PR merged
five minutes ago in a different session or from a different machine.
`git fetch origin` first, always, before checking out, branching from,
or pushing to anything that isn't brand new.

## Verify a publish by querying each registry for the specific version

A workflow's own green checkmark means the workflow ran to completion,
not that npm, PyPI, or the Go module proxy actually now serve the new
version -- registries have real propagation lag, sometimes tens of
seconds, sometimes longer for a specific endpoint (the Go proxy's own
`@latest` has lagged behind a real, tagged, resolvable version more than
once in this project). Query the exact version directly
(`npm view <pkg>@<version>`, a PyPI JSON API hit for the specific
release, `go list -m <module>@v<version>`), not `@latest`, and not the
workflow's own exit status.

**npm's own bare package-document endpoint
(`registry.npmjs.org/<pkg>`, no version) can 404 with a real, genuine
`{"error":"Not found"}` for a package that is, at the exact same
moment, fully live and correctly resolvable at its specific-version
endpoint (`registry.npmjs.org/<pkg>/<version>`) and its `dist-tags`
endpoint.** Confirmed live on `ubx-sdk-digitalocean`'s own first real
publish: the bare package document 404'd on repeated queries minutes
apart, while `.../1.0.0` returned the full real manifest (real
`shasum`, real signed provenance, real tarball URL) and `.../dist-tags`
correctly showed `{"latest":"1.0.0"}` the whole time. This is a real
reason on top of the propagation-lag one above to always query the
specific version (or `dist-tags`), never the bare package document --
the bare document is a different, apparently less reliably-cached
resource, and its 404 is not evidence the publish failed.

**For a scoped package, the real `dist-tags` endpoint is
`registry.npmjs.org/-/package/<url-encoded-name>/dist-tags`, not
`registry.npmjs.org/<url-encoded-name>/dist-tags`.** Confirmed live,
UBI-222: the latter 404s with `"version not found: dist-tags"` -- npm
parses the path segment after the package name as a VERSION specifier
on that route, and "dist-tags" is not one, so it fails in a way that
reads like a real "not published yet" result rather than a wrong URL.
The former is the real, correct route.

## `gh secret list` on a repo does not show org-level secrets, and reads as absent rather than as "check elsewhere"

`gh secret list --repo <owner>/<repo>` only lists secrets created
directly on that repo. An org-level secret with a "selected
repositories" visibility that includes the repo is real and live --
its own workflows can read it -- but `gh secret list` against that
same repo returns empty, with nothing in the output distinguishing
"no secret configured anywhere" from "a real secret exists at the org
level, this command just cannot see it." This has produced the exact
same real, wrong conclusion three separate times in this org:
`ubx-sdk-typescript`'s and `ubx-sdk-python`'s own earlier onboardings,
and again here with `ubx-sdk-cloudflare` (UBI-222) -- a new repo's
`gh secret list` came back empty, read as "the tokens need to be
created," when the real, only-missing piece was adding the new repo to
`NPM_TOKEN`/`PYPI_TOKEN`'s own existing org-level access list.

**The real check**: `gh api repos/<owner>/<repo>/actions/organization-secrets`
-- repo-scoped, lists exactly the org-level secrets that repo can
currently see, distinct from the org-wide listing endpoint (which
needs `admin:org` this session's own token may not have). Confirmed
live: this endpoint returned the two real tokens, with matching
creation timestamps, for `ubx-sdk-digitalocean` and `ubx-sdk-github`
(both known to have published successfully before) and an empty list
for `ubx-sdk-cloudflare` -- proving the tokens were real, org-level,
and selected-repositories-scoped, not missing, before a single new
token needed creating. Run this before ever concluding a secret
"doesn't exist" for a repo that a plain `gh secret list` came back
empty on.

## Build from current source, not whatever binary is in the path

A stale `ubx` on `PATH` produces a result that looks identical to a
correctly-fixed one until the specific bug the fix addressed is tested
directly -- and by then the false confidence has already been reported.
Rebuild (`make build` / `make install` in `ubiquex`, which prints
`ubx version` immediately after so this is one step, not a separately
remembered one) and confirm the printed commit matches the fix's own
commit before trusting any result against it.

## A spec existing is not the same as it loading

A provider publishing a real, public, machine-readable spec is the
first real check, not the last one. DigitalOcean's own spec is genuine
OpenAPI 3.0, confirmed reachable by direct fetch -- and still failed to
load through `ubx-provider-dynamic`'s own `openapi.Load` outright, on
two Redocly-only `$ref` conventions real OpenAPI 3.0 does not define (a
Tag Object's own `description` expressed as `$ref`; every Path Item's
own HTTP-method field expressed as `$ref` to a separate per-operation
file). Run the real loader against the real spec before committing to
anything downstream of "this provider has a spec" -- a URL returning 200
is not the bar.

## A GitHub secret-scan block on vendor content needs a human click

Pushing a schema snapshot that happens to contain a vendor's own real
documentation example (a placeholder API key, a sample webhook URL) can
trip GitHub's push protection even when nothing live is in it.
DigitalOcean's own `v1.0.0` was blocked this way twice, at two locations
each in two files, on a Slack Incoming Webhook URL PATTERN that turned
out to be `https://hooks.slack.com/services/T1234567/AAAAAAAA/ZZZZZZ` --
textbook placeholder tokens straight out of DigitalOcean's own public
spec, not a live credential. There is no CLI or API path around this for
a repo without the org's own secret-scanning bypass permission -- it
needs a human to open the real, specific unblock URL the rejected push
itself printed and allow the finding. Budget for this as a possible
manual gate on the FIRST push to any new schema repo, and never scrub
legitimate spec content to work around it without first confirming, by
reading the flagged bytes directly, that it's genuinely not live.

## Don't trust a prior session's own "published" claim

`ubx-schema-digitalocean`'s own first `HISTORY.md` entry is titled
"repo created, v1.0.0 published" and says the push succeeded -- written
before the push was ever confirmed, and the push had in fact been
rejected outright by the secret-scan block above. `HISTORY.md` is a
narrative archive, never rewritten once written, so the wrong claim is
still there; the correction lives in `STATE.md`'s own "In flight"
section instead, which says so explicitly rather than repeating the
claim. Before trusting any repo's own "published"/"merged"/"pushed"
language, verify against the real remote directly (`git fetch` and
`git merge-base --is-ancestor`, or a registry query per the trap above)
-- a repo's own state files describe intent at the time they were
written, not necessarily what actually landed.

## A hardcoded provider allowlist is invisible until a genuinely new provider arrives (Mintlify-only, UBI-240)

**This entire trap category does not apply to a provider using
`write-artifacts`'s own SDK-repo target or `ubx-docs-providers`.**
Every dict named below belongs to `ubiquex-docs`'s own Mintlify
generation pipeline specifically -- `coverage_check.py`'s own
`--artifacts-root` flag (UBI-240) skips the registry lookup entirely
for that path, and `ubx-docs-providers` has no per-provider dict of any
kind, only one committed `config/providers.json` a provider is either
listed in or isn't. This trap is real, confirmed history, worth
keeping for exactly the case it still applies to: `/regen-mintlify-docs`
(renamed from `/regen-docs`), run only when a provider is being given
Mintlify coverage on purpose, never as part of a new provider's own
default path.

Six of the six existing providers this whole chain has ever run against
were added to `ubiquex-docs`'s own generator scripts one at a time, over
a long history, each addition folded quietly into whatever dict already
existed. Nothing about that history ever forced anyone to notice the
dicts themselves are a hardcoded enumeration, not derived from anything
live -- every one of the six providers was always already there by the
time any of these scripts were read or run, so the gap had no occasion
to surface.

DigitalOcean, the seventh, hit it repeatedly, in two separate hops:

`write-artifacts`: `coverage_check.py`'s own `DUMP_DIR` dict failed
outright with "unknown provider" before anything could even be
recomputed -- the hop's own core command, unusable until fixed.

`regen-mintlify-docs`: five more, each a real crash or a real silent skip --
`gen_all_data_source_pages.py`'s `PROVIDERS` (`KeyError`),
`regen_all.py`'s `RESOURCE_REGEN_PROVIDERS`/`ALL_PROVIDERS` (would have
silently produced data-source pages only, never real resource pages --
this is the gate deciding whether `regen_pages.py` even runs, so the
failure is silent, not a crash), `regen_pages.py`'s
`PROVIDER_DISPLAY`/`SDK_REPO_ID`/`SKIP_INJECTION_SOURCE` plus a second,
local `sdk_repo_id` dict duplicating the module-level one, and
`corpus_index.py`'s `PROVIDER_TAB_NAMES` (`KeyError` in
`provider_group`). See `ubiquex-docs` PR #61 for the real fix, and
`ubiquex` `STATE.md`'s own UBI-222 entry for the full account.

A close relative, same root cause, different shape: `docs.json` itself
carried no "DigitalOcean" nav group at all, and `provider_group`/
`rebuild_provider_nav` have no create-if-missing path by design (they
reconcile an already-existing group against the real file tree, never
invent one) -- every provider this mechanism had ever run against
before already had its own group from an earlier, separate bootstrap
nobody had to think about since it predated the mechanism itself.

None of this is DigitalOcean-specific. It is what happens to any
genuinely new provider run through any of these scripts for the first
time -- **before touching `write-artifacts` or `regen-mintlify-docs` for a
provider that has never been through either hop before**, grep each
script above for its own dict literal and confirm the new provider's
key is actually present; do not assume a script "supports all
providers" just because it has no per-provider `if` branches -- an
enumerated dict with no such branch fails exactly as hard as one that
has them, just later and less legibly (a `KeyError` deep in a call
stack, or worse, a silent partial run with the wrong pages simply never
generated). Same for `docs.json`'s own nav groups: confirm the
provider's own group already exists before assuming `regen-mintlify-docs` will
create it.

## A cold cache exercises code paths a warm one never does

Two real bugs sat in `ubx-docs-providers`'s own fetch/build scripts for
as long as they existed, both invisible for the same reason: every
fetch against them, across this whole project's history, had a warm
`.docs-cache` or a local `UBX_DOCS_MIRROR` satisfying the request, so
the real "download from a GitHub Release and verify it" path had
genuinely never run end to end before UBI-245 forced a first real
fetch with an empty cache.

**`SHA256SUMS`'s own recorded filename is a full CI runner path, not a
bare filename.** Every `ubx-sdk-<provider>` `publish.yml` runs
`sha256sum "${{ runner.temp }}/docs.tar.gz" > SHA256SUMS`, and
`sha256sum` echoes back exactly the path it was given -- so the real,
live file reads `<digest>  /home/runner/work/_temp/docs.tar.gz`, not
`<digest>  docs.tar.gz`. `fetch-docs.mjs`'s own checksum parser matched
by exact string, which made every real release un-fetchable the moment
there was no cache entry to fall back on. Match by basename, not exact
string, when parsing a `SHA256SUMS` line this project did not generate
under its own control.

**A subprocess formatting hundreds or thousands of generated files can
exceed a 1MB default stdout/stderr buffer before it ever produces a
real error.** `build-examples.mjs`'s own `gofmt`/`deno fmt` calls used
`execFileSync`'s default `maxBuffer` (1MB) -- fine at a few thousand
examples, not once real coverage across old and new versions together
crossed 24,000. Both tools print real per-file output as they walk a
directory, and that output alone crossed the buffer limit before
either tool reached a genuine syntax problem, surfacing as a truncated,
misleading "rejected generated source" error with no real message at
the end of it. A build that only ever ran against a small, single-
version fixture had no occasion to hit this; raise the buffer
generously (not tuned to today's exact count) the first time a real
multi-version, real-scale build is attempted, rather than waiting to
discover the limit live.

## A script's stdout looking right to a human is not the same as its being parseable

A script whose own doc comment promises a specific machine-readable
output (pure JSON on stdout, for one real example) can still interleave
real, readable log lines ahead of that output from a function it calls
-- invisible to a human skimming the terminal, since the JSON is still
there, just not alone. Nothing caught this until an actual caller fed
the real stdout through `json.loads` and it failed on the first
character. If a runbook step's own output is meant to be consumed by
another step or another session, verify the literal contract (parse it
the same way the real consumer will), not just that the output looks
right.

**This exact bug recurred for real in `regen_all.py`**, the actual
orchestrator this whole `/regen-mintlify-docs` chain runs -- its own final JSON
report shared stdout with regen_pages.py's/gen_all_data_source_pages.py's
own uncaptured narration, and CI's own "Build the step summary" step
broke on it twice, both times on a real, successful DigitalOcean
regeneration, reporting the whole run as a red failure. A red workflow
that is actually fine, twice, is exactly what trains everyone to stop
looking at it -- the same real problem the golden-page-gate had before
its own known failure mode was fixed. The durable fix was not "keep
stdout pure by discipline" (the same discipline that already failed
once for `stage_gap_free.py`) but moving the report off stdout
entirely -- a real, required `--json-out` file path, decoupled from
whatever any current or future line in the chain prints. Prefer this
shape (an explicit output file, never an implicit stream) for any new
script in this chain whose output another step or session needs to
parse.

## A required check with zero possible runs blocks a PR forever, and looks like a real failure

Branch protection required a `stale-base-check` status context on six
real repos whose own workflow file producing that check did not exist
yet -- a real, permanent block, not a slow CI run or a flaky one. From
outside the repo, a PR sitting unmergeable with an empty check list is
indistinguishable from a real failure until the branch protection
rules and the actual workflow files are checked directly against each
other. Confirmed the fix works before assuming it: a newly-added
required-check workflow can pass on the very PR that adds it (GitHub
evaluates a `pull_request`-triggered workflow using the file as it
exists on the PR's own head), and a branch whose own commits predate
that workflow landing on the base needs one more push (a rebase or a
merge-in of the base) before its own check can run at all -- not a new
file, just a fresh commit.

Also found here: "never self-merge" is not one rule with no
exceptions. It draws a real, closed line between a PR with content to
judge (schema changes, generated output, description or page content,
anything that changes what ships) and a PR that is purely mechanical
(an identical file copied verbatim across repos, a rebase or merge-in
with no new content, a version bump with no content change). Before
this line was written down explicitly, seventeen real, correctly
prepared, passing-checks PRs across nine repos sat unmerged waiting
for a human simply because nothing distinguished them from the PRs
that genuinely needed review. The line has to be written so checking
it is mechanical too -- if a diff has anything outside the closed list,
even one line, it needs review as a whole; if it is unclear which
category a PR falls into, it needs review.

## A 10% random sample across a member group can miss every real hit

UBI-222's own cross-provider check for the `findCreate` inline-envelope
false-create-matching bug (see hop 8's own notes) sampled 30 of
Azure's 302 real non-`_ds` members (seeded random) to check exposure
without running all 302 -- came back completely clean, reported as
such. Running the real, exhaustive 302-member check afterward (not
much more expensive: ~1s per member, parallelizable) found 13 real
affected members (4.3%), 46 real affected resources -- the 30-sample
missed every single one, not by a small margin. The affected members
were not scattered evenly; nine of the thirteen were all inside one
service family (`apimanagement`), which a small random sample across
302 members has real, structural odds of missing entirely (a cluster
this size is easy to draw zero of at n=30) even though the true
provider-wide rate (4.3%) is not actually rare. Lesson: for a
member-grouped provider (Azure's own real per-service-spec shape),
"sample a fraction and report clean" is not a substitute for the real
exhaustive check when the check itself is cheap -- reserve sampling for
when exhaustive is genuinely infeasible (very large N, expensive
per-item cost), and even then, report the sample's own real confidence
bound, not just "N of M checked, clean."

## A scratch probe against a provider with real external `$ref`s must call Bundle, not just Load

`internal/openapi.Load` alone resolves a spec's own top-level document
and any relative `$ref` it can reach against the source's own real
location, but does not perform the separate, explicit `Bundle()` pass
`internal/snapshot/generate.go`'s own real generation path always
applies afterward for a provider whose spec references *other* files
(`internal/openapi.Bundle`'s own doc comment has the full real reason
-- Azure's own real ARM specs split themselves across shared files by
relative path, e.g. `../../../../../../common-types/resource-management/
v5/types.json#/definitions/ProxyResource`, and Azure's own ref graph is
genuinely cyclic, not just deep). A scratch tool built to probe
`resourcemap.Discover()` behavior that calls `Load` alone against a
live URL, without the follow-up `Bundle()` call, produces a REAL,
DIFFERENT result for any such provider -- not an error, not a crash,
a plausible-looking but wrong Discover() output, because an external
`$ref` that never gets resolved into a real local named component
shows up differently to `findCreate`'s own `$ref`-identity check than
it would once bundled, pushing genuine same-schema create/read pairs
into the wrong matching branch or missing them outright.

**Confirmed live, UBI-222, at real cost**: a cross-provider exposure
check for the `findCreate` inline-envelope/path-relationship bugs used
`Load(liveURL)` alone against Azure's real specs (both the sampled and
the later exhaustive 302-member sweep) and reported 16 affected members
(53 resources) with wrong or falsely-creatable bindings. A real
`ubx-provider-dynamic` release was cut and an `ubx-schema-azure`
regeneration PR opened on the strength of that finding before it was
double-checked. Re-running the exact same 302-member comparison the
correct way -- reloading each member's own already-committed, already-
bundled `raw_spec` via `openapi.Parse(rawSpec, nil)` (the identical
real code path `internal/snapshot/generate.go` itself uses to reload a
frozen member, confirmed by reading that file directly rather than
assumed) instead of re-fetching and `Load`-ing the live URL -- found
**zero** real differences across all 302 members. Every one of the 16
originally-flagged members turned out to be a real, legitimate
`$ref`-identity match (the SAME named schema shared by a read and its
own real create, exactly the primary, strongest signal `findCreate`
already trusts, untouched by either of the two path-relationship/
envelope fixes) that the unbundled probe's own broken `$ref` resolution
had scrambled into looking like a false positive.

**The real check for any provider whose spec references other files**:
either call `openapi.Load` immediately followed by `openapi.Bundle(doc)`
(matching `generate.go`'s own real two-step sequence exactly), or --
simpler and less error-prone -- reload an already-committed member's
own `raw_spec` field via `openapi.Parse(rawSpec, nil)`, since that
content was already bundled once at generation time and needs no
second pass. A provider whose real spec is a single self-contained file
with no external refs (Cloudflare's own real case, confirmed this same
session -- "no redocly_bundle needed") is not exposed to this trap, but
nothing about a scratch tool's own output tells you which kind of
provider you're looking at, so treat this as required for any
multi-file spec, not just Azure's.
