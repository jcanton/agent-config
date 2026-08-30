---
name: working-principles
description: Load before writing or reviewing code, and again over the finished diff. Distilled software design practice — complexity, deep modules, DRY, domain modelling, architecture, daily craft — plus a red-flag checklist that is mechanically runnable over a diff.
---

# Working principles

The full text lives in `principles.md` next to this file (a symlink into the
`gt4py_knowledge` repo, so it always reflects the current version). Read it —
do not work from this summary alone.

## When to read what

- **Before writing or reviewing code:** read `principles.md` §1 (prime
  directives), §2 (modules and interfaces), §5 (construction and daily
  practice).
- **Over the finished diff, before declaring work done:** run §6 (red-flag
  checklist). This is the highest-value section — it is the only one that is
  mechanically runnable over a diff.

## Apply critically, not mechanically

- §3 and §4 (DDD tactical/strategic — ENTITY / VALUE OBJECT / AGGREGATE /
  REPOSITORY, bounded contexts) are largely inert for scripts and data
  pipelines. Forcing that vocabulary onto plumbing code adds ceremony without
  insight. The doc's own "an ill-fitting structure is worse than none" is the
  licence to skip it.
- "Define errors out of existence" vs "crash early" is a real tension the doc
  does not scope. Working resolution: crash early within a unit of work whose
  failure is independently recoverable; aggregate at the boundary above it.
- "Don't program by coincidence — prove assumptions with real data and boundary
  conditions" is filed under §5 but behaves like a prime directive. It is the
  rule that catches plausible-but-wrong readings of a data format, which tests
  do not.

## Why it earns the read

On 2026-07-30 the §6 checklist caught three real defects in freshly written
code that had already passed tests and pre-commit: a discarded return value
that left a function dead, an inconsistent dataclass interface (mixing counts
and lists) that also cost real reporting ability, and a duplicated constant.
None were visible from the tests.

## Relation to `design-principles`

`design-principles` (from `msimberg/agent-skills`) is a shorter list of review
taste distilled from real review comments — simplicity, reuse, scope
discipline, comment rules. It overlaps this doc and does not replace it: use
`design-principles` for the reviewer's-eye judgment calls, this skill for the
underlying design theory and the §6 diff checklist. When they conflict, prefer
the more specific and concrete guidance.
