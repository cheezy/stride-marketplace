# Changelog

All notable changes to the Stride marketplace pin set will be documented in this file.

## [1.33.0] - 2026-06-05

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.18.0` to **`1.19.0`** so `/plugin update stride@stride-marketplace` pulls the new release. v1.19.0 (1) adds three per-section verdict objects to the `agents/task-reviewer.md` reviewer schema — `testing_strategy` / `patterns` / `pitfalls`, each `{ "status": "passed" | "failed" | "not_assessed" }`, rendered as section tiles in the Kanban review queue, with a consistency rule tying every `"failed"` verdict to a matching-category `issues[]` entry — and bumps `schema_version` from `"1.1"` to `"1.2"`; (2) rewrites the `skills/stride-completing-tasks/SKILL.md` completion contract to persist the reviewer agent's full structured JSON block verbatim as `reviewer_result` (merged with the legacy summary fields) instead of the thin issues-found envelope that stripped the issues, acceptance verdicts, and code-review checks the review queue renders; and (3) fixes the `after_doing` hook (`hooks/stride-hook.sh` + `hooks/stride-hook.ps1`) so the `/changed_files` diff PUT resolves its URL and bearer token from `$PROJECT_DIR/.stride_auth.md` (production `**API Token:**`, never `**Local API Token:**`) with the `$COMMAND` literal extraction as fallback — fixing the empty `changed_files` that occurred on every completion whose curl used `$STRIDE_API_URL` / `$STRIDE_API_TOKEN` shell variables. Forward-compatible additive change; older orchestrators emitting only the legacy envelope still validate. The stride plugin description in the marketplace entry gains a `v1.19.0+:` paragraph. Marketplace `metadata.version` minor-bumped from `1.32.0` to `1.33.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table to version `1.19.0` and refreshed its one-line summary to surface the section verdicts, the rich-block persistence, and the changed_files hook fix.

### Backward compatibility

Pin-only change for the `stride` plugin; the other three plugin pins (`stride-security-review` `2.3.0`, `stride-ideation` `0.7.0`, `stride-lite` `0.10.0`) are unchanged. The stride v1.19.0 wire changes are additive and forward-compatible (see the stride plugin CHANGELOG `1.19.0`).

## [1.32.0] - 2026-05-27

### Added

