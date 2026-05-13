# Stride Marketplace

Three plugins for [Claude Code](https://docs.claude.com/en/docs/claude-code) that integrate with the [Stride](https://www.stridelikeaboss.com) task management platform — task lifecycle, AI-powered security review, and structured ideation. Each plugin is independent and can be installed on its own.

## Installation

Add the marketplace to Claude Code once:

```bash
/plugin marketplace add cheezy/stride-marketplace
```

Then install any subset of the plugins below.

## Available Plugins

| Plugin | Version | One-line summary |
|---|---|---|
| [`stride`](#stride) | 1.12.0 | Task lifecycle for Stride kanban — claim, complete, create, decompose, with automatic Claude Code hook execution |
| [`stride-security-review`](#stride-security-review) | 2.2.0 | AI-powered security review of code changes with per-framework rule packs and SARIF / CI integration |
| [`stride-ideation`](#stride-ideation) | 0.5.0 | Turn a fuzzy idea into a committed requirements doc and Stride tasks via two slash commands |

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
- **Framework rule packs** (substantially expanded in 2.2.0) — Django/Python (8 rules), Phoenix/Elixir (9 rules), Rails/Ruby (8 rules). Includes open-redirect across all three, deserialization (`pickle` / `yaml` / `signing.loads` and `Marshal` / `YAML` / `YAML.unsafe_load`), Django SSRF with AWS IMDS / GCP / Azure metadata focus, DRF `ModelSerializer fields='__all__'`, production-settings hardening, Phoenix.Token replay, `System.cmd` shell-wrapper injection, LiveView upload guards, Rails `connection.execute` injection, missing `authenticate_user!`, and unfiltered JSON model render leaking `password_digest` / `*_token` / `*_secret` / `encrypted_*`.
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
- A TAP 13 eval runner (`scripts/run_eval.sh`) over 40+ fixtures with both positive-control (assert finding fires) and negative-control (assert finding does NOT fire) coverage. Includes its own CI workflow that re-runs the eval on every change to the agent prompt.

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

**Repository:** <https://github.com/cheezy/stride-ideation>

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
