# MANIFEST.md -- the resume mechanism

Every runbook writes a small JSON manifest, one file per target
(provider name, or provider+schema-version for regen), recording which
hop was reached and its real outcome. A session invoked with the same
runbook and the same target reads this file first, reports what's
already done, and continues from the first hop that isn't -- it never
restarts a hop already confirmed complete, and it never trusts a hop
marked complete without a fresh check (see "manifest records history,
never the resume point" below).

## Shape

```json
{
  "runbook": "onboard-provider",
  "target": "digitalocean",
  "started": "2026-08-29",
  "updated": "2026-08-30",
  "hops": [
    {
      "id": "spec-discovery",
      "status": "done",
      "note": "real, public OpenAPI 3.0 spec confirmed reachable"
    },
    {
      "id": "live-entry",
      "status": "done",
      "commit": "https://github.com/Ubiquex/ubiquex/pull/30"
    },
    {
      "id": "loader-check",
      "status": "blocked",
      "note": "two Redocly-only $ref conventions, real OpenAPI 3.0 does not define either -- see TRAPS.md#a-spec-existing-is-not-the-same-as-it-loading",
      "resolved_by": "https://github.com/Ubiquex/ubx-provider-dynamic/pull/39"
    }
  ]
}
```

`status` is one of `done`, `blocked`, `skipped` (with a `note` saying
why it doesn't apply to this target), or absent entirely (not yet
reached -- an absent hop is not the same as a failed one). `note` and
`commit`/`resolved_by` are free text and a real URL respectively, always
pointing at something a reader could open and check, never a bare
claim.

## Where each runbook's own manifest lives

The manifest is committed alongside the real work each hop produces --
never a separate side channel, never `/tmp` (which does not survive
between sessions or machines). Reviewable in the same PR as the work
itself.

- **onboard-provider**: `ubiquex`, at
  `sdk/providers/.onboarding/<provider>.json`. Onboarding starts before
  the provider's own schema or SDK repos exist, and `ubiquex` is the
  one real coordinating repo present from hop one.
- **regen-schema**: the provider's own `ubx-schema-<name>` repo, at
  `.regen-manifest.json` in the repo root, alongside that repo's own
  `STATE.md`.
- **write-artifacts**: `ubx-sdk-<provider>` (UBI-240 -- moved from
  `ubiquex-docs`' own `scripts/resource-reference-gen/artifact-manifests/<provider>.json`,
  where every provider's own artifacts used to live), at
  `.artifact-manifest.json` in the repo root. Records batch history
  (batch number, wire count written, timestamp) for narrative purposes
  only -- what remains is always recomputed live (see below), never
  read from this file's own running count. **The migration off the old
  location was documented here but never actually carried out**,
  confirmed live, UBI-222: neither `ubx-sdk-digitalocean` nor
  `ubx-sdk-github` had this file at all before Cloudflare's own
  categories batch created the first real one, matching this shape.
  Check a target provider's own repo directly before assuming this
  file already exists there -- its absence is not evidence no work has
  happened, only that this specific tracking file was never written.
- **regen-mintlify-docs** (renamed from **regen-docs**, UBI-240 --
  Mintlify-only, not part of a new provider's own default path): no new
  manifest -- this runbook is the manual/scoped path through the same
  real mechanism UBI-137's own automation already runs in CI
  (`regen_all.py`). Its own JSON report, written to
  `/tmp/regen-scratch/<provider>_regen_result.json` and
  `<provider>_datasource_regen_result.json` during a run, already
  records per-wire coverage results and page paths -- reuse it, don't
  duplicate it. See `regen-mintlify-docs.md` for why this one hop's
  manifest is intentionally ephemeral where the other three are not: a
  docs regeneration is re-run from a fresh schema pin every time, never
  resumed against a stale one.

## The manifest records history, never the resume point

`write-artifacts` in particular: a hundred intros will not fit in one
session, and a resuming session must know exactly which wires still
need one. That answer is **always recomputed live** -- diff the real,
current schema's own wire list against the real, currently-committed
`artifacts/<provider>/intros.json` (or the equivalent
descriptions/categories file), or run `coverage_check.py` directly and
read its own gap report -- never trusted from the manifest's own last
batch count.

This is not a hypothetical caution. A real, fresh recompute against the
current corpus, done because a batch total had stopped reconciling
cleanly partway through, found the running narrative count was wrong --
not by a rounding error, by enough that trusting it would have reported
work as complete that wasn't (`ubiquex` `STATE.md`, UBI-209's own entry).
The manifest is worth keeping because it tells a resuming session WHAT
WAS TRIED and WHEN, useful context for judging why a hop is stuck -- it
is not worth trusting as WHAT REMAINS.