- **`.claude-plugin/marketplace.json`** — Added a fourth plugin entry, `stride-lite` (v0.10.0), so `/plugin install stride-lite@stride-marketplace` becomes available. `stride-lite` is a lightweight companion plugin to Stride that produces Stride-shaped goal and task markdown documents on disk from a free-text prompt plus an optional requirements directory — no API calls, no kanban setup, no auth files. Surface: three slash commands (`/stride-lite:create-goal`, `/stride-lite:create-task`, `/stride-lite:init`), one workflow skill (`stride-lite-workflow` — file-based equivalent of the full Stride plugin's `stride-workflow`, walks a goal directory through an eight-step task lifecycle), and three subagents (`stride-lite:create-decomposer` decomposes a prompt into a goal + child tasks YAML with no API calls; `stride-lite:task-explorer` is read-only codebase enrichment that appends `## Exploration Report`; `stride-lite:task-reviewer` reviews changes against acceptance criteria, pitfalls, patterns, and testing strategy with narrowly-scoped read-only git Bash and appends `## Review Report` with embedded `reviewer_result` JSON, schema `1.1`). v0.9.0+ ships a `hooks/` enforcement layer (`hooks/hooks.json` + `hooks/stride-lite-hook.sh` + `hooks/stride-lite-hook.ps1`) — Claude Code PreToolUse/PostToolUse handlers auto-fire the three `.stride_lite.md` sections at the corresponding workflow intercepts (`## before_task` before the `task-explorer` Agent dispatch; `## after_task` before the `task-reviewer` Agent dispatch — both blocking; `## after_goal` after the goal.md write that appends `## Completion Summary` — advisory). Cross-platform from day one with `.sh` and `.ps1` mirrors and auto-delegation on native Windows. v0.10.0+ adds a terminal PENDING → IMPLEMENTED archive move in `stride-lite-workflow` Step 8: after `## after_goal` fires, the goal directory moves from `docs/implementation/PENDING/<slug>/` to `docs/implementation/IMPLEMENTED/<slug>/` via `git mv` (when tracked, preserves history) or plain `mv` otherwise, with collision suffixing (`-2`, `-3`, … up to a 1000-iteration safety cap), an after-goal-failure no-move guard, non-`/PENDING/` path skip with a stderr warning, and clean exit on filesystem-mv failure. Single-task files under `PENDING/tasks/` are never moved. Marketplace `metadata.version` minor-bumped from `1.31.0` to `1.32.0` to match the new plugin surface.
- **`README.md`** — Added the `stride-lite` row to the `Available Plugins` table (version `0.10.0`, one-line summary surfacing the v0.9.0 harness hook enforcement and the v0.10.0 terminal archive move) and a new `## stride-lite` per-plugin section beneath the existing three sections, following the same shape (intro paragraph, install command in a fenced bash block, slash-commands list, workflow-skill blurb, agents list, v0.9.0 cross-platform hook enforcement subsection with the trigger table, v0.10.0 terminal-move subsection, and Repository link). The intro paragraph at the top of the README is updated from "Three plugins" to "Four plugins" and the one-line description gains the "file-only lightweight companion" surface.

### Backward compatibility

The three existing plugin pins (`stride` 1.18.0, `stride-security-review` 2.3.0, `stride-ideation` 0.7.0) are unchanged — byte-identical to v1.31.0 in `.claude-plugin/marketplace.json`. Users who do not install `stride-lite` see no behavior change. `stride-lite` is itself a file-only plugin — no Stride server, no `.stride_auth.md`, no `.stride.md` required. The `.stride_lite.md` config file produced by `/stride-lite:init` is optional; both `/stride-lite:create-goal` and `/stride-lite:create-task` work without ever invoking it.

### Source

W918 — adds `stride-lite` to the stride-marketplace catalog so the plugin is installable via `/plugin install stride-lite@stride-marketplace` instead of requiring a manual clone + symlink. Source of truth for the version (`0.10.0`) and description content is `stride-lite/.claude-plugin/plugin.json` + `stride-lite/README.md` + `stride-lite/CHANGELOG.md` (read but not modified by this task).

## [1.30.4] - 2026-05-25

### Fixed

- **`README.md`** — The `Available Plugins` summary table at the top of the README listed the `stride` plugin at the long-stale version `1.14.1`. This row was the user-visible plugin catalog and had not been bumped during the v1.30.0 / v1.30.1 / v1.30.2 / v1.30.3 marketplace releases — six stride plugin patch / minor releases (v1.15.0 → v1.17.3) had landed in `.claude-plugin/marketplace.json` without the table row catching up. The row now reflects `1.17.3` and its one-line summary surfaces the `## after_goal` hook addition (v1.17.0+) and the four review_queue-scored fields emphasis (v1.17.3+) — the two visible feature deltas since 1.14.1. The `stride-security-review` row (`2.3.0`) and the `stride-ideation` row (`0.7.0`) were already in sync with `marketplace.json`; no edits required there.

### Backward compatibility

Identical to v1.30.3 at the user-visible behavior surface — this release exists solely to correct the stale `stride` row in the README table. The pin set in `.claude-plugin/marketplace.json` is byte-identical to v1.30.3 (stride 1.17.3, stride-security-review 2.3.0, stride-ideation 0.7.0). Marketplace `metadata.version` patch-bumps from `1.30.3` to `1.30.4` to match.

### Source

W874 — caught by the user post-v1.30.3 release; folded into a follow-up release rather than amending the published 1.30.3 tag.

## [1.30.3] - 2026-05-25

### Changed

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.17.2` to `1.17.3` so `/plugin update stride@stride-marketplace` picks up the **content-only review_queue scoring emphasis release** (W850 / W851 / W852 bundled under W873). v1.17.3 strengthens three SKILL.md files — `stride-creating-tasks`, `stride-enriching-tasks`, and `stride-creating-goals` — to name the four fields the review_queue dashboard scores at completion (`acceptance_criteria`, `testing_strategy`, `pitfalls`, `patterns_to_follow`) and the empty-pill consequence of omitting any of them. Each skill gains a top-of-file ⚠️ REVIEW QUEUE SCORING callout co-located with the existing MANDATORY / Iron Law callouts; the same four fields are reinforced inside each skill's existing Red Flags - STOP / Rationalization Table / Phase 4 checklist / Task Nesting Rules structures (no new top-level sections introduced). `stride-enriching-tasks` specifically promotes the four fields to individual mandatory-for-review items in the Phase 4 16-item pre-submission checklist (replacing the prior single-line bundling); `stride-creating-goals` stresses that nested tasks are graded individually — no "it's just a subtask" discount, and the goal-level `description` does not satisfy nested-task fields. The plugin description in this file is updated in lockstep so the marketplace search/listing surface reflects the new emphasis. Marketplace `metadata.version` patch-bumped from `1.30.2` to `1.30.3` to match.

### Backward compatibility

Identical to v1.30.2 at the user-visible behavior surface. The plugin-level update is content-only — no hook script, parser contract, env-var matrix, API field shape, or workflow step changed. Every existing task-creation, enrichment, and goal-creation call continues to validate without modification. No `.stride.md`, `.stride_auth.md`, or `.gitignore` changes are required. Agents that already populate the four scored fields see no behavior change; agents that previously skipped them now get explicit prose about the downstream review_queue scoring consequence.

### Source

Stride plugin release: https://github.com/cheezy/stride/releases/tag/v1.17.3 — content-only emphasis release for G166 (W850 / W851 / W852 SKILL.md edits) and W873 (this release). Patch release because the changes are documentation-only emphasis updates inside three SKILL.md files. The goal of the change set is to raise the floor on the four fields the review_queue dashboard scores at completion, so empty pills become rare rather than common.

## [1.30.2] - 2026-05-25

### Changed

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.17.1` to `1.17.2` so `/plugin update stride@stride-marketplace` picks up the **critical fix** for the hook body-shape regression (D35 + W835). Under 1.17.1 and earlier, `finalize_after_doing` PUT a bare top-level array as the request body — `[{...}, {...}]` instead of `{"changed_files": [...]}`. A bare array lands at `params['_json']` under Plug.Parsers, validates as `{:ok, nil}`, and persists as NULL — silently clearing `changed_files` on every task completion against a 1.16.0+ server. Symptom for end-users: the review queue diff panel showed no per-file unified-patch text for any task completed under 1.17.1, even though the on-disk `.stride-changed-files.json` snapshot was correctly captured. v1.17.2 wraps the body on both the bash and PowerShell hooks (inline `cat` substitution on bash; `@{ changed_files = @(...) } | ConvertTo-Json` on PS), preserves the snapshot write-to-disk for legacy consumers, preserves the fail-soft contract, and adds a gated end-to-end PUT round-trip (Test Group 11) so the wire shape can never silently regress again. Marketplace `metadata.version` patch-bumped from `1.30.1` to `1.30.2` to match.

### Backward compatibility

Identical to v1.30.1 at the user-visible surface. The plugin-level patch is a pure bugfix — the wrapped body has always been the documented contract; older deployments observed accepting a bare array (none on 1.16.0+) continue to accept the wrapped form because both routes land at the same `params['changed_files']` slot when the body is a proper object. The four other `.stride.md` hooks (`before_doing`, `after_doing` outer body, `before_review`, `after_review`, `after_goal`) produce byte-identical output to v1.17.1. Users on 1.17.1 who already completed tasks with NULLed `changed_files` cannot recover the lost diffs — those completions persisted NULL at the server. Going forward, every task completed under 1.17.2+ will populate `changed_files` correctly.

### Source

Stride plugin release: https://github.com/cheezy/stride/releases/tag/v1.17.2 — critical patch release for D35 (wire-shape fix in `hooks/stride-hook.{sh,ps1}` + Group 8 assertions in `hooks/test-stride-hook.{sh,ps1}`) and W835 (gated end-to-end Group 11 in `hooks/test-stride-hook.sh` + README docs). Critical because every Stride task completion under 1.17.1 silently destroyed diff data.

## [1.30.1] - 2026-05-22

### Changed

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.17.0` to `1.17.1` so `/plugin update stride@stride-marketplace` picks up the after_goal routing fix (G117 / W504-W508). The v1.17.0 release announced the `## after_goal` hook section and shipped the docs / parser tests / metadata, but the actual routing in `stride-hook.sh` and `stride-hook.ps1` was never wired — a user adding `## after_goal` to `.stride.md` saw no execution even when the server delivered the hook entry. v1.17.1 closes that gap: response payload inspection, blocking execution of the `## after_goal` section, structured JSON for the agent to forward via `PATCH /api/tasks/:goal_id/after_goal`, missing-section back-compat no-op, and end-to-end test coverage on both Unix and Windows harnesses. Marketplace `metadata.version` patch-bumped from `1.30.0` to `1.30.1` to match.

### Backward compatibility

Identical to v1.30.0. The plugin-level patch is a pure bugfix — the four existing hook routes produce byte-identical output (empirically confirmed by the 118 pre-existing tests passing unchanged after the parse-and-exec refactor). A missing `## after_goal` section continues to be a clean no-op, and the server-side grace-window worker bridges older agent runtimes that don't speak the after_goal protocol.

### Source

Stride plugin release: https://github.com/cheezy/stride/releases/tag/v1.17.1 — patch release for G117 / W504 (bash routing), W505 (PowerShell mirror), W506 (end-to-end tests), W507 (SKILL.md Step 9), W508 (README). v1.17.0 retains the original announcement; v1.17.1 makes that announcement true.

## [1.30.0] - 2026-05-22

### Changed

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.16.0` to `1.17.0` so `/plugin update stride@stride-marketplace` picks up the after_goal hook addition (G113). The plugin now recognizes a fifth `.stride.md` section — `## after_goal` — that fires after the parent goal's final child task completes. Same blocking, single-bash-fence parsing rule as the four existing hooks. The executor forwards GOAL_ID / GOAL_IDENTIFIER / GOAL_TITLE / GOAL_DESCRIPTION env vars verbatim from the server-supplied `hook.env`. The plugin description gains a `v1.17.0+:` clause describing the hook, the GOAL_* env vars, back-compat semantics, and the server-side telemetry / adoption / latency companions. The `v1.16.0+ / v1.15.1+ / v1.15.0+ / v1.14.0+ / v1.14.1+ / v1.13.0+` feature descriptions are preserved. Marketplace `metadata.version` minor-bumped from `1.29.0` to `1.30.0` to match the new feature surface.

### Backward compatibility

A missing `## after_goal` section parses as a clean no-op — older `.stride.md` files that predate the section keep working without modification. The server-side grace-window path remains the back-compat bridge for agent runtimes that don't speak the after_goal protocol: when no agent report arrives within the configured window, the server's `AfterGoal.GraceWorker` synthesizes an attempt with `source: "after_goal_grace_worker"` and promotes the goal. The kanban-side adoption and latency metrics explicitly exclude the grace-worker source tag so back-compat traffic never inflates feature-usage telemetry.

### Source

Stride plugin release: https://github.com/cheezy/stride/releases/tag/v1.17.0 — minor release for G113 / W494-W497 (parser docs + tests + executor contract) and W501 (SKILL.md hooks-table update). Server-side companion: kanban W498 (delivery telemetry), W499 (adoption metric), W500 (goal-to-Done latency p50/p95). The receiving Stride server must include the `PATCH /api/tasks/:id/after_goal` endpoint and the `after_goal_status` / `after_goal_result` / `after_goal_attempts` columns on the `tasks` table for the hook to land — otherwise the agent's POST 404s silently and the grace-window path remains the only promotion path.

## [1.29.0] - 2026-05-21

### Changed

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.15.1` to `1.16.0` so `/plugin update stride@stride-marketplace` picks up the G162 hook-uploaded `changed_files` flow. The plugin's `after_doing` hook now PUTs `.stride-changed-files.json` to `/api/tasks/:id/changed_files` immediately after writing it to disk; URL and Bearer token are extracted from the intercepted agent completion command, no `.stride_auth.md` read, no new env vars; PowerShell mirror in `hooks/stride-hook.ps1`. The companion `stride-completing-tasks` SKILL.md drops `--argjson cf` from the canonical curl and body-shape examples and demotes the inline-cat pattern to a "Legacy inline pattern (≤ v1.15.x deployments)" back-compat fallback. Marketplace `metadata.version` minor-bumped from `1.28.1` to `1.29.0` to match the new agent-side semantic (the agent no longer owns the upload on v1.16.0+ servers). Stride plugin `description` field extended with a `v1.16.0+:` clause describing the hook PUT and the server-side endpoint dependency; v1.15.1+ / v1.15.0+ / v1.14.0+ / v1.14.1+ / v1.13.0+ feature descriptions preserved.

### Backward compatibility

Both modes coexist. On a Stride server v1.16.0+ (W777 endpoint deployed), the hook PUTs the snapshot and the agent's completion body does NOT need `changed_files`. Against older servers, the hook PUT 404s harmlessly (fire-and-forget) and the legacy inline-cat pattern remains the only path that carries the snapshot — SKILL.md's "Legacy inline pattern" subsection documents it explicitly for that case. Agent installs that continue to inline `--argjson cf` work against both server versions (v1.16.0+ servers treat the PUT-uploaded value as authoritative when both are present).

### Source

Stride plugin release: https://github.com/cheezy/stride/releases/tag/v1.16.0 — minor release for G162/W780 (hook PUT) + W781 (SKILL.md). Server-side endpoint shipped as kanban W777 (`PUT /api/tasks/:id/changed_files`); the deployed kanban server must include W777 for the hook PUT to land — otherwise it 404s silently and the back-compat inline-body path still works.

## [1.28.1] - 2026-05-21

### Changed

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.15.0` to `1.15.1` so `/plugin update stride@stride-marketplace` picks up the W767 hotfix in `skills/stride-completing-tasks/SKILL.md`. The fix defaults `$CLAUDE_PROJECT_DIR` to `.` in the three canonical inline-cat occurrences, so agents running under Claude Code's TypeScript SDK (where the variable is unset/empty) successfully read the snapshot they were already trying to send and stop POSTing `changed_files: []`. Marketplace `metadata.version` patch-bumped from `1.28.0` to `1.28.1` to match. Stride plugin `description` field extended with a `v1.15.1+:` clause noting the inline-cat default; v1.15.0+ / v1.14.0+ / v1.14.1+ / v1.13.0+ feature descriptions preserved (the older v1.15.0+ clause's embedded example also updated to the defaulted form for consistency).

