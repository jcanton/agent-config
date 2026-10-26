---
name: verify
description: Exercise a change before committing it - run the narrowest tests that execute the changed code and report what they show. Has the specifics for icon4py.
---

# Verify a change

The evidence is a run that executes the changed code. Typecheck or lint alone
is not verification.

Read the diff (`git diff HEAD`, plus `git diff @{upstream}...HEAD` on a
branch). Skip verification for docs-only or tests-only changes and say so.

Outside icon4py: find the repo's test command (CLAUDE.md, README, pyproject,
noxfile) and run the narrowest tests that execute the changed code. With no
tests, run the changed code directly (call the function, run the script on a
safe input) and show the output. Then report as in step 6 below.

## icon4py

1. Read the diff as above.
2. Pick the narrowest tests that execute each changed path:

   | Change | Run |
   |---|---|
   | stencil (`stencils/*.py`) | its stencil test: `-k <stencil_name>` in `model/<component>/tests/<component>/stencil_tests/` |
   | component or granule logic | its unit tests; if numerics change, the matching datatest (`--datatest-only -k <test>`) |
   | grid, geometry, interpolation (`model/common`) | `model/common` unit tests for that module, then one datatest of a component that uses it |
   | halo exchange, decomposition | the component's `mpi_tests` (command in CLAUDE.md), 2 ranks |
   | driver | `model/driver` tests touching the changed step |

   Command form: `uv run --group test --frozen pytest -n0 <path> -k <expr> --backend=<b>`.
3. Backends on this machine: `embedded` (fast, no compilation) and `gtfn_cpu`
   (exercises code generation; use it when the stencil body or its domain
   changed). Never `gtfn_gpu` or `dace_*` locally.
4. Before running:
   - Datatests with manually placed data: `touch testdata/ser_icondata/<experiment>/.extraction_complete`
     first, or the download step wipes the directory.
   - `gtfn_cpu` with a cold cache, or MPI: set `GT4PY_BUILD_JOBS=2`. Warm the
     cache single-rank before an MPI run.
   - In a worktree on a different Python version, use a separate
     `GT4PY_BUILD_CACHE_DIR`.
5. Run `uv run --group dev --frozen --isolated pre-commit run --files <changed files>`.
6. Report one verdict per changed path: PASS, FAIL (with the failing assertion
   and the measured deviation), or NOT RUN (why, e.g. GPU-only path, missing
   data). For NOT RUN paths, name the CSCS CI subset that would cover them
   (`cscs-ci run default;...`, see CLAUDE.md).
