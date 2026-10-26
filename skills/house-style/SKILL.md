---
name: house-style
description: How the icon4py and gt4py developers write code comments, docstrings, commit messages, PR descriptions and review comments. Use before writing or editing any of these.
---

# Write like the team

Measured on what jcanton, dropd, havogt, egparedes, msimberg and tehrengruber
wrote in icon4py and gt4py between 2023 and mid-2025 (about 170k words, before
agents). Their text is short, plain and specific. Agent-written text in the same
repos was 3-10x longer, with headings, bold and em dashes.

## Everywhere

- Say the thing. No preamble, no closing summary, no recap of the diff.
- Start every comment, sentence and bullet with a capital letter.
- No emoticons or emoji.
- No em or en dashes (0 in the corpus). Split the sentence, or use a comma or
  parentheses.
- No bold, no headings, no semicolons, no `→` in prose.
- Not in their vocabulary: robust, comprehensive, leverage, utilize, seamless,
  crucial, ensure.
- Name code in backticks.

## Code comments

- Short fragments: median 6 words, rarely a full sentence, usually no trailing
  period.
- Say why, a unit, or a constraint the code can't show. Never what the next
  line does.
- About 6 comment lines per 100 lines of code. Most blocks need none.
- `# TODO(handle): ...` with an owner.
- Justify pragmas inline: `# type: ignore[arg-type]  # checked in assert`.
- Cite Fortran (file, routine) only where a reader needs it to check the port.
- Plain `#`; `#:` only where the file already documents attributes that way.

```python
# These magic values seem to mark a torus: they are set in all torus grid files
# TODO(havogt): Should be replaced by offset_provider_type, but global_tmps currently relies on runtime info
# Note: the lift is trivial so inlining it is not an issue with respect to tree size
```

## Docstrings

- Usually one line, imperative, ending with a period:
  `"""Extract the set of symbol names passed into field operators within a program."""`
- Multi-line: summary on the line after `"""`, then a short paragraph.
- `Args:`/`Returns:` only for functions with many arguments, without types
  (the annotations have them).
- About one docstring per 100 lines of code; not every helper needs one.

## Commit messages

- Subject: imperative, 40-60 characters, no trailing period.
- icon4py squash-merges the PR title, so the PR title is the commit subject.
  gt4py uses `type[scope]: subject`, e.g.
  `bug[next]: fix constant folding of expression with boolean literals`.
- Body: usually none. If needed, one or two sentences of why.

## PR descriptions

- Median 24-34 words, one paragraph. A small PR can have one line, or none.
- Open with the change or the bug, not "This PR introduces".
- When behavior changes, before/after in plain words.
- Link related work: `Fixes #1861.`, `Depends on #596`.
- Code or IR in a fenced block when before/after code is the clearest
  explanation.
- Bullets only for separate items. No Summary/Changes/Testing sections, no test
  logs (CI runs anyway).

> Before this PR, the actual compilation error can only be found in the build
> directory. Now, the error is attached to the exception.

> Calling `is_call_to` with the arguments in the wrong order happens easily.
> This PR adds a runtime check to avoid this.

> `pre-commit` was not doing any checks in CI, because it was run without
> `--all-files`.

## Review comments

- One point per comment, median 8-20 words. Questions and direct requests:
  - "Why not a (frozen?) dataclass?"
  - "Missing return type hint"
  - "Let's make this a trait."
  - "Please describe what a `None` result means"
- A suggestion block needs no text when the change speaks for itself.
- Replies to review: "done", "fixed".
- Review body: usually empty or one line ("LGTM", "Please add a small todo
  then we can merge.").

## Before posting

Re-read the draft. Remove dashes, bold, headings, semicolons and emoticons, and
capitalize the first letter of each comment and sentence. Cut every
sentence that restates the code or the diff. Compare the length with the
medians above. Text posted on jcanton's behalf still ends with the footer from
CLAUDE.md.
