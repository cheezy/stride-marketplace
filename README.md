# Stride Marketplace

Four plugins for [Claude Code](https://docs.claude.com/en/docs/claude-code) that integrate with the [Stride](https://www.stridelikeaboss.com) task management platform — task lifecycle, AI-powered security review, structured ideation, and a file-only lightweight companion. Each plugin is independent and can be installed on its own.

## Installation

Add the marketplace to Claude Code once:

```bash
/plugin marketplace add cheezy/stride-marketplace
```

Then install any subset of the plugins below.

## Available Plugins

| Plugin | Version | One-line summary |
|---|---|---|
| [`stride`](#stride) | 1.30.0 | Task lifecycle for Stride kanban — claim, complete, create, decompose, with automatic Claude Code hook execution, per-file diff capture (G148/W719 contract), the `## after_goal` hook section, the five review_queue-scored fields (`acceptance_criteria`, `testing_strategy`, `security_considerations`, `pitfalls`, `patterns_to_follow`) called out as first-class deliverables across the task-authoring skills, (v1.18.0+) project-level checks in `task-reviewer` — reads `CODE-REVIEW.md` at the project root and emits a `project_checks[]` field in the structured reviewer payload, (v1.19.0+) per-section `testing_strategy`/`patterns`/`pitfalls` verdict objects, completion now persisting the reviewer's full structured `reviewer_result` block (not the thin issues-found envelope), and an `after_doing` hook fix that resolves the `/changed_files` upload creds from `.stride_auth.md` so diffs upload even when the completion curl uses `$STRIDE_API_URL`/`$STRIDE_API_TOKEN` shell variables, (v1.20.0+) two new slash commands `/stride:create-tasks` and `/stride:create-goals` with a `--dir`/`--context` markdown-context option that route through `stride-workflow`, and (v1.21.0+) `security_considerations` elevated to a first-class review_queue-scored field with a `task-reviewer` **Security Considerations Alignment** step and a `security_considerations` section verdict (reviewer schema bumped to `1.3`), (v1.22.1+) a fix so completion persists `reviewer_result` **verbatim** (passthrough, never re-enumerate) — restoring the dropped `project_checks` the review queue's Code review panel renders, and (v1.23.0+) the `project_checks[]` per-entry `status` enum gains `not_applicable` with the reviewer now emitting every `CODE-REVIEW.md` bullet (inapplicable ones marked N/A rather than omitted) so the review-queue Code review panel shows the full checklist (reviewer schema bumped to `1.4`), and (v1.24.0+) review reports must be delivered **complete — NO EXCEPTIONS** (G222): every supplied review field is passed to the reviewer and the reviewer's whole structured output (every project check and section verdict) reaches the server intact, via a count-checked whole-object `reviewer_result` passthrough and a mandatory pre-submission self-check hard gate — fixing the recurring `not_assessed` security considerations and truncated `project_checks` (3 of 26), and (v1.25.0+) the changed-files diff upload now survives the after_doing timeout — early capture and PUT before the quality-gate commands run (post-commands refresh kept), a `.stride-diff-upload-state`-backed before_review self-heal retry on missing/stale/non-2xx state, PowerShell parity (plus a repair of the ps1 test suite that failed at baseline on pwsh 7.6), and 120s → 300s Bash hook timeouts with the budget documented, and (v1.28.0+) the claim-time `TASK_BASE_REF` is always refreshed — a persisted-output file fallback plus an unconditional base-ref refresh so an oversized claim response can no longer leave a stale ref that makes `changed_files` span unrelated commits (G224), and (v1.29.0+) the optional free-form `technical_details` task field is documented across the creation, enrichment, decomposition, and workflow/exploration guidance — not a review_queue-scored field (G243), and (v1.30.0+) the creation skills now document the optional `created_by_agent` field on the create request bodies so the `/agents` feed attributes the creating agent instead of a `?` — set to the plugin's own agent name (same as `agent_name` on claim/complete), create-only and forbidden on `PATCH`, propagated from a batch goal to its child tasks (G254) |
| [`stride-security-review`](#stride-security-review) | 2.4.0 | AI-powered security review of code changes — seven framework rule packs (Android, Django, Express, iOS, Phoenix, Rails, React/Next.js), CI/CD pack, defense-in-depth pack, with SARIF / CI integration. v2.4.0: 70-fixture eval coverage (Azure/Drone/Jenkins/Tekton + lockfile-drift/typosquat) and golden-file transform tests |
| [`stride-ideation`](#stride-ideation) | 0.8.0 | Turn a fuzzy idea into a committed requirements doc and Stride tasks via two slash commands. v0.7.0: four-layer resilience model on `/stridify`. v0.8.0: a guided, human-in-control `/ideate` session (per-round completeness recap, "I'm not sure — propose candidates" path, profile recommendation, `--input` brain-dump seed, intra-session draft autosave/resume, reviewer-findings decision) plus a `/stridify` preview-and-approval gate with `--yes` bypass |
| [`stride-lite`](#stride-lite) | 0.10.0 | File-only lightweight companion — writes Stride-shaped goal and task markdown to disk via `/stride-lite:create-goal` / `/stride-lite:create-task` / `/stride-lite:init`. v0.9.0+ harness-enforced `.stride_lite.md` hook auto-fire (PreToolUse/PostToolUse, cross-platform); v0.10.0+ terminal PENDING → IMPLEMENTED archive move in `stride-lite-workflow` |

---

## stride

Task lifecycle skills for Stride kanban: claiming, completing, creating tasks and goals, enriching tasks, workflow orchestration, subagent dispatch, and automatic hook execution via Claude Code hooks.

**v1.30.0+:** the creation skills now document the optional `created_by_agent` field (G254). Agent-created tasks previously landed with `created_by_agent` nil, so the `/agents` activity feed rendered an uninformative `?` avatar on every `created` row. `stride-creating-tasks` adds it to the complete-task example, the Field Quick Reference table, and an explanatory note; `stride-creating-goals` adds it to the batch goal example. Set it to the plugin's own agent name — the exact same value sent as `agent_name` on claim/complete, using the plain agent name and never the `ai_agent:<model>` token form — so one agent stays one roster identity. The field is **create-only** (forbidden on `PATCH`, cannot be backfilled), and the server propagates a batch goal's value to **every nested child task**, so it is set once on the goal. Documentation-only: no wire-shape, hook, `.stride.md`, or `.stride_auth.md` change; `created_by_agent` was already accepted by the create API.

**v1.29.0+:** the `technical_details` task field is now documented across the plugin (G243). It is an **optional, free-form JSON object** a task may carry for any additional technical context that does not fit the structured fields — data shapes, gotchas, key decisions, reference links. Unlike `testing_strategy`, it has **no fixed keys**, and it is **not** one of the five review_queue-scored fields (`acceptance_criteria`, `testing_strategy`, `security_considerations`, `pitfalls`, `patterns_to_follow`), so a blank `{}` is never a scoring gap. The creation contracts (`stride-creating-tasks`, `stride-creating-goals`), the enrichment and decomposition guidance (`task-enricher`, `stride-enriching-tasks`, `task-decomposer`), and the workflow/exploration references agents read while working a task (`stride-workflow` Step 1, `task-explorer`) all describe the field consistently — optional, free-form, never fabricated, with a no-secrets reminder. Documentation-only: no wire-shape, hook, `.stride.md`, or `.stride_auth.md` change.

**v1.28.0+:** the claim-time `TASK_BASE_REF` is always refreshed (G224). The PostToolUse hook records the HEAD commit at claim time in `.stride-env-cache`, and the `after_doing` flow diffs the working tree against it to build `changed_files`. The cache was only written when the claim response JSON parsed out of `tool_response.stdout`; an oversized claim response (~200KB+) is persisted to a file by Claude Code, leaving only a `Full output saved to: …` notice in stdout — so the parse failed, the write was skipped, and a **stale** `TASK_BASE_REF` from a previous claim survived, making `changed_files` span unrelated commits (D60 showed 27 files for a 3-file task). **`hooks/stride-hook.sh`** (W1086) adds a persisted-output file fallback (recover the API JSON from the saved file — validated as a regular file and parsed with `jq` only, never sourced/eval'd; space- and quote-tolerant path) and makes the base-ref refresh **unconditional** on every claim: when no task JSON is obtainable it still rewrites `TASK_BASE_REF` to the current HEAD and clears `.stride-changed-files.json` / `.stride-diff-upload-state`, preserving `TASK_` identity lines (silent no-op in a non-git dir). **`hooks/stride-hook.ps1`** (W1087) ports the same behavior for Windows parity (the ps1 hook previously never wrote `TASK_BASE_REF`) and fixes a pre-existing `Set-StrictMode` hazard where reading `.data` on the stdout-wrapper object threw and aborted caching. Bash Test Group 14 and PowerShell Test Group 10 mirror test-for-test (bash 237, PowerShell 172 green). No wire-shape, hook-timeout, `.stride.md`, or `.stride_auth.md` change. This pin also carries the previously-unreleased **v1.26.0** (D65/D66) and **v1.27.0** (D67) plugin changes.

**v1.25.0+:** the changed-files diff upload survives the `after_doing` timeout (W1093-W1096). The after_doing quality gate (tests with coverage, credo, sobelow, auto-commit) shares one hook timeout with the per-file diff upload, which used to run only **after** every gate command succeeded — a slow gate let the timeout kill the process before the upload, silently losing the task's diffs. Four defense layers: **(1)** `hooks/stride-hook.sh` now captures and PUTs the snapshot **before** the first after_doing section command executes (gated internally on the GLOBAL `$HOOK_NAME` so the after_goal reuse stays inert; degraded captures still write a best-effort `[]` and never block the gate), keeping the post-commands call as a refresh so tree-mutating gate commands still surface in the final snapshot. **(2)** A self-heal layer: `finalize_after_doing` records each PUT outcome in `.stride-diff-upload-state` (task id + HTTP code only, never credentials), and the before_review hook — a fresh PostToolUse timeout budget — verifies that state, re-capturing against `TASK_BASE_REF` and re-PUTting when it is missing, names a different task, or recorded a non-2xx; a healthy 2xx short-circuits before credential resolution. The upload moved into a shared `upload_changed_files_snapshot` helper preserving the D61 base64 envelope; the state file is cleaned at the claim refresh and the after_review cleanup. **(3)** `hooks/stride-hook.ps1` Windows parity — the same early upload, state file, and before_review retry, with the claim refresh and after_review cleanup also clearing the snapshot (the ps1 script has no capture step, so a stale snapshot must never re-upload under a new task id) — plus a repair of the ps1 test suite, which failed at baseline on pwsh 7.6 (reserved `$Input` param name, StrictMode `.Count`-on-null crash, `Start-Process -ArgumentList` argv mangling on Unix .NET that silently lost all section-command output — replaced with `ProcessStartInfo.ArgumentList` and concurrent `ReadToEndAsync` drains — invalid hand-rolled JSON dropping the Bearer token, a listener-startup race, and stale pre-D61 assertions). **(4)** Both Bash hook timeouts in `hooks/hooks.json` raised 120 → 300 seconds (the Skill-matcher gate stays at 10s), with a README subsection documenting the after_doing time budget as a shared ceiling, the kill behavior, and the trim-or-fork recommendation. Test suites: bash 198 assertions green (new Groups 12+13), PowerShell 131 green (new Group 9). Projects should gitignore `.stride-diff-upload-state` alongside `.stride-env-cache` and `.stride-changed-files.json`.

**v1.24.0+:** review reports must be delivered complete — **NO EXCEPTIONS** (goal G222). Every part of a dispatched review report is now mandatory on every task completion: all review information the task supplies (`security_considerations`, `testing_strategy`, `patterns_to_follow`, `pitfalls`, `acceptance_criteria`) must be passed to the reviewer, and the reviewer's entire structured output (every project check and every section verdict) must reach the server intact — not for small or trivial tasks, never with a promise to fix it later. This is the plugin-side source fix for the recurring defect where a task's security considerations came back **`not_assessed`** and the project-checks list was silently **truncated** (3 of 26 reached the server). `skills/stride-workflow/SKILL.md` (W1072) now passes **every** review field the task supplies to the reviewer (it previously listed only four, omitting `security_considerations`, so it came back `not_assessed`); `agents/task-reviewer.md` (W1073 / W1076) reserves `not_assessed` **strictly** for a section the task itself left empty and single-sources its input contract; the `reviewer_result` passthrough (W1074, in `skills/stride-workflow/SKILL.md` + `skills/stride-completing-tasks/SKILL.md`) is now a mechanical **whole-object copy** with a count-checked self-check (submitted `project_checks` count must equal the reviewer's); and `skills/stride-completing-tasks/SKILL.md` (W1075) adds a mandatory **pre-submission self-check hard gate** that refuses to submit a thin or task-inconsistent report. Documentation/agent-prompt change only — no wire-shape, hook, `.stride.md`, or `.stride_auth.md` change; the Kanban server independently hard-rejects an incomplete or task-inconsistent report (goal G221), so the two halves enforce completeness from both ends.

**Install:**

```bash
/plugin install stride@stride-marketplace
```

**Skills (all MANDATORY — each contains required API fields only documented in that skill):**

- `stride-workflow` — RECOMMENDED entry point — single orchestrator for the complete task lifecycle (claim → explore → implement → review → complete)
- `stride-claiming-tasks` — MANDATORY before `GET /api/tasks/next` or `POST /api/tasks/claim`
- `stride-completing-tasks` — MANDATORY before `PATCH /api/tasks/:id/complete`
- `stride-creating-tasks` — MANDATORY before `POST /api/tasks` (work tasks or defects)
- `stride-creating-goals` — MANDATORY before `POST /api/tasks/batch` (goals with nested tasks)
- `stride-enriching-tasks` — MANDATORY when task has empty `key_files` / `testing_strategy` / `verification_steps`
- `stride-subagent-workflow` — MANDATORY after claiming any task, before implementation (Claude Code only)

**Agents:**

- `stride:task-decomposer` — Breaks goals and large tasks into dependency-ordered child tasks
- `stride:task-explorer` — Targeted codebase exploration after claiming a task, guided by `key_files` and `patterns_to_follow`
- `stride:task-reviewer` — Pre-completion code review validating changes against `acceptance_criteria` and `pitfalls`
- `stride:hook-diagnostician` — Analyzes hook failure output (structured JSON or raw text) and returns a prioritized fix plan

**Per-File Diff Capture (v1.14.0+):**

The plugin implements the G148/W719 per-file diff JSON contract. The `after_doing` hook captures `git diff $TASK_BASE_REF..HEAD` per changed file, truncates each diff at exactly 500 lines with the contract marker, emits the binary placeholder for files git marks binary in `--numstat`, and writes the resulting JSON array to `.stride-changed-files.json` for the completion payload to carry as the optional `changed_files` field. Reviewers see per-file unified-patch text inline in the Stride review queue. The field is optional — legacy completions without `changed_files` continue to validate.

**v1.14.1+:** the canonical API Request Format example body and the pre-completion verification checklist in `stride-completing-tasks/SKILL.md` both surface `changed_files` directly, so agents reading the standard example are prompted to embed the snapshot during normal payload assembly (no need to find the dedicated optional section first).

**v1.16.0+:** the `after_doing` hook PUTs `.stride-changed-files.json` to `/api/tasks/:id/changed_files` immediately after writing it to disk — URL and Bearer token are extracted from the intercepted agent completion command, fire-and-forget, no new env vars or `.stride_auth.md` read. PowerShell mirror in `hooks/stride-hook.ps1`. The agent's completion body no longer needs `changed_files` on a Stride server v1.16.0+ deployment (kanban W777 endpoint). The legacy inline-cat pattern is preserved in SKILL.md under a "Legacy inline pattern" subsection for back-compat against ≤ v1.15.x server deployments — both modes coexist.

**v1.22.0+:** the `after_doing` hook PUTs the snapshot to `/api/tasks/:id/changed_files` as a transport-encoded envelope — `{"changed_files":{"encoding":"base64","data":"<single-line-base64>"}}` — instead of the raw `{"changed_files":[...]}` array, so an edge request filter (WAF) in front of the Stride server cannot misread a dense code diff as an attack payload and silently drop the upload (which left `changed_files` empty in the review queue). The server decodes the base64 back to the identical list. When `base64` is unavailable the hook falls back to the raw array shape (the object wrapper is preserved on both paths so the value never lands at `params['_json']` as NULL), and a non-2xx upload response is now surfaced as a stderr warning instead of being discarded (non-fatal to completion; the bearer token is never logged). PowerShell mirror updated to match. Requires the Kanban server to accept the `base64` / `gzip+base64` envelope on `/changed_files` (decoded with bounded streaming inflate + size guards), which ships independently in the kanban repo. Source: D61.

**v1.22.1+:** fixes a data-loss defect where the completion flow dropped `reviewer_result.project_checks`. `skills/stride-workflow/SKILL.md` Step 6 built `reviewer_result` from a hand-maintained enumerated copy-list that omitted `project_checks`, so the reviewer's CODE-REVIEW.md per-bullet audit was silently dropped on completion and the Kanban review queue's **Code review** panel (gated on `reviewer_result["project_checks"]`) rendered nothing. The guidance is now a **verbatim passthrough** — copy the reviewer's entire parsed JSON object into `reviewer_result` and overlay only the legacy summary fields — so any key the schema gains later flows through automatically with no consumer edit. `agents/task-reviewer.md` gains a **consumption invariant** making it the single place the structured key-set is enumerated (consumers passthrough verbatim, never re-enumerate), reinforced by a `stride-completing-tasks` completion-checklist guard. Documentation/skill-instruction change only — no wire-shape change; `project_checks[]` already existed (v1.18.0+) and was already rendered. The kanban-repo regression test ships independently. Source: D63 / W1049 / W1051 (goal G218).

**v1.17.0+:** adds the `## after_goal` hook section — fires after the parent goal's final child task completes, blocking, 60s timeout, same single-bash-fence parsing rule as the four existing hooks. The executor forwards `GOAL_ID` / `GOAL_IDENTIFIER` / `GOAL_TITLE` / `GOAL_DESCRIPTION` env vars from the server-supplied `hook.env` (server is the single source of truth — the executor never invents or looks up these values client-side). A missing `## after_goal` section parses as a clean no-op so older `.stride.md` files keep working; the server-side grace-window path bridges agent runtimes that don't speak the after_goal protocol. SKILL.md gains a five-row hooks reference table (timing/blocking/timeout columns), a Hook Environment Variables matrix, and a Canonical Hook Examples block — the hook is general-purpose (Slack notifications, artifact archival, release pipelines, project-level smoke tests are equally valid uses). Server-side companion (kanban-side): W498 emits `[:kanban, :api, :after_goal_delivered]` telemetry on each real delivery, W499 surfaces an after_goal adoption tile (distinct boards reporting in trailing 7d, grace-worker excluded), W500 surfaces p50/p95 goal-to-Done latency (14d window, server-side timestamps, grace-worker excluded).

**v1.17.3+:** strengthens the three task-authoring SKILL.md files (`stride-creating-tasks`, `stride-enriching-tasks`, `stride-creating-goals`) to name the four review_queue-scored fields explicitly — `acceptance_criteria`, `testing_strategy`, `pitfalls`, `patterns_to_follow` — and the empty-pill consequence of omitting any of them at completion. Each skill gains a top-of-file ⚠️ REVIEW QUEUE SCORING callout (co-located with the existing MANDATORY / Iron Law callouts), plus reinforcement inside the skill's existing Red Flags - STOP, Rationalization Table, Phase 4 checklist, or Task Nesting Rules structures (no new top-level sections introduced). `stride-enriching-tasks` promotes the four fields to individual mandatory-for-review items in the Phase 4 16-item pre-submission checklist (replacing the prior single-line bundling); `stride-creating-goals` stresses that nested tasks are graded individually — no "it's just a subtask" discount, and the goal-level `description` does not satisfy nested-task fields. Content-only release: no hook script, parser, env-var matrix, API field shape, or workflow step changed.

**Commands (v1.20.0+):**

Two slash commands create Stride work from existing project markdown. `/stride:create-tasks [--dir <path>] [task description]` and `/stride:create-goals [--dir <path>] [goal description]` (alias `--context`; also accepts `--dir=<path>`) load the `.md` files in the given directory as a read-only context bundle and route through the `stride-workflow` orchestrator — which dispatches `stride-creating-tasks` / `stride-creating-goals` with the bundle forwarded verbatim. They never call the creation sub-skills directly; the activation marker and sub-skill gate still apply. A `--dir` set but missing errors (non-zero); an empty directory warns and continues. Context augments interactive intent and never excuses a blank required field.

**Explorer/Reviewer Result Enforcement (v1.9.0+):**

Every `/complete` call must include `explorer_result` and `reviewer_result` objects — either a dispatched subagent result or a self-reported skip with a reason from the fixed enum. Summary must be at least 40 non-whitespace characters. Server validates in grace mode (log + succeed) until the strict flag is flipped after all plugins release, then rejects with 422.

**Automatic Hook Execution (v1.5.0+):**

The plugin ships Claude Code hooks that automatically execute `.stride.md` commands without permission prompts. When the plugin is enabled, Stride API calls are detected and the corresponding hook section runs as a shell process on the harness — bypassing Claude's tool permission layer entirely.

| Claude Code Event | API Pattern | Stride Hook |
|---|---|---|
| PostToolUse (Bash) | `/api/tasks/claim` | `before_doing` |
| PreToolUse (Bash) | `/api/tasks/:id/complete` | `after_doing` (blocks on failure) |
| PostToolUse (Bash) | `/api/tasks/:id/complete` | `before_review` |
| PostToolUse (Bash) | `/api/tasks/:id/mark_reviewed` | `after_review` |

Task environment variables (`$TASK_IDENTIFIER`, `$TASK_TITLE`, etc.) are cached from the claim response and available to all hooks. Structured JSON diagnostics are emitted on both success and failure for integration with the hook-diagnostician agent.

**Repository:** <https://github.com/cheezy/stride>

---

## stride-security-review

AI-powered security review of code changes via the `/stride-security-review:security-review` slash command. Semantic analysis, not pattern matching — a `grep` hit on `eval(` is not a finding; `eval(user_input)` at a trust boundary is.

**Install:**

```bash
/plugin install stride-security-review@stride-marketplace
```

**Two scan modes:**

- **Diff (default)** — reviews working-tree changes against `HEAD`. The PR-gating shape.
- **`--full`** — reviews every tracked text file under a 256 KiB cap, batched in groups of 10 with cross-batch dedup and skipped-files reporting. The codebase-posture-check shape.

**What it catches:**

- **10 universal vulnerability classes** — injection (SQL, command, LDAP, NoSQL, XXE, template, header), authentication, authorization, data exposure, cryptography, input validation, race conditions, XSS / code execution, insecure configuration, supply chain.
- **5 MAESTRO-derived agentic-AI classes** (when the file imports an LLM/agent/MCP SDK) — prompt injection, tool abuse, agent trust boundary, model output execution, vector-store poisoning.
- **Seven framework rule packs** (Android/Kotlin, Django/Python, Express/Node.js, iOS/Swift, Phoenix/Elixir, Rails/Ruby, React/Next.js — covering the full mobile + web + Node ecosystem). Highlights:
  - **Android/Kotlin** — exported components without signature permission, WebView `setJavaScriptEnabled` + `addJavascriptInterface` RCE, cleartext traffic, SharedPreferences storing credentials, SQLite injection, trust-all `HostnameVerifier` / `X509TrustManager`. Activates on `.kt` / `.java` / `.gradle` imports OR `AndroidManifest.xml` content.
  - **Django/Python** — `mark_safe`, `extra` / `raw()` SQL interpolation, CSRF disabled, production settings hardening (DEBUG / ALLOWED_HOSTS / SECURE_*), mass-assignment, open-redirect, deserialization, SSRF with cloud-metadata focus, DRF `ModelSerializer fields='__all__'`.
  - **Express/Node.js** — reflected XSS via `res.send(req.query.x)`, `eval` / `Function()` / `vm.runInNewContext` RCE, `child_process.exec` shell injection, Mongoose `Model.find(req.body)` NoSQL operator injection, prototype pollution via `lodash.merge` / `_.set`.
  - **iOS/Swift** — sensitive data in `UserDefaults` instead of Keychain, WKWebView file access + JS bridge, ATS `NSAllowsArbitraryLoads`, URLSession trust-all delegate, deep-link missing re-auth. Activates on Swift / Obj-C / Obj-C++ imports OR `Info.plist` content.
  - **Phoenix/Elixir** — `Phoenix.HTML.raw/1`, missing `force_ssl`, CSRF, `Ecto.Query.fragment` interpolation, LiveView scope, changeset mass-assignment, `Phoenix.Token` replay, `System.cmd` shell, LiveView `allow_upload` guards, open-redirect.
  - **Rails/Ruby** — `html_safe` / `raw`, `find_by_sql` / `connection.execute` interpolation, `protect_from_forgery` disabled, `params.permit!`, `eval` / `send`, open-redirect, `Marshal.load` / `YAML.load` deserialization, missing `authenticate_user!`, unfiltered `render json: @user` leaking `password_digest` / `*_token` / `*_secret`.
  - **React/Next.js** — `dangerouslySetInnerHTML` XSS, Next.js API route missing auth, redirect open-redirect, `<a href={user_url}>` with `javascript:` scheme bypass, `getServerSideProps` / Route Handler secret-leak via props.
- **CI/CD pipeline pack** — five rules across eight platforms (GitHub Actions, GitLab CI, CircleCI, Bitbucket Pipelines, Jenkins, Azure Pipelines, Drone, Tekton).
- **Web defense-in-depth pack** — framework-agnostic checks for missing CSP, HSTS, X-Frame-Options, and `Set-Cookie` without `Secure` / `HttpOnly` / `SameSite`.

Every finding carries CWE-ID and OWASP Top 10 references.

**Opt-in flags (composable):**

| Flag | Effect |
|---|---|
| `--full` | Full-codebase scan instead of diff. |
| `--json` | Raw JSON output (mutually exclusive with `--sarif`). |
| `--sarif` | SARIF v2.1.0 output for GitHub Code Scanning. |
| `--fail-on <severity>` | Exit non-zero on findings at/above `critical \| high \| medium \| low`. CI gate. |
| `--base <ref>` | Diff against `<ref>...HEAD` (three-dot range) instead of working tree against HEAD. The PR-against-base shape. |
| `--maestro` | Adds MAESTRO 7-layer classification to each finding plus a `By MAESTRO layer` summary section. |
| `--rci [N]` | Run N (1-3) recursive critique-and-refine passes after the initial review. |
| `--baseline [PATH]` | Suppress acknowledged findings from a baseline file. |
| `--update-baseline` | Write a new baseline from the current run. |
| `--patches` | Emit surgical-fix unified-diff `patch` field per finding when one exists. |

**Ships:**

- A reference GitHub Actions workflow (`.github/workflows/security-review.yml`) that gates PRs via a belt-and-suspenders `jq + claude_exit` check.
- A TAP 13 eval runner (`scripts/run_eval.sh`) over 64 fixtures with both positive-control (assert finding fires) and negative-control (assert finding does NOT fire) coverage. Includes its own CI workflow that re-runs the eval on every change to the agent prompt.

**Loosely based on** [`anthropics/claude-code-security-review`](https://github.com/anthropics/claude-code-security-review).

**Repository:** <https://github.com/cheezy/stride-security-review>

---

## stride-ideation

Turn a fuzzy idea into shipped Stride tasks via two slash commands. Replaces `superpowers:brainstorming` for Stride projects; the two coexist for non-Stride projects where brainstorming remains the right fit.

**Install:**

```bash
/plugin install stride-ideation@stride-marketplace
```

**Two slash commands:**

### `/stride-ideation:ideate`

Drives an interactive round-based question loop and writes a committed requirements markdown doc — the terminal state, never auto-funnels into the next step. Up to 4 batched `AskUserQuestion` questions per round, mandatory round-3 framing checkpoint, mandatory round-4 premortem. Hard-gated on seven required sections (Goal, Problem, Outcome, Assumptions, Constraints, Non-goals, Success Metrics) with shape requirements:

- **Assumptions** must be ranked highest-to-lowest risk, riskiest marked `(R)`.
- **Success Metrics** must include both leading and lagging indicators.

An advisory `requirements-reviewer` subagent runs after the doc is assembled and surfaces gaps before commit.

**Profiles (v0.4.0+):** select the round structure and reviewer rubric with `--profile`.

| Profile | Round structure | Adds |
|---|---|---|
| `lean` (default, byte-for-byte v0.3.0) | 4 rounds | Goal / Problem / Outcome / Premortem |
| `product` | Same + JTBD four-forces in Round 1 | Optional "Concrete Example" section |
| `discovery` | Same + Why-now and Alternative-options in Round 2 | — |
| `lean-startup` (v0.5.0) | Same + mandatory Round-5 MVP-design batch | Optional "MVP / Validation experiment" section + two reviewer checks (MVP section presence, falsifiable success/failure criteria) |

The `lean-startup` profile is anchored on the `(R)`-marked Assumption — source techniques: Eric Ries (Lean Startup), Steve Blank (Customer Development), Giff Constable (Riskiest Assumption Test).

**Session experience (v0.8.0+):** the round loop is now guided, recoverable, and human-in-control. Additive — a flag-free or `--profile=lean` run produces the same committed doc.

- **Per-round recap** — before every round, a display-only status (`solid` / `thin` / `empty`) for the seven gated sections; never changes the gate, round order, or `<=4`-question budget.
- **"I'm not sure — propose candidates"** — every gated-section and forcing question carries this option; picking it makes the skill propose 2–4 topic-tailored candidates with rationales. A candidate never satisfies the gate until you confirm it.
- **Profile recommendation** — when `--profile` is omitted, a recommended-first question (lean default) runs before the rounds; explicit `--profile` skips it.
- **`--input <path>` brain-dump seed** — reads a freeform notes file read-only and pre-fills draft sections, then focuses the rounds on gaps. Distinct from and composable with `--continue`; the file is never modified, moved, or committed.
- **Draft autosave & resume** — the in-progress draft is autosaved after every round to a gitignored `.stride/` scratch file; resume-or-fresh is offered for the same slug on start; the scratch is deleted after a successful commit. Never holds the Stride API token.
- **Reviewer decision** — advisory `requirements-reviewer` findings are surfaced as a multi-select (severity-tagged) with an explicit "Address none — write as-is" choice feeding the single refinement round. At most one refinement round; the reviewer never blocks the write.

### `/stride-ideation:stridify`

End-to-end pipeline from requirements doc to created Stride goals. Validates the seven required sections, preflights auth from `.stride_auth.md`, dispatches the `requirements-decomposer` subagent (canonical batch shape embedded; two-reason multi-goal split rule of code-coupling + ~10-task soft cap), stamps `source_spec` and `source_spec_sha256`, writes and commits a sibling timestamped batch JSON for audit, then strips local-audit fields and POSTs to `/api/tasks/batch` with token hygiene (no `curl -v`, no token in logs). Renders the created G/W identifiers.

**Resilience model (v0.7.0+):** four layers that protect against transient Anthropic API capacity spikes and oversized many-surface decomposition runs. Activates only on the relevant trigger — v0.6.0 behavior is preserved byte-for-byte on the happy path.

| Layer | Trigger | What happens |
|---|---|---|
| Preflight advisory | Doc enumerates >3 surfaces under `## Decomposition seams` AND `--goal` is unset | One-line stderr suggestion to use `--goal`; never blocks |
| `--goal <name|index>` | User invokes per-surface dispatch | Prompt scoped to one seam (integer-index OR slug-match; both `--goal value` and `--goal=value` forms); per-goal batch JSON sits side-by-side via `sti_unique_path` |
| Subagent dispatch retry | HTTP 529 / transient network / `overloaded` classification | 3 attempts, ~30s / ~90s backoff; terminal classifications (bad subagent, contract violation, hard 4xx) fail fast on attempt 1 |
| Retry-exhaustion fallback | 3 consecutive transient failures | Writes `<source-stem>-decomposer-prompt.md` sibling with the assembled prompt, verbatim last error, and recovery README; **Stride API POST is NOT attempted** |

The Stride API POST itself is still not retried — partial-batch idempotency is not guaranteed; the user remains the retry mechanism on a 4xx/5xx.

**Preview-and-approval gate (v0.8.0+):** before the POST, `/stridify` renders the decomposed goal/task tree (each goal title, its task count and titles, and the cross-goal claim order from `decomposition_notes`) and requires explicit human approval. The batch JSON is written and committed *before* the gate, so a decline stops cleanly (exit 0) with the audited artifact intact and no POST. Pass `--yes` / `--auto-approve` (explicit only, never inferred) to bypass the gate and preserve the historical fire-and-forget behavior byte-for-byte for scripted callers. The preview reads only the on-disk JSON and never prints the API token.

**Repository:** <https://github.com/cheezy/stride-ideation>

---

## stride-lite

A lightweight companion plugin to Stride — produces Stride-shaped **goal and task markdown documents on disk** from a free-text prompt plus an optional requirements directory. No API calls, no kanban setup, no auth files. Just markdown.

Useful when you want the Stride field discipline (acceptance criteria, key files, pitfalls, testing strategy, dependencies) without committing to the full Stride server workflow. The documents it produces can be reviewed, edited, or later submitted to a real Stride deployment by hand or by tooling — this plugin never does so itself.

**Install:**

```bash
/plugin install stride-lite@stride-marketplace
```

**Three slash commands:**

- `/stride-lite:create-goal <prompt>` — decomposes a free-text prompt into a goal plus 1–8 child tasks (capped at 8) and writes them to `docs/implementation/PENDING/<slug>/` (`goal.md` plus `taskN.md` per child).
- `/stride-lite:create-task <prompt>` — renders a single task markdown file at `<output-dir>/tasks/<slug>.md`. No goal wrapper, no children.
- `/stride-lite:init` — scaffolds a project-local `.stride_lite.md` with an email field plus three hook sections (`## before_task`, `## after_task`, `## after_goal`). Shape matches the full Stride plugin's `.stride.md` so snippets transfer across plugins.

**Workflow skill:**

- `stride-lite-workflow` — file-based equivalent of the full Stride plugin's `stride-workflow`. Walks a goal directory through an eight-step task lifecycle (select → before_task → explore → implement → after_task → review → review-loop decision → completion + archive), dispatching the two read-only subagents at the right moments. Never POSTs to any API.

**Agents:**

- `stride-lite:create-decomposer` — Decomposes a free-text prompt into a goal + child tasks YAML. No API calls.
- `stride-lite:task-explorer` — Read-only codebase enrichment (Read / Grep / Glob; no Bash, no WebFetch). Appends `## Exploration Report` to the task file.
- `stride-lite:task-reviewer` — Reviews changes against acceptance criteria, pitfalls, patterns, and testing strategy with narrowly-scoped read-only git Bash. Appends `## Review Report` with embedded `reviewer_result` JSON (schema `1.1`).

**Cross-platform hook enforcement (v0.9.0+):**

`hooks/hooks.json` registers Claude Code PreToolUse/PostToolUse handlers that auto-fire the three `.stride_lite.md` sections at the corresponding workflow intercepts:

| Trigger | Phase | Section | Blocking? |
|---|---|---|---|
| `task-explorer` Agent dispatch | PreToolUse | `## before_task` | yes (exit 2 stops the dispatch) |
| `task-reviewer` Agent dispatch | PreToolUse | `## after_task` | yes (exit 2 stops the dispatch) |
| Final-task `goal.md` write with `## Completion Summary` | PostToolUse | `## after_goal` | advisory (PostToolUse cannot roll back the write) |

Cross-platform from day one: `hooks/stride-lite-hook.sh` (POSIX bash, pure-bash JSON parsing, no jq dependency) plus `hooks/stride-lite-hook.ps1` (PowerShell 5.1+, `ConvertFrom-Json` / `ConvertTo-Json`, no module installs). The `.sh` script auto-delegates to `.ps1` on native Windows (OSTYPE unset + COMSPEC set).

**Terminal PENDING → IMPLEMENTED archive move (v0.10.0+):**

After `## after_goal` fires on the final task of a goal, `stride-lite-workflow` Step 8 moves the goal directory from `docs/implementation/PENDING/<slug>/` to `docs/implementation/IMPLEMENTED/<slug>/`. Prefers `git mv` when the directory is git-tracked (preserves rename history); falls back to plain `mv` otherwise. Collision suffixing on the target with `-2` / `-3` / … up to a 1000-iteration safety cap — the archive never overwrites prior entries. After-goal-failure guard (a structured `"status": "failed"` JSON from the harness skips the move so the user can inspect and re-trigger). Non-`/PENDING/` paths (custom `--output-dir`) are left where they are with a stderr warning. Filesystem-mv failures log and exit cleanly — the workflow does not fail because of an archive-move failure. Single-task files under `PENDING/tasks/` are NEVER moved; only goal directories.

**Repository:** <https://github.com/cheezy/stride-lite>

---

## Marketplace Structure

```
stride-marketplace/
├── .claude-plugin/
│   └── marketplace.json       # Plugin catalog (the source of truth for plugin versions)
├── README.md                  # This file
└── LICENSE                    # MIT
```

## Support

- **Marketplace issues:** <https://github.com/cheezy/stride-marketplace/issues>
- **Per-plugin repos** are linked in each plugin's section above; report plugin-specific issues there.

## License

MIT License — see [LICENSE](LICENSE).
