# CLAUDE.md -- ubx-provider-runbook

## What this is

Executable runbooks for the four recurring operations against a `ubx`
provider: onboarding a new one, regenerating a schema when upstream
drifts, writing artifacts (descriptions, intros, categories), and
regenerating docs against a new pin. Each runbook is a slash command
(`.claude/commands/<name>.md`, invoked as `/<name> <provider>`) that a
Claude Code session runs, not a document a person reads and manually
translates into commands.

This repo carries **operation** -- the steps, in order, with real
commands and real verification. `ubiquex-internals` carries
**explanation** -- why the process is shaped this way, same split
already established there for the rest of the system (UBI-191).

Coordinating repo: `github.com/Ubiquex/ubiquex`. This repo has no code
of its own to build or test -- every command in every runbook runs
against `ubiquex`, `ubx-provider-dynamic`, a provider's own
`ubx-schema-<name>`/`ubx-sdk-<name>` repos, or `ubiquex-docs`.

## The manifest convention

Every runbook records which hop it reached, in a small JSON file
committed alongside the real work each hop produces -- not a separate
side channel, not `/tmp`. A resuming session reads the manifest straight
off the checkout it's already in. See `MANIFEST.md` for the exact shape
and where each runbook's own manifest lives.

**The manifest records history, never the resume point.** What remains
to be done is always recomputed live from real, current state (the
actual schema, the actual committed artifacts, the actual registry) --
never trusted from the manifest's own last-written count. A prior
session's own narrative counts have been wrong before, for real,
independently-confirmed reasons (see `TRAPS.md`).

## The traps

`TRAPS.md` is not optional reading -- every runbook links into it at the
exact step where a trap applies. These are real, repeated, or newly
found failures, not hypothetical ones. A runbook that skips the trap
that caused it will hit it again.

## Git rules (strict)

- PR-only. Never self-merge -- push a branch, open a PR, wait for the
  founder.
- Before pushing more commits to a branch with an open PR, confirm it is
  STILL open (`gh pr list --state open` or `gh pr view <n>`) -- a merged
  PR's branch looks identical to any other from `git status` alone, and
  a push after merge lands nowhere near `main`, silently. `TRAPS.md`
  has a real, freshly-lived example of exactly this.
- NO AI attribution anywhere in commits or PR bodies.
- No em dashes in commit messages or PR bodies.

## Content discipline

- A runbook change is real work, not documentation -- test it against
  the same proving-case discipline the runbooks themselves require of
  the operations they describe (DigitalOcean, for provider onboarding;
  a small existing provider for the other three). A step that has never
  actually been run will hit its own real blocker the first time it is,
  same as UBI-216's own finding about automation that skips the manual
  run first.
- "Committed and pushed" is only true once `git log -1` in this real
  checkout shows the commit AND the content is confirmed via `gh api
  repos/Ubiquex/ubx-provider-runbook/contents/<path>`.