### Source

Stride plugin release: https://github.com/cheezy/stride/releases/tag/v1.15.1 — patch release for W767. Hotfix-only release; wire shape and hook behavior unchanged.

## [1.28.0] - 2026-05-20

### Changed

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.14.1` to `1.15.0` so `/plugin update stride@stride-marketplace` picks up the G156/W758 Option D semantic for `capture_changed_files()` (snapshot reflects the agent's working state at completion time, including uncommitted tracked edits and untracked new files) AND the inline-cat fix in the canonical SKILL.md example (the snapshot read now happens INSIDE the curl invocation via `jq --argjson cf`, so the PreToolUse-on-complete hook has finished writing the file before the read). Marketplace `metadata.version` minor-bumped from `1.27.1` to `1.28.0` to match the broadened plugin semantic. Stride plugin `description` field extended with a `v1.15.0+:` clause describing the new working-tree capture and the inline pattern; v1.14.0+ / v1.14.1+ / v1.13.0+ feature descriptions preserved.

### Source

Stride plugin release: https://github.com/cheezy/stride/releases/tag/v1.15.0 — minor release for G156/W758. The plugin's hook capture broadens (committed + staged + modified-uncommitted + untracked-new all in one snapshot), the canonical SKILL.md example switches to the inline cat pattern, and the kanban-repo `docs/diff-contract.md` Encoding section was updated in lockstep with the working-tree-relative semantic. Wire shape is unchanged; legacy payloads continue to validate.

## [1.27.1] - 2026-05-20

### Changed

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.14.0` to `1.14.1` so `/plugin update stride@stride-marketplace` picks up the W748 documentation surfacing edits (canonical API Request Format example body now includes `changed_files`; pre-completion verification checklist now contains the "Did you embed `.stride-changed-files.json` into the payload as `changed_files`?" item). Marketplace `metadata.version` patch-bumped from `1.27.0` to `1.27.1` to match. Stride plugin `description` field extended with a `v1.14.1+:` clause noting the surfacing edits; v1.14.0+ feature description preserved.

### Source

Stride plugin release: https://github.com/cheezy/stride/releases/tag/v1.14.1 — patch release for W748 (SKILL.md surfacing edits). No functional change in the plugin; pure documentation update so installed plugins prompt for `changed_files` during normal payload assembly.
