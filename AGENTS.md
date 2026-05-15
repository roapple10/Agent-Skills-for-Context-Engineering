# AGENTS.md

Workspace memory for agents collaborating on this repository. Keep entries durable and broadly applicable; one-off task state belongs in chat or in a run thread, not here.

## Learned User Preferences

- For autonomous research and repo-improvement work in this workspace, prefer proceeding through concrete research loops, subagents, validation, and edits when the scope is clear rather than asking broad process questions.
- Avoid stale regex or keyword-list heuristics in skills and scripts; prefer mechanism-level criteria, rubrics, and evidence-backed validation.
- Never push to GitHub or merge a PR without explicit user approval. Preparing branches, commits, and PRs is permitted only when the user has approved that specific action.
- Tone is technical CTO: direct, no marketing language, no exclamation marks, no emojis, no em dashes. State trade-offs and complexity upfront.
- When the scope spans multiple architectural decisions or irreversible changes, propose a plan first instead of executing.

## Learned Workspace Facts

- This repo is an autonomous research-to-skill organization. External AI research is curated through rubrics and distilled into context-engineering and harness-engineering skill updates.
- `researcher/` is repo-native and file-based so agents can resume, audit, validate, and prepare PR-ready skill changes without a hosted scheduler.
- Per-run state lives in `researcher/runs/<run-id>/run-state.json` with explicit transitions (`initialized -> retrieved -> evaluated -> proposed -> novelty_checked -> validated -> pr_ready -> closed`). Use `research_loop.py` subcommands to advance state, never hand-edit `run-state.json`.
- Repo health (`validate_repo.py`) and per-run readiness (`validate_run.py`) are different questions. CI runs `validate_repo.py --strict`, `run_benchmarks.py`, and `check_activation_cases.py` on every PR via `.github/workflows/validate.yml`.
- The mechanism registry (`researcher/mechanisms/registry.jsonl`) is the encyclopedia backbone. Promotion is gated by `research_loop.py promote-mechanisms` with a recorded reviewer; ledgers live under `researcher/mechanisms/ledgers/`.
- Claim provenance for numeric or volatile claims lives in `researcher/claims/index.jsonl`. Add an entry for any new benchmark or volatility-sensitive claim.
- The corpus index (`researcher/corpus/index.json`) is the machine-readable map of skills, activation scenarios, mechanisms, and claims. Update it when adding or restructuring skills.
- The continuous loop (`researcher/scripts/loop_*.py`) runs from launchd via `researcher/orchestration/launchd/`. It never invokes paid LLMs; HTTP retrieval is stdlib-only with a 1.5 MB cap and a 30-second timeout.
- Runtime state is not committed: `researcher/queue/*.jsonl`, `researcher/queue/.locks/`, `researcher/reports/{logs,snapshots,loop-events.jsonl,loop-failures.jsonl,status.md,parked-review.md}`, and `researcher/runs/*/` are gitignored. The seed run `20260515-035228-executable-autonomous-research-frameworks` is the only committed run; it is closed as `reference-only` and serves as a worked example.
- The current published version is 2.2.0 across `.claude-plugin/marketplace.json`, `.plugin/plugin.json`, and root `SKILL.md`. There are 14 skills.
- Detailed lessons from building the researcher OS are recorded in `researcher/insights/auto-research-experiment.md`; read it before extending the harness.

## Repository Operating Defaults

- Deterministic checks before model judges. Always run `validate_repo.py --strict` before claiming a change is complete.
- Adversarial benchmarks before declaring the harness safe. Add a scenario when a new failure mode is discovered.
- Append-only ledgers for accepted and rejected mechanisms so future agents do not rediscover failed paths.
- Atomic writes (`tempfile` + `os.replace`) and `fcntl` locks for any shared file the loop touches.
- Live execution is the highest-signal validation for orchestration code; smoke-test changes against the actual loop before declaring them safe.
