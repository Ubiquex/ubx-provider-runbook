# TRAPS.md — failures that will happen again if skipped

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
