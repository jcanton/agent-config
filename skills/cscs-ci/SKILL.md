---
name: cscs-ci
description: Load when dealing with CSCS CI: checking status, fetching CI logs, triggering pipelines (cscs-ci run), writing CI config, or encountering GitLab mirror URLs.
---

## What CSCS CI is

CSCS use GitLab CI through a middleware (`cicd-ext-mw.cscs.ch`). GitHub
repositories are registered with CSCS; a webhook pushes a mirror to GitLab and
pipelines run there. GitHub check-run `details_url`s point to the CSCS viewer,
not to GitLab directly. Status is posted back to GitHub as check runs/statuses.

CI status names on GitHub follow the format `cscs/<pipeline_name>` (e.g.
`cscs/default`, `cscs/distributed`). This is how CSCS pipelines appear in
`gh pr checks` output and the GitHub PR UI.

CI may trigger automatically on push or PR events depending on how the
project is configured (CI-enabled branches and trusted users, set on the
CSCS CI setup page). This is project-specific. If it is not explicitly
stated for the current project, ask the user before assuming CI will run
or won't run on its own.

## `cscs-ci run` is NOT a shell command

This is a **PR comment** posted on GitHub to trigger a specific
pipeline. Do not try to run it in a shell.

Syntax:
- `cscs-ci run`: triggers the single (default) pipeline
- `cscs-ci run PIPELINE_NAME`: triggers a named pipeline
- `cscs-ci run PIPELINE_1,PIPELINE_2`: multiple pipelines, comma-separated
- `cscs-ci run PIPELINE;VAR=value;VAR2=value2`: injects variables as
  environment variables into the pipeline

Rules:
- Disallowed characters in pipeline name, variable name, and variable value:
  `,;=` (they serve as separators between components)
- Only the first line of the comment is evaluated; lines 2+ are free context
- Only works if the commenting user is in the pipeline's trusted users list
- The target branch is ignored; you can trigger a pipeline even if the
  target branch is not in the pipeline's CI-enabled branches

## Fetching logs

### The `&type=gitlab` gotcha

GitHub check-run `details_url`s for CSCS CI end in `&type=gitlab`. This
forces an OAuth login and cannot be fetched by text-mode tools. **Always
strip `&type=gitlab` from the URL first.**

### CSCS viewer URLs

Pipeline result page (without `&type=gitlab`):
```
https://cicd-ext-mw.cscs.ch/ci/pipeline/results/{namespace}/{project_id}/{pipeline_id}?iid={pr_id}
```
This static HTML page contains the job list. Per-job result pages:
```
https://cicd-ext-mw.cscs.ch/ci/job/result/{namespace}/{project_id}/{job_id}?iid={pr_id}
```

**The job page embeds the full trace inline as HTML, unauthenticated — try this
before the GitLab API.** It works while the job is still running, so it is also how
to check an in-flight job. Take `{namespace}/{project_id}` straight from the
pipeline `details_url`.

Scrape the `/ci/job/result/.../{job_id}` hrefs from the pipeline page **in document
order** and pair each with its following label: the job ids are not in display
order, so sorting them numerically mislabels every job.

Each log line sits in its own element with a `HH:MM:SS` sibling. To recover plain
text: strip tags, `html.unescape`, then drop lines matching `^\s*\d\d:\d\d:\d\d\s*$`.

### GitLab raw traces

Note the API project path built from the viewer URL ids
(`cscs-ci/ci-testing/webhook-ci/mirrors/{namespace}/{project_id}`) does not resolve
for every registered project — it 404s for C2SM/icon4py. On a 404, fall back to the
job pages above instead of hunting for the right path.


The GitLab API endpoints `/jobs/{id}` and `/jobs/{id}/trace` require
authentication and return 401 even for public projects. Use the raw trace
endpoint instead. It is publicly accessible:
```
https://gitlab.com/{project/path}/-/jobs/{id}/raw
```
Use the project path with **literal slashes**, not percent-encoded. The
`%2F` form is for `/api/v4/projects/{id}` endpoints only; on this web route
it 403s and redirects to `/users/sign_in`, which looks like an auth wall but
is just the wrong URL. Do not conclude auth is required before trying `/raw`
with plain slashes.

