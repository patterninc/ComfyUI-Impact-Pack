# Engineering Best Practices Audit — comfyui-impact-pack

| | |
|---|---|
| **Repository** | `patterninc/ComfyUI-Impact-Pack` |
| **Audit date** | 2026-09-10 |
| **Auditor** | Claude — gauge-repo skill |
| **Rubric version** | `item-credit-v1` — 2026-09-04 (`references/best-practices.md`) |

## Repo profile

ComfyUI custom-node pack — a Python plugin/library (with a small companion `js/` layer of ComfyUI frontend widgets) that adds Detector, Detailer, Upscaler, Pipe, and related nodes to the ComfyUI host runtime. It is a Pattern-maintained vendored fork of upstream `ltdrdata/ComfyUI-Impact-Pack`, onboarded to the AIPLATFORM Backstage catalog (`backstage.yaml`, system `aiplatform`, owner `ai-infra-comms`). Distribution model is a **published package**: a GitHub Action publishes to the Comfy Registry on pushes that touch `pyproject.toml` (`.github/workflows/publish.yml`). There is **no owned deployed service, no database, and no AWS footprint** — the nodes run in-process inside a user's ComfyUI. There is no owned standalone browser UI (widgets render inside the ComfyUI host). Contribution history in this fork is a single automation identity (`patterninc-gha-runner`), so it is effectively a solo/sync-maintained mirror rather than a multi-contributor team repo. GitHub ownership was verified as `patterninc` via `gh repo view`, so Pattern's inherited org-wide Wiz (secret scanning, SAST, dependency coverage) and Toolsmith-managed MCP controls apply. These profile facts justify the N/A decisions below (no wire API, no deploy environment, no owned UI, no database).

## Scorecard

| Metric | Value |
|--------|-------|
| **Critical gates** | **RED** |
| **Adjusted compliance** | **23.0%** |

At least one applicable critical gate is incomplete, so the safety floor is **RED**: AGENTS.md (2), required CI checks (16), unit tests (23), integration tests (24), and reproducible builds/lockfiles (48) are all Gaps. README setup (6), branch protection (15), secret scanning (19), SAST (20), and scoped secrets (40) are Met.

Adjusted compliance is calculated independently:

`(7 Met + 0.5 × 3 Partial) / (49 total − 12 justified N/A) = 8.5 / 37 = 23.0%`

### Status totals

| Status | Items |
|--------|------:|
| Met | 7 |
| Partial | 3 |
| Gap | 27 |
| N/A | 12 |
| **Total** | **49** |

### Per-category breakdown

| Category | Met | Partial | Gap | N/A |
|----------|----:|--------:|----:|----:|
| Documentation & Context | 1 | 1 | 4 | 3 |
| Guardrails & Enforcement | 3 | 0 | 10 | 0 |
| Testing & Feedback Loops | 0 | 0 | 9 | 4 |
| Environment & Tooling | 3 | 2 | 3 | 5 |
| Agent dispatch | 0 | 0 | 1 | 0 |
| **Total** | **7** | **3** | **27** | **12** |

## Documentation & Context

| # | Practice | Status | Evidence | Recommendation / rationale |
|---|----------|--------|----------|----------------------------|
| 1 | Skills / reusable prompt workflows | **Gap** | No `.claude/`, `.cursor/`, or reusable prompt workflows | Add repo-scoped skills for the recurring maintenance tasks (upstream sync, registry publish, node smoke-check). |
| 2 | AGENTS.md | **Gap** | No `AGENTS.md` or `CLAUDE.md` | Add `AGENTS.md` covering layout (`modules/impact/`, `js/`), how to install/run inside ComfyUI, the upstream-fork sync policy, and publish flow. **Critical gate.** |
| 3 | Architecture decision records (ADRs) | **Gap** | No `docs/adr/` | Low priority for a vendored fork; if Pattern-specific divergences from upstream accrue, record them in `docs/adr/`. |
| 4 | Runbooks | **Gap** | `troubleshooting/TROUBLESHOOTING.md` is user-facing, not operational | Add a short maintainer runbook for the registry-publish and upstream-sync procedures. |
| 5 | API contract docs (OpenAPI / protobuf) | **Not applicable** | Plugin registers ad-hoc routes on the ComfyUI aiohttp server (`impact_server.py`); no published wire contract | No externally consumed wire API — clients are ComfyUI graphs, not codegen consumers. |
| 6 | README with setup & run instructions | **Met** | `README.md` (extensive: install, node catalog, compatibility notes) | — **Critical gate.** |
| 7 | Changelog with migration notes | **Partial** | README `NOTICE` section documents per-version compatibility/migration notes; no dedicated `CHANGELOG.md` | Extract release/migration notes into a structured `CHANGELOG.md` (or Keep-a-Changelog format) so upgrades are machine-scannable. |
| 8 | On-call playbooks | **Not applicable** | Distributed package; no owned production service to page on | Runtime is the end user's ComfyUI; there is no on-call surface for this repo. |
| 9 | CODEOWNERS | **Not applicable** | Single automation maintainer; org ruleset already requires PR review | Vendored single-maintainer fork; CODEOWNERS adds no routing value and org-level `require-pr-review` already gates changes. |

