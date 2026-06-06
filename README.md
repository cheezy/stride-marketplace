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
| [`stride`](#stride) | 1.21.0 | Task lifecycle for Stride kanban — claim, complete, create, decompose, with automatic Claude Code hook execution, per-file diff capture (G148/W719 contract), the `## after_goal` hook section, the five review_queue-scored fields (`acceptance_criteria`, `testing_strategy`, `security_considerations`, `pitfalls`, `patterns_to_follow`) called out as first-class deliverables across the task-authoring skills, (v1.18.0+) project-level checks in `task-reviewer` — reads `CODE-REVIEW.md` at the project root and emits a `project_checks[]` field in the structured reviewer payload, (v1.19.0+) per-section `testing_strategy`/`patterns`/`pitfalls` verdict objects, completion now persisting the reviewer's full structured `reviewer_result` block (not the thin issues-found envelope), and an `after_doing` hook fix that resolves the `/changed_files` upload creds from `.stride_auth.md` so diffs upload even when the completion curl uses `$STRIDE_API_URL`/`$STRIDE_API_TOKEN` shell variables, (v1.20.0+) two new slash commands `/stride:create-tasks` and `/stride:create-goals` with a `--dir`/`--context` markdown-context option that route through `stride-workflow`, and (v1.21.0+) `security_considerations` elevated to a first-class review_queue-scored field with a `task-reviewer` **Security Considerations Alignment** step and a `security_considerations` section verdict (reviewer schema bumped to `1.3`) |
| [`stride-security-review`](#stride-security-review) | 2.3.0 | AI-powered security review of code changes — seven framework rule packs (Android, Django, Express, iOS, Phoenix, Rails, React/Next.js), CI/CD pack, defense-in-depth pack, with SARIF / CI integration |
| [`stride-ideation`](#stride-ideation) | 0.7.0 | Turn a fuzzy idea into a committed requirements doc and Stride tasks via two slash commands. v0.7.0: four-layer resilience model on `/stridify` (preflight advisory, `--goal` per-seam partitioning, bounded subagent-dispatch retry, retry-exhaustion fallback) |
| [`stride-lite`](#stride-lite) | 0.10.0 | File-only lightweight companion — writes Stride-shaped goal and task markdown to disk via `/stride-lite:create-goal` / `/stride-lite:create-task` / `/stride-lite:init`. v0.9.0+ harness-enforced `.stride_lite.md` hook auto-fire (PreToolUse/PostToolUse, cross-platform); v0.10.0+ terminal PENDING → IMPLEMENTED archive move in `stride-lite-workflow` |

---

## stride

Task lifecycle skills for Stride kanban: claiming, completing, creating tasks and goals, enriching tasks, workflow orchestration, subagent dispatch, and automatic hook execution via Claude Code hooks.

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