Find the project path and the job ids without authentication:
```bash
P=cscs-ci/ci-testing/webhook-ci/mirrors/{repo-id}/{project-id}
PA=$(printf %s "$P" | sed 's|/|%2F|g')
# parent pipeline -> child pipeline id (test jobs live in the child)
curl -sS "https://gitlab.com/api/v4/projects/$PA/pipelines/{parent_id}/bridges" # unauth OK
# job ids and statuses
curl -sS "https://gitlab.com/api/v4/projects/$PA/pipelines/{child_id}/jobs?per_page=100" # unauth OK
curl -sL "https://gitlab.com/$P/-/jobs/{job_id}/raw" -o job.log # literal slashes
```
The pipeline/jobs/bridges API endpoints work unauthenticated; only
`/jobs/{id}` and `/jobs/{id}/trace` need a token.

### Download procedure

CI logs can be large (100k+ lines). Never fetch full logs into context.  Always
download to a temp file and grep:

```bash
LOGFILE=$(mktemp /tmp/cscs-log.XXXXXX.log)
curl -sL "<stripped-url>" -o "$LOGFILE"
wc -l "$LOGFILE"
```

If the fetched page is empty (traces are sometimes JavaScript-loaded), ask
the user to manually export the log from the CSCS viewer and provide the
file path.

## CSCS CI-specific pitfalls

These differ from standard GitLab CI:

- `CI_PIPELINE_SOURCE` is always `trigger`, never `push` or
  `merge_request` or any other value from GitLab's predefined variables.
- `rules: changes` will not work; pipelines are always triggered via the
  middleware, never as branch or merge request pipelines.
- `only`/`except` and `rules` can restrict jobs to specific branches, but
  a pipeline is only triggered if it matches the rules defined on the
  repository's CI setup page (not just the YAML).
- Trigger jobs for child pipelines must use
  `trigger: forward: pipeline_variables: true` to propagate parent
  variables. Without this, child pipelines will not receive variables like
  `CSCS_CI_MW_URL`.
- `ERROR: Job failed: exit status 0` is an **infrastructure** failure, not a
  test failure. The `f7t`/Slurm custom executor lost the compute node; the
  job script never reported a real exit code. Look for
  `Compute node never connected` or a long gap between the last log line and
  the failure. Do not debug the code for these; re-trigger. Several backends
  failing at once with this message points at the cluster, not the branch.
- Trigger-comment variables override `variables:` set in the CI YAML
  (trigger variables have the highest GitLab precedence), so a comment can
  flip a value that `ci/base.yml` hardcodes.

## Key CI variables

| Variable | Value | Notes |
|---|---|---|
| `CSCS_CI_MW_URL` | `https://cicd-ext-mw.cscs.ch/ci` | |
| `CSCS_REGISTRY` | `jfrog.svc.cscs.ch` | Internal container registry |
| `CSCS_REGISTRY_PATH` | `jfrog.svc.cscs.ch/docker-ci-ext/<repository-id>` | Write path in registry; images under `**/public/**` are pullable by anyone on CSCS network |
| `CSCS_CI_DEFAULT_SLURM_ACCOUNT` | Configured on CI setup page | Default Slurm account; overridable per-job via `SLURM_ACCOUNT` |
| `CSCS_CI_ORIG_CLONE_URL` | HTTPS (public) or SSH (private) | Clone URL of the registered project, not the mirror |
| `ARCH` | `x86_64` or `aarch64` | |

## Runner types

Runners are referenced by including the CSCS config:
```yaml
include:
  - remote: 'https://gitlab.com/cscs-ci/recipes/-/raw/master/templates/v2/.ci-ext.yml'
```

Runner naming: `.<type>-<cluster>-<microarchitecture>` (e.g.
`.container-runner-daint-gh200`). The cluster and microarchitecture
determine which hardware the job runs on.

| Type | Description |
|---|---|
| `container-builder` | Builds a Docker image from a Dockerfile, pushes to registry. No Slurm. |
| `container-runner` | Runs a container image via Slurm/FirecREST. For GPU/MPI workloads. |
| `container-runner-lightweight` | No Slurm, faster start. Max 60 min, 4 CPUs, 4GB. Public images only. |
| `uenv-builder` | Builds a uenv (unified environment) from recipe YAML. |
| `uenv-runner` | Runs with a uenv image mounted (not inside a container). Via Slurm. |
| `baremetal-runner` | Almost equivalent to a Slurm sbatch script. No container. |
| `f7t-controller` | FirecREST controller for job submission. |
| `reframe-runner` | Runs ReFrame regression tests. |

## Links

- **Clusters:** `https://docs.cscs.ch/clusters/`: available clusters on
  Alps with hardware descriptions (Daint GH200, Eiger zen2, Clariden GH200,
  Santis GH200, Bristen A100, etc.).
- **CI/CD docs:** `https://docs.cscs.ch/services/cicd/`: full setup guide,
  runner reference with all variables and example jobs, best practices, and
  common pitfalls.