## Guardrails & Enforcement

| # | Practice | Status | Evidence | Recommendation / rationale |
|---|----------|--------|----------|----------------------------|
| 10 | Linters | **Gap** | No `ruff`/`flake8`/`pylint` config | Add `ruff` with a lightweight ruleset; run it in CI. |
| 11 | Formatters | **Gap** | No `black`/`ruff format` config | Adopt `black` or `ruff format` and apply once across `modules/`. |
| 12 | Type checking | **Gap** | No `mypy`/`pyright` config; code is untyped | Introduce `mypy` in non-blocking mode, tightening incrementally on `modules/impact/`. |
| 13 | Pre-commit hooks | **Gap** | No `.pre-commit-config.yaml` | Add `pre-commit` wiring lint/format/secret-scan for local feedback. |
| 14 | Commit message conventions | **Gap** | No commitlint / Conventional Commits config | Low priority given automation-driven commits; adopt if changelog generation is added (item 7). |
| 15 | Branch protection rules | **Met** | Org ruleset `require-pr-review` active (`pull_request`, `non_fast_forward`, `deletion`, `copilot_code_review`) | — **Critical gate.** |
| 16 | Required CI checks before merge | **Gap** | Only workflow is `publish.yml` (push-triggered publish); no PR test/build gate; ruleset has no required status checks | Add a CI workflow (lint + import/smoke check) and mark it a required status check in the ruleset. **Critical gate.** |
| 17 | Dependency allow-lists / deny-lists | **Gap** | No dependency policy config | Low priority; consider a minimal allow policy if third-party surface grows. |
| 18 | License compliance scanning | **Gap** | No license-check job (Wiz does not cover this per scoring rules) | Add a license-compatibility check in CI for the runtime dependencies. |
| 19 | Secret scanning | **Met** | Inherited Pattern Wiz policy (org-wide) | — **Critical gate.** |
| 20 | SAST / static analysis gates | **Met** | Inherited Pattern Wiz policy (org-wide SAST + blocking) | — **Critical gate.** |
| 21 | Max complexity limits | **Gap** | No complexity ceiling enforced | Optional; enable `ruff` complexity rules (`C901`) once linting lands (item 10). |
| 22 | Import boundary enforcement | **Gap** | No import-layering rules | Low priority; the `modules/impact` vs `modules/thirdparty` split is convention-only. |

## Testing & Feedback Loops

