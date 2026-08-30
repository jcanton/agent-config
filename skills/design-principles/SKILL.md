---
name: design-principles
description: Load when making design decisions, writing implementation code, reviewing code, or evaluating tradeoffs. Shared design principles for all build-* roles.
---

## Simplicity

Does the design do what it needs and nothing more? Can it achieve the same
with simpler code, less code? Does the problem need the most engineered
solution, or is something suboptimal but simple sufficient for the context?
Match engineering effort to stakes; a CI pipeline script doesn't need the
same rigor as a distributed consensus algorithm. Over-engineering is a bug,
not a feature: unnecessary abstraction, premature generalization, and
defensive code that doesn't pull its weight all add maintenance cost without
value. When two approaches solve the problem, the simpler one is correct
unless there's a concrete reason it won't hold up.

> "we don't necessarily need a belt-and-suspenders fix. this is ci, as long
> as it fails loudly it's ok to keep things simple."

## Reuse over duplication

Before creating a new file, layer, abstraction, or wrapper, check whether an
existing one serves the same purpose. A new file whose only role is to
forward to an existing tool adds indirection without value. If two
mechanisms serve the same purpose, one is redundant. This applies at every
level: functions, modules, config files, CI steps, and entire subsystems.

> "the shell script entry point question was rather: why do you have that
> when the python script on its own can act as the entry point?"

## Convention consistency

Check existing solutions for the same kind of problem in the project before
introducing a new pattern. Follow established conventions unless there's a
deliberate reason to diverge, and if you diverge, note it explicitly. A
project with five different ways to do the same thing is harder to maintain
than one with one way and a known exception.

## Scope discipline

Implement only what the spec covers. Trivial, localized cleanups (same
lines, immediate neighborhood) are acceptable. Larger cleanups, refactors,
or improvements become recommendations for follow-up work, not silent scope
expansion. "Don't touch" does not require documentation; just don't touch
it.

## Concern separation

For systems interacting with external services, separate the concerns:
input/source, transformation/processing, output/destination, credentials,
and failure modes. Specs that couple unrelated concerns make every change
more expensive and every failure harder to diagnose.

## Honest tradeoffs

Every design approach has real drawbacks. A proposal with no cons is
incomplete. When you encounter genuine tradeoffs during design, escalate
them to the Coordinator or user rather than silently choosing. Consider:
implementation complexity, maintenance cost, performance implications,
backward compatibility, alignment with existing patterns, and risk level.

## Empirical verification of external behavior

When a design depends on external behavior that is not contractually
guaranteed (tool exit codes, CLI output formats, timing, resource limits),
verify it empirically at least once. Documentation and source-code reading
are not sufficient when the behavior is load-bearing. Before declaring
something unreproducible, make a genuine effort (large sample sizes,
varying parameters).

> "you didn't put a lot of effort into reproducing the issue. you have time
> to run e.g. 100 or 100 repetitions."

## API stability awareness

Examine the project version before making interface changes. Pre-1.0 or
experimental projects: API evolution is acceptable if it improves the
design. Established projects (major version >= 1 or explicit stability
policy): preserve backward compatibility by default. If unclear, ask rather
than guessing.

## Defensive minimalism

Do not add guards, fallbacks, or belt-and-suspenders logic for scenarios
the rest of the design already precludes. If you add a defensive guard, it
must not mask real failures. When the architecture changes, remove
defensive measures that were only needed for the old design.

> "if the job filtering is correct now, there should be not jobs generated
> with no tests, so for actual test runs adding NO_TESTS_COLLECTED_EXIT_CODE
> is actually harmful."

## No unnecessary files or wrappers

Don't create files, modules, or wrappers that only forward to existing
code. Before creating a new file, ask whether an existing entry point
already provides the capability. Prefer using existing entry points
(shebang scripts, `scripts/run`, nox sessions, existing modules) over
creating new indirection.

> "why does scripts/__init__.py need to exist?" ... "good remove it"

## Dependency minimalism

Each external dependency is a maintenance cost, a version conflict risk, and
a security surface. Prefer stdlib/built-in solutions when they're good
enough. A 20-line function is often better than a new dependency that does
the same thing with more generality than you need. The cost of a dependency
is not just its import. It's the transitive closure, the lock file churn,
the breaking updates, and the debug sessions through unfamiliar code.

## Failure mode explicitness

Design with failure in mind. Prefer loud, early, clear failures over silent
degradation. If something can go wrong, the code should make that visible;
not swallow it in a catch-all that returns a default value. Silent failures
defer debugging to production and hide design problems. The question to
ask: if this step fails, will the next step produce a confusing error, a
wrong result, or no output at all?

## Testability as a design signal

If core logic is hard to test without elaborate mocking or environment
setup, that's a design problem, not a testing problem. Design so that the
important behavior can be verified simply. When a test requires ten lines
of mock setup to exercise three lines of logic, the interface is telling
you it has too many dependencies.

## Reversibility

When two approaches are otherwise equal, prefer the one that's easier to
reverse or change later. Avoid decisions that lock in a hard-to-undo
architecture when a reversible alternative exists. This is not about being
indecisive. It's about keeping options open when the cost of keeping them
open is low.

> "revert the change even if it means an expensive communicator creation.
> it's ok for now as long as it's correct."

## Coupling cost awareness

Every coupling between modules has a cost. If module A needs to know about
module B's internals, both are harder to change. Minimize coupling,
especially across subsystem boundaries. The test: if changing B would break
A in a non-obvious way, the coupling is too tight.

## Code comments

Comments state what is non-obvious and permanent about the code. They do
not restate what the code does, document the author's reasoning process,
or confirm defaults.

**What to comment:**
- A non-obvious constraint not visible in the code (e.g., "must be
  called before X because Y reads from the result")
- An intentional divergence from a reference or convention
- A fact that prevents a future developer from introducing a bug (e.g.,
  "This value is intentionally left unset here; do not add a
  default")
- A bare citation to a reference implementation, spec, or ADR

**What not to comment:**
- Restating what the code does. If the function is named
  `exchange_halo`, the comment "Exchange the halo" adds nothing. The
  code already says that. Exception: when the purpose of an action is
  non-obvious (why this specific variable, why at this point in the
  sequence), a comment explaining the purpose can help.
- Documenting the author's process. "I checked the reference and this
  matches" is about the author's workflow, not the code's state. If
  a reference matters, state it as a bare citation ("Implements
  ref.f90: sync_array(SYNC_E, ...)").
- Confirming the default. In a port of an existing codebase, matching
  the original is the expected state. Only divergences from the
  reference are worth noting.
- Commenting about what was. Git history exists for that. "This is implemented
  the same way as the old version" is meaningless. If a comment is needed,
  document what is right now.

**The test:** Would this comment still be useful to a reader who knows
nothing about how the code was written? If the comment only makes sense
in the context of "I was writing this PR and wondered whether this call
was needed," it is process documentation. If it makes sense as a
standalone fact about the code, it is a permanent comment.
