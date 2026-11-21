---
name: coding-lead
description: "Coordinate an approved multi-ticket engineering goal as a Codex Coding Lead. Require the user's actual invocation of Matt Pocock implement-spec, verify worker tdd Skill invocations, and use Herdr Core hc for agent lifecycle, monitoring, integration evidence, and safe cleanup. Also use to resume an existing Coding Lead; not for single isolated edits."
---

# Coding Lead — Herdr adapter for `implement-spec`

**Single source of truth:** Matt Pocock `implement-spec` owns the Spec → ticket graph → integration → review workflow. This skill adds only Herdr Core (`hc`) orchestration and evidence gates; it does **not** restate or simulate `implement-spec`, `tdd`, or `code-review`.

## 1. Invocation gate — before implementation

1. Identify the approved Spec, canonical issue tracker, project rules and exact **Codex Lead Session**. Verify installed Matt Pocock skills and their current invocation metadata.
2. For a **new multi-ticket goal**, require the user to type **`$mattpocock-skills:implement-spec` in that Lead Session**. The upstream skill currently declares `disable-model-invocation: true`; another agent cannot invoke it on the user's behalf. Confirm the **actual user invocation and skill-injection record** in that Session. Merely reading `SKILL.md`, naming it in a prompt or copying its steps is not invocation.
3. On a **resumed goal**, verify the original invocation and an unambiguous handoff identifying the same Goal, Spec, tracker and Session lineage; do not re-trigger the skill or assume authorization transfers between projects.
4. Until confirmed, do read-only Preflight and report `BLOCKED_SKILL_INVOCATION` with the single required user action. Do not start implementation or dispatch workers.
5. After the invocation, **follow the full, currently loaded upstream `implement-spec`**. If the upstream skill changes, use its current instructions rather than a cached description here.

**Gate passes only when:** the correct Lead Session has a verifiable `implement-spec` invocation (or verified same-goal continuation), and the project's approved test seams, source permissions, and Git baseline are known.

## 2. Herdr Core execution adapter

- Run `hc --skill` first, then inspect `hc ls/get/read` and existing sessions. Reuse existing work; launch **all new child agents** (implementers, mergers, reviewers) with verified `hc run --bg --no-notify --parent <LEAD> --cwd <WORKTREE> --name <NAME> ...` to suppress automatic parent-terminal notes. `--no-notify` does **not** suppress explicit `hc send`; enforce the Worker contract below. Default to **at most two concurrent implementers**; verify available models and effort settings rather than assuming names or capabilities.
- Let **`implement-spec`** determine ticket dependencies, worktrees, merger subagents, integration branch, PR milestones and review steps. This adapter only maps them to `hc` sessions and protects worktree ownership. Send [the Worker contract](references/worker-contract.md) as the initial instruction for each implementer; merger/reviewer tasks must likewise finish with a local report and refrain from routine `hc send` to Lead. Verify initial-task delivery before assuming any child is working.
- The Worker must **load the genuine Matt Pocock `tdd` Skill through Codex's native Skill invocation** before writing tests or code. Codex supports an explicit `$mattpocock-skills:tdd` mention in a user message (including an `hc send` message) or model selection of an available model-invocable Skill; **a separate `Skill tool` is not required**. For a fresh Worker, have the first task message explicitly include `$mattpocock-skills:tdd`. Follow the injected Skill at the user-approved public-interface seam. The Worker uses `tdd` per ticket; `implement-spec` remains the user-invoked parent workflow and is not re-triggered in Workers.
- Verify **actual Skill injection** in the Worker Session (e.g. `<skill>` block with the Matt Pocock `tdd` name and path), then require observable RED → GREEN results before accepting a ticket. Reading `SKILL.md`, merely mentioning a Skill in ordinary prose, or finding no standalone `Skill tool` is **not** proof of invocation failure. Only if native discovery/explicit mention does not load the Skill, mark the ticket `BLOCKED_SKILL_INVOCATION` with that evidence; do not silently substitute a homemade TDD recipe.
- Final independent review likewise follows the **actual upstream `code-review` invocation**; collect its verified output and re-review any changed head SHA. Additional security reviewers may supplement but never substitute for the required review.

**Gate passes only when:** each active ticket has a unique owner, session, worktree and acceptance target; each completed ticket has actual `tdd` invocation and test evidence.

## 3. Pull-first result collection, monitoring and integration

- **Lead owns collection:** use `hc wait --children any` to await child-state changes without polling; then `hc get <id> --json` for ownership/status and `hc read <id> --lines N` for the Worker's final report. Read any evidence file it points to. For a busy Worker, `hc read` may show only the current screen: wait for its turn to finish before treating the record as complete. Handle `hc wait` timeouts as check-ins, not failures.
- **Worker reports locally:** each Worker finishes with a compact `READY_FOR_INTEGRATION` or `BLOCKED` summary in **its own terminal**, and writes verbose RED/GREEN logs, diff notes and GUI evidence to task-owned files when needed; send only artifact paths in that local summary. Ordinary completion and routine blockers must **not** call `hc send <LEAD>` or paste long results into the Lead composer. Reserve one short `hc send` for a time-critical safety/authorization escalation that cannot wait for normal collection; never use it for progress spam.
- Lead verifies the Worker's report, commit SHA, changed files, RED/GREEN evidence, permissions and worktree state before acceptance; integrate **using the upstream implement-spec procedure**, then independently rerun tests and E2E. UI/device checks remain `NOT_VERIFIED` unless actually performed. A child `done` state alone is not acceptance; diagnose genuine stalls/errors from evidence using [recovery branches](references/recovery.md).
- Keep a single tracker and inspectable integration branch. Apply project `AGENTS.md` and explicit Git permissions; do not auto-merge, modify production data or silently enable paid/external systems.

**Gate passes only when:** each acceptance claim is supported by the corresponding Skill invocation, immutable commit and independently run tests/review; all remaining blockers are labeled.

## 4. Close and hand off

- Before `hc kill`/`hc rm`, confirm via `hc get/read` that the worker belongs to this Lead (or its owner approves), is no longer working, has delivered results, and will not be called again. Never close another owner's or a currently working terminal.
- Preserve Lead Session, source history, worktrees/commits needed for recovery, PR, and review evidence. Follow upstream `implement-spec` worktree cleanup once safe. Stop task-specific monitoring when engineering work ends.
- Report separately: **ENGINEERING_DONE** (tests/integration/review), **GUI_VERIFIED** (actual desktop/browser checks), **HUMAN_APPROVED** (human authorization), and readiness for merge/deployment.

**Gate passes only when:** the required workflow is verifiably complete or explicitly blocked; no orphan workers remain and every unverified gate has a named owner.

## Boundaries

This skill grants no extra read/write, network, sensitive-data, spending, push or deployment authority. Project-specific rules take precedence. A CLI bypass flag is not OS isolation.