| # | Practice | Status | Evidence | Recommendation / rationale |
|---|----------|--------|----------|----------------------------|
| 23 | Unit tests | **Gap** | `test/` holds only ComfyUI workflow JSON (manual fixtures); no `pytest` suite | Add unit tests for pure helpers in `modules/impact/` (e.g. `utils.py`, `segs_nodes.py` mask math). **Critical gate.** |
| 24 | Integration tests | **Gap** | Workflow JSONs are run by hand; no automated harness | Add an automated headless harness that loads the node package and executes a minimal graph. **Critical gate.** |
| 25 | Snapshot / golden-file tests | **Gap** | None | Once a test harness exists, add golden-image/SEGS comparisons for detailer output. |
| 26 | Contract tests (Pact) | **Not applicable** | No independently consumed service API | Nodes are consumed via ComfyUI graphs, not cross-service contracts. |
| 27 | End-to-end tests (Playwright) | **Not applicable** | No owned standalone browser UI; widgets render inside the ComfyUI host | E2E of the host UI is ComfyUI's responsibility, not this plugin's. |
| 28 | Visual regression tests | **Not applicable** | No owned UI surface to screenshot-diff | Node widgets are host-rendered; there is no independent visual surface. |
| 29 | Test coverage thresholds | **Gap** | No tests, no coverage config | Establish a baseline coverage gate after items 23–24. |
| 30 | Mutation testing | **Gap** | None | Low priority; defer until a unit-test suite exists. |
| 31 | Load / performance benchmarks | **Gap** | None | Optional; add micro-benchmarks for hot sampling/upscaler paths if perf regressions become a concern. |
| 32 | Flaky test quarantine | **Gap** | No automated suite to quarantine from | Follows naturally once items 23–24 land; not worth building ahead of a suite. |
| 33 | Structured CI output | **Gap** | Only CI is the publish action; no machine-readable test results | Emit JUnit/structured results once a test CI job exists. |
| 34 | Deterministic test fixtures | **Gap** | Workflow JSON fixtures exist but no seeded/pinned test state | Provide deterministic seeds/fixtures alongside the future automated harness. |
| 35 | Smoke tests for deploys | **Not applicable** | "Deploy" is a Comfy Registry publish; no runtime environment to probe | Publish is validated by `publish-node-action`; there is no deployed instance to smoke-test. |

## Environment & Tooling

| # | Practice | Status | Evidence | Recommendation / rationale |
|---|----------|--------|----------|----------------------------|
| 36 | Devcontainer config | **Gap** | No `.devcontainer/` | Optional; a devcontainer bundling a ComfyUI host would ease contributor onboarding. |
| 37 | One-command setup (`make dev`) | **Partial** | `install.py` bootstraps dependencies; no `Makefile`/`justfile` unified dev target | Add a single `make dev` (or `justfile`) wrapping install + local ComfyUI link. |
| 38 | Seed scripts for local databases | **Not applicable** | No database | The plugin has no persistence layer. |
| 39 | MCP servers for external tools | **Met** | Toolsmith-managed MCP access (inherited) | — |
| 40 | Scoped secrets per environment | **Met** | `publish.yml` uses `secrets.REGISTRY_ACCESS_TOKEN` (GitHub Actions secret); no credentials in code | — Single publish credential, scoped to CI, not committed. **Critical gate.** |
| 41 | Preview environments per PR | **Not applicable** | Published package; nothing to deploy per PR | No deployable service to spin up per pull request. |
| 42 | Hot-reload / watch mode | **Not applicable** | Node reload is provided by the ComfyUI host runtime | Developer reload is a host feature, not owned by the plugin. |
| 43 | Structured logging (JSON) | **Gap** | Plain `print`/console logging | Low priority (host captures stdout); adopt structured logging if operational diagnostics are needed. |
| 44 | Observable traces and metrics | **Not applicable** | In-process plugin; observability is owned by the user's ComfyUI | No owned production runtime to instrument. |
| 45 | Feature flags with local overrides | **Partial** | `modules/impact/config.py` reads `impact-pack.ini` toggles (`mmdet_skip`, `sam_editor_cpu`, `disable_gpu_opencv`, …) as user-local overrides | Config-file toggles cover the local-override need; no runtime flag system — acceptable for the profile. |
| 46 | Database migration tooling | **Not applicable** | No database | No schema to migrate. |
| 47 | Dependency update automation | **Met** | Org-wide Wiz for verified Pattern repos (inherited) | — |
| 48 | Reproducible builds (lockfiles) | **Gap** | `requirements.txt` / `pyproject.toml` deps are largely unpinned (`segment-anything`, `scikit-image`, …); no lockfile (`uv.lock`/`poetry.lock`) | Pin dependencies and commit a lockfile so builds are reproducible. **Critical gate.** |

## Agent dispatch

| # | Practice | Status | Evidence | Recommendation / rationale |
|---|----------|--------|----------|----------------------------|
| 49 | Agent-dispatch manifest (`.agents/pattern-agents.json`) | **Gap** | No `.agents/pattern-agents.json`; only `backstage.yaml` catalog metadata | Add `.agents/pattern-agents.json` with `schema_version`, `github.repo`, `clickup_list_id`, `slack_channel`, `datadog.service`/`env`, and `skills.plugins`. No `aws[]` needed — the repo has no AWS footprint. |

