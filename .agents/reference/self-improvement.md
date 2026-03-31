# Self-Improvement

Every agent session should improve the system, not just complete its task.

**State observation.** `TODO.md`, `todo/PLANS.md`, and GitHub issues/PRs are the state database. Never duplicate into separate files/databases/logs.

**Signals** (check via `gh` CLI, not custom state):

- PR open 6+ hours with no progress
- Same issue/PR in consecutive pulses with no state change
- PR closed (not merged) — worker failed
- Multiple PRs fail CI with same error pattern
- Worker creates a duplicate PR

**Response: file an issue, not a workaround.** Describe pattern, root cause hypothesis, proposed fix. Never patch around a broken process — fix the process.

## Routing

**Framework-level** (files under `~/.aidevops/`, framework scripts, agent prompts, supervisor/pulse, cross-repo orchestration) → `framework-issue-helper.sh log` on `marcusquinn/aidevops`. **Project-specific** (CI, code, deps, domain logic) → current repo. Test: "Would this fix apply to every managed repo?" Never create framework tasks in project repos — invisible to maintainers.

### Filing framework issues (GH#5149)

Use `framework-issue-helper.sh`, not `claim-task-id.sh`:

```bash
# Detect framework vs project (exit 0 = framework, exit 1 = project)
~/.aidevops/agents/scripts/framework-issue-helper.sh detect "description"

# File directly on marcusquinn/aidevops (deduplicates automatically)
~/.aidevops/agents/scripts/framework-issue-helper.sh log \
  --title "Bug: supervisor pipeline fails when stdin is consumed" \
  --body "Observed in ai-lifecycle.sh phase 3: ..." \
  --label "bug"
```

## Scope boundary (t1405, GH#2928)

`PULSE_SCOPE_REPOS` lists repos where worktrees/PRs are allowed. Filing issues is always allowed on any repo. Code changes outside scope → file the issue and stop. Empty/unset (interactive mode) → no restriction.

## What counts

- Filing issues for repeated failure patterns
- Improving agent prompts when workers consistently misunderstand
- Identifying missing automation (manual steps → `gh` commands)
- Flagging stale blocked tasks not marked as such
- Running session miner pulse (`scripts/session-miner-pulse.sh`)
- **Information gaps (t1416):** Comments lacking model tier, branch name, or failure diagnosis → file an issue. Gaps cause cascading waste.

## Issue quality filter (GH#6508)

Before filing enhancements: (1) Observed failure, or preemptive? Preemptive rules for unobserved failures = prompt bloat. (2) Deterministic mechanism where model judgment works better? (3) Comparing to another framework — is the "gap" a deliberate choice? Bar: **observed failure first, minimal guidance**. Bug reports with repro steps are exempt.

## Intelligence over determinism

Pointer: `prompts/build.txt` "Intelligence Over Determinism". Deterministic rules for one-correct-answer items (CLI syntax, paths, security). Everything else is judgment — use cheapest model that handles it.

## Autonomous operation

"continue"/"monitor"/"keep going" → autonomous mode: sleep/wait loops, perpetual todo for compaction survival, interrupt only for blocking errors requiring user input.
