# Changelog

All notable changes to this project are documented here. Versions follow semantic versioning where practical, with skill content treated as data.

## [2.2.0] - 2026-05-15

### Added

#### Researcher operating system

- **Mechanism registry** (`researcher/mechanisms/registry.jsonl`) seeded with five accepted mechanisms (`locked-editable-surfaces`, `durable-research-thread`, `deterministic-first-validation`, `structured-novelty-gate`, `pairwise-skill-revision`).
- **Mechanism ledgers** (`researcher/mechanisms/ledgers/accepted.jsonl`, `rejected.jsonl`) for append-only promotion events.
- **Claim provenance** (`researcher/claims/index.jsonl`) for six volatile or benchmark-backed claims across `evaluation`, `multi-agent-patterns`, `context-optimization`, `memory-systems`, `advanced-evaluation`, and `harness-engineering`.
- **Corpus index** (`researcher/corpus/index.json`) mapping skills to activation scenarios, mechanism IDs, and claim IDs.
- **Activation regression fixtures** (`researcher/fixtures/activation-cases.jsonl`) covering high-risk skill-boundary pairs.
- **Adversarial benchmark harness** (`researcher/benchmarks/scenarios/adversarial.jsonl` + goldens) with seven scenarios that try to game the loop.
- **Benchmark history** (`researcher/reports/benchmark-history.jsonl`) for longitudinal trend tracking.
- **Pairwise revision rubric and script** (`researcher/rubrics/pairwise-skill-revision.md`, `researcher/scripts/compare_skill_revisions.py`).
- **Run state machine** in `run-state.json` with explicit transitions: `initialized -> retrieved -> evaluated -> proposed -> novelty_checked -> validated -> pr_ready -> closed`.

#### Continuous loop

- **Queue infrastructure** (`researcher/queue/`): inbox, parked, done, quarantine.
- **Orchestration config** (`researcher/orchestration/config.json`) with daily/active/parked/failure budgets.
- **Discovery feeder** (`researcher/scripts/loop_discover.py`) reading from `researcher/discovery/manual-seed.jsonl`.
- **Loop step orchestrator** (`researcher/scripts/loop_step.py`) that reaps closed runs, pulls from inbox, retrieves via stdlib `urllib`, and parks at human-review gates.
- **Daily ops** (`researcher/scripts/loop_daily.py`) running validators, benchmarks, activation cases, and writing dated snapshots.
- **Status dashboard** (`researcher/scripts/loop_status.py`) plus parked-review surface.
- **launchd service definitions** (`researcher/orchestration/launchd/`) with install/uninstall scripts and per-script wrappers.
- **Continuous operation runbook** (`researcher/runbooks/continuous-operation.md`).

#### Scripts

- `researcher/scripts/validate_run.py`: per-run publish readiness, skips closed runs.
- `researcher/scripts/research_loop.py` subcommands: `retrieve`, `evaluate`, `propose`, `novelty`, `validate-run`, `pr-ready`, `close`, `promote-mechanisms`.
- `researcher/scripts/check_activation_cases.py`: deterministic activation regression checker.
- `researcher/scripts/run_benchmarks.py`: runs deterministic gates, scenarios, optional history recording.
- `researcher/scripts/loop_common.py`: shared atomic-write helpers and `fcntl` locks.

#### CI

- `.github/workflows/validate.yml` runs `validate_repo.py --strict`, `run_benchmarks.py`, `check_activation_cases.py`, and Python compile checks on every push and PR.

### Changed

- Skill activation surface refactored from keyword triggers to task-boundary descriptions in frontmatter and README. Affected: all 14 published skills plus the example skills in `examples/`.
- `validate_repo.py` hardened: duplicate JSON keys, exact doc sync, rubric IDs, run artifacts, registry schema, evidence paths, partial-retrieval approvals, root-level raw provenance, claims schema, corpus index consistency, activation cases, benchmark scenarios.
- `novelty_check.py` now compares mechanism registry overlap as the primary signal, with corpus overlap secondary.
- Mechanism registry evidence may now reference claim IDs from `researcher/claims/index.jsonl` in addition to URLs and repo paths.

### Hardened

- All queue mutations use atomic temp-file `os.replace` and `fcntl` exclusive locks scoped per queue family.
- `read_jsonl` is tolerant: malformed lines quarantine to `researcher/reports/jsonl-quarantine/` rather than crashing the loop.
- `fetch_url` allows only `http(s)://` and re-checks scheme after redirect.
- Closed runs are automatically reaped from `parked.jsonl` and recorded in `done.jsonl`.
- Inbox lock now held through `init_run` so concurrent loop_step invocations cannot exceed budgets.
- URL deduplication normalizes case before hashing.

### Repository policy

- Active research runs under `researcher/runs/*/` are runtime state and not committed. The seed run `20260515-035228-executable-autonomous-research-frameworks` is kept as a worked-example fixture.
- Runtime queue and report files (`researcher/queue/*.jsonl`, `researcher/reports/{logs,snapshots,loop-events.jsonl,loop-failures.jsonl,status.md,parked-review.md}`, `researcher/queue/.locks/`) are gitignored.

### Out of scope for 2.2.0

- LLM-judge adapters for advancing `retrieved -> evaluated` automatically.
- Automated source discovery beyond the manual seed file (Parallel deep research and web search adapters are placeholders behind config toggles).
- Log rotation; benchmark history pruning.

## [2.1.0] - 2026-05-14

### Added

- `harness-engineering` skill: locked/editable surface model, durable threads, novelty gates, rollback, human approval boundaries.
- `researcher/` directory v1: source registry, content/skill/harness rubrics, source-evaluation JSON template, skill-proposal template, autonomous research loop runbook, PR readiness runbook.

## [2.0.0] - earlier

Baseline corpus of 13 skills distributed as a single Claude Code plugin.