## Prioritized recommendations

1. **[S] Gap — AGENTS.md (item 2, critical gate):** Add `AGENTS.md` covering repo layout, run-in-ComfyUI instructions, the upstream-fork sync policy, and the publish flow.
2. **[S] Gap — reproducible builds (item 48, critical gate):** Pin dependency versions in `pyproject.toml`/`requirements.txt` and commit a lockfile.
3. **[M] Gap — required CI checks (item 16, critical gate):** Add a PR-triggered CI workflow (lint + import/smoke check) and mark it required in the org ruleset.
4. **[M] Gap — unit tests (item 23, critical gate):** Add `pytest` coverage for pure helpers in `modules/impact/`.
5. **[L] Gap — integration tests (item 24, critical gate):** Build a headless harness that loads the node package and runs a minimal graph.
6. **[S] Gap — agent-dispatch manifest (item 49):** Add `.agents/pattern-agents.json` so dispatched agents self-configure.
7. **[S] Gap — linter (item 10):** Adopt `ruff` and run it in CI.
8. **[S] Gap — formatter (item 11):** Adopt `black`/`ruff format` and apply once across `modules/`.
9. **[M] Gap — type checking (item 12):** Introduce `mypy` in non-blocking mode, tightening incrementally.
10. **[S] Gap — pre-commit hooks (item 13):** Wire `pre-commit` for local lint/format/secret feedback.
11. **[S] Gap — license compliance scanning (item 18):** Add a dependency-license check in CI.
12. **[S] Partial — changelog (item 7):** Extract the README `NOTICE` migration notes into a structured `CHANGELOG.md`.
13. **[S] Partial — one-command setup (item 37):** Add a `make dev`/`justfile` target wrapping `install.py` and a local ComfyUI link.
14. **[M] Gap — test coverage threshold (item 29):** Set a baseline coverage gate once items 23–24 land.
15. **[S] Gap — runbook (item 4):** Add a maintainer runbook for registry-publish and upstream-sync.

(Remaining lower-priority Gaps — items 1, 3, 14, 17, 21, 22, 25, 30, 31, 32, 33, 34, 36, 43 — follow in checklist order once the above are addressed.)

## Declined practices

| # | Practice | Rationale |
|---|----------|-----------|
| 5 | API contract docs | No externally consumed wire API; consumers are ComfyUI graphs, not codegen clients. |
| 8 | On-call playbooks | Distributed package with no owned production service to page on. |
| 9 | CODEOWNERS | Single automation maintainer plus org-level `require-pr-review`; path routing adds no value. |
| 26 | Contract tests | No independently consumed service API to verify on both sides. |
| 27 | End-to-end tests | No owned standalone browser UI; host-UI E2E belongs to ComfyUI. |
| 28 | Visual regression tests | No owned visual surface; widgets are host-rendered. |
| 35 | Smoke tests for deploys | "Deploy" is a registry publish, validated by the publish action; no runtime instance to probe. |
| 38 | Seed scripts for local databases | No database. |
| 41 | Preview environments per PR | Published package; nothing deployable per PR. |
| 42 | Hot-reload / watch mode | Node reload is provided by the ComfyUI host runtime. |
| 44 | Observable traces and metrics | In-process plugin; observability is owned by the user's ComfyUI. |
| 46 | Database migration tooling | No database schema to manage. |

## Beyond the checklist

- **Config-driven optional dependencies** — `modules/impact/config.py` + `impact-pack.ini` let heavy/optional detectors (e.g. `mmdet`) be toggled per install, keeping the base install lighter.
- **Automated registry publishing** — `publish.yml` publishes to the Comfy Registry only on `pyproject.toml` changes, decoupling releases from routine commits.
- **Runnable example workflows as documentation** — `test/*.json` doubles as a library of reproducible ComfyUI graphs contributors can load to exercise the nodes.
- **Dedicated troubleshooting doc with screenshots** — `troubleshooting/TROUBLESHOOTING.md` gives end users a first-line diagnostic path.
- **Backstage catalog onboarding** — `backstage.yaml` registers the repo in the AIPLATFORM catalog with owner/system metadata.
