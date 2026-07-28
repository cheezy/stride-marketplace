# Changelog

All notable changes to the Stride marketplace pin set will be documented in this file.

## [1.67.0] - 2026-07-28

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.41.0` to **`1.42.0`** so `/plugin update stride@stride-marketplace` pulls the new release. v1.42.0 adds the missing worked instance of `category: "security"` to the `task-reviewer` schema of record: the category was mandated in four prose places and demonstrated in no example anywhere in the tree, while `category: "project_check"` was both mandated *and* shown — and models copy examples far more reliably than prose, so a `failed` `security_considerations` verdict routinely shipped without the backing issue the fail-closed escalation and Consistency rules require. The worked example now carries an `important`-severity security issue over a synthetic unaddressed bounds-check, its second `considerations[]` entry flipped to `unmitigated`, and its section verdict flipped to `failed`, with `issue_counts`, the example `summary`, and the preamble updated so the three legs visibly move together and the Important-unless-exploitable severity split is demonstrated rather than only stated. Marketplace `metadata.version` bumped from `1.66.0` to `1.67.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table to version `1.42.0` with a `v1.42.0+` clause covering the above.

### Backward compatibility

Prompt-and-documentation changes only. No `.stride.md`, `.stride_auth.md`, hook, wire-shape, or reviewer-schema change — `schema_version` stays `1.6`, and the seven-value `issues[].category` enum the example draws from was already legal (the Kanban server widened to seven in W1940). No plugin pin other than `stride` is touched.

### Note on versioning (D177)

This entry was written by **D177**, whose job was to reconcile this catalog's self-version record. Five findings, recorded so the next engineer does not re-investigate:

- **The drift D177 was filed against is already fixed.** The defect described `metadata.version` stuck at `1.63.0` while tags reached `v1.65.0`. The `[1.66.0]` entry below closed exactly that gap and recorded the two releases (`v1.64.0`, `v1.65.0`) that had been tagged without a bump. No `metadata.version` backfill remains to do, and those two GitHub releases are deliberately not rewritten. (The *changelog* gap is a separate matter — see the fourth bullet.)
- **What was actually missing was this entry.** The commit that bumped the pin to `1.42.0` and `metadata.version` to `1.67.0` landed with its README sync but without a CHANGELOG entry or a tag — a half-finished release under the repo's current single-commit convention. D177 completes `1.67.0` rather than bumping past it; bumping to `1.68.0` would have left `1.67.0` permanently unrecorded, which is the very drift class this defect exists to end.
- **The untagged sync commits are not missed releases — decision: leave them as-is.** `eeaed1e`, `a44d82b` and `5e33131` are each the *first half* of a completed two-commit release from the `v1.61.0`–`v1.63.0` era, where an untagged "Sync … catalog entry" commit bumped only a plugin's own `version`/`description` and the following tagged commit bumped `metadata.version`. Each has its tagged partner (`4aff71f`, `ea23202`, `d69e1ee`). Tagging them now would manufacture releases for intermediate states no user ever resolved through. From `v1.64.0` onward the repo uses a single consolidated commit instead; that is the convention going forward, and it is now written down in the README's **Releases and tagging** section.
- **The changelog gap is far wider than `[1.66.0]` recorded — 32 of 63 tags, not two.** In the recent window (every tag from `v1.50.0` forward) the unrecorded releases are **`v1.58.0`, `v1.59.0`, `v1.61.0`, `v1.62.0`, `v1.64.0` and `v1.65.0`** — six, where `[1.66.0]` named only the last two, and where of the three two-commit pairs above only `d69e1ee` (`v1.63.0`) wrote an entry at all (`4aff71f` and `ea23202` touched `marketplace.json` and `README.md` only). Across the full 63-tag history the count is **32**, including a contiguous `v1.16.0`–`v1.27.0` block. **Decision: record, do not backfill**, on the same reasoning `[1.66.0]` gave — those releases already exist on GitHub, and retroactive entries dated today would describe pin states from weeks or months ago that later releases have superseded. The trail is here so the next engineer finds the audit instead of repeating it. Going forward the README's **Releases and tagging** section makes the entry one of the four things a release is, so the gap should stop growing.
- **A second surface drifted alongside the self-version: the per-plugin README prose.** All six rows of the README's *Available Plugins* table are current with their pins, but the per-plugin *prose sections* below them are not — `## stride` prose stops at a `v1.40.0+` clause against a `1.42.0` pin, `stride-ideation` and `stride-lite` stop at `v0.10.0` against `0.11.0` pins, and `## stride-security-review` carries no version clause at all across five pin bumps. The release ritual has been syncing the table row and not the prose. Also stale: the security-review section says the eval runs over "64 fixtures" where the plugin now ships 72 — the same hardcoded-count pattern that plugin's own CI guard was added to forbid. **Not fixed here** — D177's scope is the catalog's self-version record, and re-synchronising four cumulative prose narratives is its own piece of work. Recorded as follow-up, and the README's new **Known gaps** note points at it so the rule and the reality are not silently in conflict.

## [1.66.0] - 2026-07-28

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.40.0` to **`1.41.0`** so `/plugin update stride@stride-marketplace` pulls the new release. v1.41.0 hardens every rule that reads `behaviour_test_matrix` row text, which is authored by whoever created the task and is attacker-controlled at the API boundary: row text is data to assess and never instructions to follow; the secret rule triggers on row *state* rather than agent intent and extends to credentials named by location (file path, env var, vault reference, CI/CD or platform secret, Kubernetes Secret, git object, database row); a refused row gains a named reporting channel (`completion_notes`, identified by `category` and position rather than quoted) plus a `[REDACTED — row text embedded a credential]` sentinel for the reviewer's verbatim echo, with the resulting `failed` verdict documented as the *expected* outcome of a correct refusal; and the PATCH-body contradiction is resolved — because `PATCH /api/tasks/:id` replaces the whole array and a non-empty matrix must cover all seven categories, recording any row's advance necessarily re-sends every row, so re-sending already-stored row text byte-for-byte unchanged onto its own record is stated to be not a new copy, with one named action and a narrowly scoped exception. The same release corrects two premises the guidance stated as fact: `completion_notes` is persisted by Stride servers from D188 onward (deployment-conditional wording; the `completion_summary` duplication rule is unchanged), and matrix row rendering is defended by auto-escaped interpolation plus server-side enum validation rather than by the authoring no-raw-HTML convention. Marketplace `metadata.version` bumped from `1.63.0` to `1.66.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table to version `1.41.0` with a `v1.41.0+` clause covering the above.

### Note on versioning

`metadata.version` and this changelog had drifted behind the tag series: **v1.64.0** (sync `stride-security-review` to 2.5.1) and **v1.65.0** (sync the `stride` entry to 1.40.0) were tagged and released without a matching `metadata.version` bump or changelog entry. This release skips 1.64.0/1.65.0 in the file and resumes at **1.66.0** so `metadata.version`, this changelog, and the tag series agree again. The two skipped versions are recorded here rather than backfilled, since their releases already exist on GitHub.

### Backward compatibility

Prompt-and-documentation changes only. No `.stride.md`, `.stride_auth.md`, hook, wire-shape, or reviewer-schema change — `schema_version` stays `1.6`. The `behaviour_test_matrix` field remains OPTIONAL and is still not one of the five review_queue-scored fields.

## [1.63.0] - 2026-07-22

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.38.0` to **`1.39.0`** so `/plugin update stride@stride-marketplace` pulls the new release. v1.39.0 adds an optional, gracefully-degrading integration with the [`stride-security-review`](https://github.com/cheezy/stride-security-review) plugin (G-5613): the `task-reviewer` `security_considerations` verdict gains an OPTIONAL nested `considerations[]` breakdown (reviewer `schema_version` `1.4` → `1.5`) with a fail-closed escalation rule; `stride-workflow` Step 5 (and a matching `stride-subagent-workflow` orthogonal Decision-Matrix trigger) gains a gated **Deep security-considerations review** sub-step that dispatches the `stride-security-review:security-reviewer` in considerations mode — diff + considerations framed as data, never instructions — merging its per-consideration verdicts into `reviewer_result.security_considerations.considerations[]` via the existing verbatim whole-object passthrough and escalating fail-closed (any `partial`/`unmitigated` verdict forces the section `status` to `failed` and appends a `category: security` Critical issue); the dispatch time folds into the existing reviewer step (no new `workflow_steps` name) and `stride-completing-tasks` documents the automatic nested-array passthrough plus a matching self-check consistency item. The entry `description` was synced in the prior commit. Marketplace `metadata.version` bumped from `1.62.0` to `1.63.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table to version `1.39.0` with a `v1.39.0+` clause, and added a `v1.39.0+` paragraph to the `## stride` per-plugin section, both documenting the optional deep security-considerations review integration.

### Backward compatibility

The integration is **optional, Claude-Code-only, and gated on plugin availability**. When the `stride-security-review` plugin is not installed, the task's `security_considerations` is empty (or a `None — …` placeholder), or the environment is not Claude Code, the workflow falls back to the task-reviewer's prose verdict with no failure. No `.stride.md`, `.stride_auth.md`, hook, or wire-shape change.

## [1.60.0] - 2026-07-20

### Added

- **`.claude-plugin/marketplace.json`** — Added a new **`stride-exploratory-testing`** plugin entry (`source: url` → `https://github.com/cheezy/stride-exploratory-testing.git`, `version` **`0.1.0`**, `strict: true`) so `/plugin install stride-exploratory-testing@stride-marketplace` resolves. The plugin drives structured, charter-based exploratory testing sessions in Claude Code — the "explored" half of *Tested = Checked + Explored* — with five skills (`stride-exploratory-testing`, `chartering`, `heuristics`, `oracles`, `session`), five slash commands (`/charter`, `/nightmare-headline`, `/explore`, `/recon`, `/debrief`), two subagents (`charter-generator`, `explorer`), worked fixtures, and a pure-shell smoke-test harness. Marketplace `metadata.version` bumped from `1.59.0` to `1.60.0`.
- **`README.md`** — Added `stride-exploratory-testing` to the Available Plugins table and a detailed per-plugin section (surface summary, install command, repository link).

## [1.57.0] - 2026-07-10

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.34.0` to **`1.35.0`** so `/plugin update stride@stride-marketplace` pulls the new release. v1.35.0 ships the `changed_files` diff-upload correctness batch on top of v1.34.0: **(D127)** the `after_doing` finalize and `before_review` self-heal now target the PUT at the task id parsed from the authoritative `/complete`|`/mark_reviewed` command URL (a new `task_id_from_command` helper) instead of the claim-time env-cache `TASK_ID`, so a stale or corrupt cache — a piped/truncated claim capture or a host restart before the on-disk cache reloads — can no longer upload the diff to the **previous** task; **(W1658)** when the `before_review` self-heal (the last retry) still returns a non-2xx, the hook logs a distinct `CHANGED_FILES UPLOAD UNRESOLVED` message and appends `unresolved=yes` to `.stride-diff-upload-state` (fail-soft — never vetoes the completion; a later successful PUT self-clears the mark); **(D126)** a root-cause doc + reproduction test for the empty-`changed_files` failure; and **(W1661)** explicit stdout-preserving curl invocation rules across the claiming/completing/workflow skills. Both hook changes land in **`stride-hook.sh` and `stride-hook.ps1`** with tests. The entry description is unchanged. Marketplace `metadata.version` bumped from `1.56.0` to `1.57.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table to version `1.35.0` with a `v1.35.0+` clause noting the URL-id targeting fix (D127) and the fail-loud terminal-upload signal (W1658), both mirrored in the PowerShell hook with tests.

### Backward compatibility

Pin-only change for `stride`; the other plugin pins (`stride-security-review` `2.4.2`, `stride-ideation` `0.11.0`, `stride-lite` `0.11.0`, `launchdarkly` `0.3.0`) are unchanged. stride v1.35.0 is fully backward compatible — no `.stride.md`, wire-shape, or `.stride_auth.md` change; the `unresolved=yes` marker is additive to the already-gitignored `.stride-diff-upload-state` and self-clears on the next successful upload. The pin is URL-based, so no re-vendoring is required.

## [1.56.0] - 2026-07-08

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.33.0` to **`1.34.0`** so `/plugin update stride@stride-marketplace` pulls the new release. v1.34.0 makes the `## after_goal` hook fire **reliably regardless of `/complete` response size** (G313). Root cause: the Claude Code harness truncates a large `tool_response.stdout` mid-JSON (the server echoes the full ~46KB `reviewer_result`), so `after_goal` detection parsed invalid JSON and the local push silently never ran — the goal reached Done via the grace worker (status-only, no push), leaving the task branch's commit unmerged. The fix is two-layered and lands in **both** `stride-hook.sh` and `stride-hook.ps1`: a canonical full-response file (`.stride/.last-api-response.json`) is preferred over the truncatable stdout (D118/W1609), and — when that is absent or truncated — the hook spawns a fresh `GET /api/tasks/:id/after_goal_status` call (D119), a subprocess not subject to Bash-tool truncation that needs zero agent cooperation and is THE reliability guarantee; the two paths are mutually exclusive so `## after_goal` runs at most once, and against a server without the endpoint the fresh call is a clean no-op. Also: the shared payload resolver now also feeds the claim `TASK_BASE_REF` refresh (closing an oversized-claim divergence), the response-capture curl pattern and a push-verification step are documented (W1610), and end-to-end truncation tests were added (suites now **388 bash / 270 pwsh**). A W1614 spike confirmed a `PreToolUse` command-rewrite enforcement is security-gated by the harness and not viable — the shipped canonical-file + fresh-call architecture is the correct design. Port parity for the five port plugins is follow-up work. The entry description is unchanged. Marketplace `metadata.version` bumped from `1.55.0` to `1.56.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table to version `1.34.0` with a `v1.34.0+` clause noting the size-independent `## after_goal` detection (canonical response file + hook-initiated fresh `GET .../after_goal_status` guarantee, PowerShell parity, 388/270 test suites).

### Backward compatibility

Pin-only change for `stride`; the other plugin pins (`stride-security-review` `2.4.2`, `stride-ideation` `0.11.0`, `stride-lite` `0.11.0`, `launchdarkly` `0.3.0`) are unchanged. stride v1.34.0 is fully backward compatible — the new `.stride/.last-api-response.json` lives under the already-gitignored `.stride/` directory and is excluded from snapshots by name, and a server without the `GET /api/tasks/:id/after_goal_status` endpoint simply falls back to a clean no-op (the grace worker still flips the goal to Done). The pin is URL-based, so no re-vendoring is required.

## [1.55.0] - 2026-07-02

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `launchdarkly` plugin pin from `0.2.0` to **`0.3.0`** so `/plugin update launchdarkly@stride-marketplace` pulls the new release. v0.3.0 is the client-side-era-modernization release (W1488–W1493): the `launchdarkly-typescript-client` skill now teaches the scoped v4 browser and React SDKs (`@launchdarkly/js-client-sdk` with `createClient`/`start()` and typed variation methods; `@launchdarkly/react-sdk` with `createLDReactProvider` and the typed hooks, `useFlags` deprecated) with the unscoped v3 packages preserved in a clearly-labeled legacy subsection; `/launchdarkly:ld-scaffold` gains real **browser** and **react** scaffold targets (four targets total, the previously unreachable branch removed) and `/launchdarkly:ld-remove-flag` becomes React-hook-aware; the `launchdarkly-reviewer` agent grows from six to seventeen anti-pattern rules (the eight the goal specified plus three audit-found parity closures) with a standing skill-to-reviewer parity note; the Java `TestData` examples are aligned with a drift-resistant version placeholder; the README and `launchdarkly-fundamentals` frontmatter are corrected to match the shipped plugin; and the plugin now ships a dependency-free `scripts/smoke.sh` validation gate. The entry description gained a `v0.3.0+:` clause in its existing inline style. Marketplace `metadata.version` bumped from `1.54.0` to `1.55.0`.
- **`README.md`** — Updated the `launchdarkly` row in the `Available Plugins` table to version `0.3.0` with a `v0.3.0+` clause noting the scoped v4 SDK modernization, the new browser/react scaffold targets, the expanded reviewer rule set, and the new smoke-script validation gate.

### Backward compatibility

Pin-only change for `launchdarkly`; the other plugin pins (`stride` `1.33.0`, `stride-security-review` `2.4.2`, `stride-ideation` `0.11.0`, `stride-lite` `0.11.0`) are unchanged. launchdarkly v0.3.0 is additive — the existing `java` and `typescript` scaffold targets and the original six reviewer rules are preserved; the v3 client-side guidance is demoted to a labeled legacy section rather than removed. The pin is URL-based, so no re-vendoring is required.

## [1.54.0] - 2026-07-02

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride-lite` plugin pin from `0.10.0` to **`0.11.0`** so `/plugin update stride-lite@stride-marketplace` pulls the new release. v0.11.0 is an accuracy-and-installer release (W1477–W1483). The headline fix: **copy installs now ship the hook enforcement layer** — `install.sh` previously copied every plugin directory except `hooks/`, so script-based installs silently lacked the documented PreToolUse/PostToolUse auto-fire entirely; pre-0.11.0 copy installs should re-run `./install.sh --force`. The installer now copies `hooks/` with the executable bit preserved, reports it in the install summary, and fails loudly before any success banner if `hooks.json` does not land. Alongside it: the init command/skill, the workflow walkthrough, and the scaffolded `.stride_lite.md` template all teach the v0.9.0 harness-executes model (the stale "static configuration" and workflow-skill-executes claims are gone, and the walkthrough no longer instructs the double execution the harness migration removed — its Step 8 now ends with the PENDING→IMPLEMENTED archive move); phantom `/stride-lite:ideate`/`/stride-lite:decompose` references are purged and the create-decomposer's unjustified `Read, Grep` grant removed after verifying both dispatch sites pass requirements text inline; the README's version label, command count, and step-count claims are corrected with drift-resistant wording; the smoke suite grows from 24 to 43 assertions with machine-enforced init-template parity and nine hook-routing payload fixtures; and the shipped init-command goal artifact is archived per the plugin's own convention. The entry description gained a `v0.11.0+:` clause in its existing inline style. Marketplace `metadata.version` bumped from `1.53.0` to `1.54.0`.
- **`README.md`** — Updated the `stride-lite` row in the `Available Plugins` table to version `0.11.0` with a `v0.11.0+` clause noting the installer enforcement-layer fix (and the reinstall guidance), the harness-executes wording, and the 43-assertion smoke suite.

### Backward compatibility

Pin-only change for `stride-lite`; the other plugin pins (`stride` `1.33.0`, `stride-ideation` `0.11.0`, `stride-security-review` `2.4.2`, `launchdarkly` `0.2.0`) are unchanged. stride-lite v0.11.0 changes installer behavior only additively (the copy set gains `hooks/`; symlink installs are untouched) and its remaining changes are documentation, tests, and one archival move — no command surface, output schema, or hook-script behavior changed. Users who installed via `./install.sh` before 0.11.0 should re-run it with `--force` to gain the enforcement layer they were missing.

## [1.53.0] - 2026-07-02

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride-security-review` plugin pin from `2.4.1` to **`2.4.2`** so `/plugin update stride-security-review@stride-marketplace` pulls the new release. v2.4.2 is a documentation-accuracy release (W1469–W1475; the analysis rules, flags, and wire schema are unchanged): the skill and command docs are unified on the fifteen-value `vulnerability_class` contract (supply chain restored to the universal list, the five MAESTRO-derived agentic classes named, the enum owned by the agent prompt and referenced — not restated — elsewhere); the framework-pack count now matches the seven-pack inventory with Express no longer offered as future work; the README CI gating snippet quotes the shipped workflow's fail-closed `-ne 0` gate verbatim instead of the fail-open `-eq 1` it documented (anyone who copied the old snippet had a gate that passed when the reviewer crashed); the `--rci` out-of-range contradiction is resolved with unambiguous consume-and-clamp semantics and a worked five-row input table; the skill's Customization section names all ten implemented flags with the command as the single semantics owner; the README eval-count annotation is genuinely count-agnostic with a scoped drift guard added to `scripts/check_fixtures.sh` (rides the existing CI step); and the CHANGELOG footer link set is complete through the current release with a release-ritual comment. Marketplace `metadata.version` bumped from `1.52.0` to `1.53.0`.
- **`README.md`** — Updated the `stride-security-review` row in the `Available Plugins` table to version `2.4.2` with a `v2.4.2:` clause noting the documentation-accuracy fixes and the fixture-count drift guard.

### Backward compatibility

Pin-only change for `stride-security-review`; the other plugin pins (`stride` `1.33.0`, `stride-ideation` `0.11.0`, `stride-lite` `0.10.0`, `launchdarkly` `0.2.0`) are unchanged. The stride-security-review v2.4.2 change is documentation and one read-only CI guard — no change to the analysis rules, the slash-command flags, or the JSON/SARIF wire shape. Repos that consumed v2.4.1 output continue to parse unchanged; the corrected CI snippet strictly strengthens the documented gate (fails closed on dispatch errors instead of passing silently), so pipelines that copy it get safer, not different, behavior on healthy runs.

## [1.52.0] - 2026-07-02

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride-ideation` plugin pin from `0.10.0` to **`0.11.0`** so `/plugin update stride-ideation@stride-marketplace` pulls the new release. v0.11.0 is an accuracy-and-behavior release (G296 / W1460–W1468): the `requirements-decomposer` contract, all three calibration fixtures (35 tasks), and the batch validator now enforce and model the **five review-queue scored fields** (`acceptance_criteria`, `testing_strategy`, `security_considerations`, `pitfalls`, `patterns_to_follow`) — the decomposer skeleton carries all five with a none-with-reason convention, the fixtures model them with scenario-realistic values, and `lib/validate_batch.py` gains an advisory scored-field completeness pass (stdout warnings, exit code unchanged) plus a fatal `length_limit` check failing oversized `title`/`security_considerations` elements at the server's real varchar(255) code-point bound (`pitfalls`/`key_files` are deliberately excluded — they are unbounded JSONB server-side). `/stridify` Step 8b now stamps `created_by_agent` on every shipped goal (server propagates to child tasks; guaranteed to survive the pre-POST strip with a round-trip regression) so batches are attributed in the `/agents` feed. Documentation corrections: the reviewer check-count contradiction (three → five), the ideate enforcement list gains the round-4 premortem and lean-startup round-5 MVP-design bullets, stale version/smoke-test/test-count claims refreshed in release-stable wording, stale `/decompose`//`ship` command references corrected across six lib scripts, and `drift_check.py` reframed as a documented standalone helper. Marketplace `metadata.version` bumped from `1.51.0` to `1.52.0`.
- **`README.md`** — Updated the `stride-ideation` row in the `Available Plugins` table to version `0.11.0` with a `v0.11.0:` clause noting the five-scored-field alignment, `created_by_agent` attribution, and validator hardening.

### Backward compatibility

Pin-only change for `stride-ideation`; the other four plugin pins (`stride` `1.33.0`, `stride-security-review` `2.4.1`, `stride-lite` `0.10.0`, `launchdarkly` `0.2.0`) are unchanged. stride-ideation v0.11.0 keeps both commands' surfaces unchanged — the validator's new length check only fails batches the production server would reject anyway, and its scored-field warnings never change the exit code.

## [1.51.0] - 2026-07-02

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.32.0` to **`1.33.0`** so `/plugin update stride@stride-marketplace` pulls the new release. v1.33.0 ships hook executor hardening in both `stride-hook.sh` and `stride-hook.ps1`: per-hook timeouts enforced per the documented budgets (section-spanning, server-supplied values take precedence, process-group kills, exit-124 reporting, macOS watchdog fallback), server-supplied hook env forwarding so `## after_goal` receives the `GOAL_*` vars, real `duration_ms` telemetry in the hook stdout JSON (`duration_seconds` deprecated, kept one release), quote-aware backslash line continuation in the `.stride.md` parser (the docs' canonical multi-line `gh pr create` example now runs as one command), and a claim-time dirty-baseline guard so pre-existing unrelated working-tree edits never pollute `changed_files` — with the hook configuration and auth dot-files hard-excluded by name from every snapshot and upload (the auth file can never be uploaded). Plus a documentation accuracy sweep: stale five-field completion claims replaced with pointers to the Completion Request Field Reference table, `hook-diagnostician` gains `after_goal` coverage, workflow step numbering is contiguous again (Steps 6–9 → 5–8), and the capture docs now match the executor (auth-file-first credentials, base64 transport envelope, nested-repo limitation) (G287 / W1449–W1458). **The five port plugins (`stride-codex`, `stride-gemini`, `stride-opencode`, `stride-copilot`, `stride-pi`) do not yet mirror these changes — port parity is follow-up work.** Marketplace `metadata.version` bumped from `1.50.0` to `1.51.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table to version `1.33.0` with a `(v1.33.0+)` clause noting the executor hardening and accuracy sweep.

### Backward compatibility

Pin-only change for `stride`; the other four plugin pins (`stride-security-review` `2.4.1`, `stride-ideation` `0.10.0`, `stride-lite` `0.10.0`, `launchdarkly` `0.2.0`) are unchanged. stride v1.33.0 keeps `.stride.md` files working unmodified, with two behavioral notes: hook commands now run in fresh shells per logical line (a `cd` on one line no longer affects the next), and hook sections exceeding their documented budget now fail with exit 124 instead of silently consuming the 300s harness ceiling.

## [1.50.0] - 2026-07-01

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.31.0` to **`1.32.0`** so `/plugin update stride@stride-marketplace` pulls the new release. v1.32.0 adds an **API Notes & Limitations** section to the `stride-workflow` orchestrator skill documenting two recurring API gotchas: tasks cannot be reparented (`parent_id` is creation-only) and there is no DELETE endpoint — moving/removing a task is a human board-UI action, never a recreate-as-supersede workaround; and raw HTTP calls must use curl or a curl/browser-like User-Agent because the hosted API edge returns `403` `error code: 1010` to default library User-Agents (G286 / W1416). Marketplace `metadata.version` bumped from `1.49.0` to `1.50.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table to version `1.32.0` with a `(v1.32.0+)` clause noting the API Notes & Limitations documentation.

### Backward compatibility

Pin-only change for `stride`; the other four plugin pins (`stride-security-review` `2.4.1`, `stride-ideation` `0.10.0`, `stride-lite` `0.10.0`, `launchdarkly` `0.2.0`) are unchanged. The stride v1.32.0 change is documentation/skill-text only — no wire-shape, hook, or auth changes.

## [1.49.0] - 2026-06-29

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.30.1` to **`1.31.0`** so `/plugin update stride@stride-marketplace` pulls the new release. v1.31.0 gives the `/stride:create-tasks` and `/stride:create-goals` commands an explicit **terminal state** — after creating, the `stride-workflow` orchestrator reports the new identifiers, clears the activation marker, and stops instead of falling through into the build loop and auto-claiming/building the just-created task — plus a **Backlog claim-fail guard** so a failed claim never falls back to building outside the lifecycle (G284 / W1400). Marketplace `metadata.version` bumped from `1.48.0` to `1.49.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table to version `1.31.0` with a `(v1.31.0+)` clause noting the create terminal-state and Backlog claim-fail guard.

### Backward compatibility

Pin-only change for `stride`; the other four plugin pins (`stride-security-review` `2.4.1`, `stride-ideation` `0.10.0`, `stride-lite` `0.10.0`, `launchdarkly` `0.2.0`) are unchanged. The stride v1.31.0 change is documentation/skill-text only — no wire-shape, hook, or auth changes; the build loop is unchanged.

## [1.45.0] - 2026-06-22

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride-security-review` plugin pin from `2.3.0` to **`2.4.0`** so `/plugin update stride-security-review@stride-marketplace` pulls the new release. v2.4.0 is a test-coverage, tooling, and documentation-accuracy release (the analysis rules are unchanged): golden-file tests for the deterministic SARIF/dedup/fingerprint/fail-on transforms; positive-control fixtures for the four previously-uncovered CI/CD platforms (Azure Pipelines, Drone, Jenkins, Tekton) and the two missing supply-chain sub-rules (lockfile-drift, typosquat), taking the eval suite from 64 to **70 fixtures**; a fixture/`EXPECTED.md` parity guard wired into CI; per-finding schema validation in the eval runner; the SARIF `tool.driver.version` drift fix (now tracks `plugin.json`); and several documentation-drift corrections (README eval count, the `EXPECTED.md` full-scan scenario, the CHANGELOG footer). The entry description's fixture count was updated `64` → `70`. Marketplace `metadata.version` bumped from `1.44.0` to `1.45.0`.
- **`README.md`** — Updated the `stride-security-review` row in the `Available Plugins` table to version `2.4.0` with a `v2.4.0` clause noting the 70-fixture coverage and golden-file transform tests.

### Backward compatibility

Pin-only change for `stride-security-review`; the other three plugin pins (`stride` `1.30.0`, `stride-ideation` `0.8.0`, `stride-lite` `0.10.0`) are unchanged. The stride-security-review v2.4.0 change adds test coverage, internal tooling, and documentation fixes plus the SARIF driver-version metadata fix — no change to the analysis rules, the slash-command flags, or the JSON/SARIF wire shape beyond `driver.version` now reflecting the real plugin version. Repos that consumed v2.3.0 output continue to parse unchanged.

## [1.43.0] - 2026-06-19

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.28.0` to **`1.29.0`** so `/plugin update stride@stride-marketplace` pulls the new release. v1.29.0 documents the **`technical_details` task field** across the plugin (G243): an optional, free-form JSON object (arbitrary keys/values) a task may carry for any additional technical context that does not fit the structured fields — data shapes, gotchas, key decisions, reference links. Unlike `testing_strategy` it has no fixed keys, and it is **not** one of the five review_queue-scored fields (`acceptance_criteria`, `testing_strategy`, `security_considerations`, `pitfalls`, `patterns_to_follow`), so a blank `{}` is never a scoring gap. The creation contracts (`stride-creating-tasks` W1179, `stride-creating-goals` W1179), the enrichment/decomposition guidance (`task-enricher` + `stride-enriching-tasks` W1180, `task-decomposer` W1180), and the workflow/exploration references (`stride-workflow` Step 1 W1181, `task-explorer` W1181) all describe the field consistently — optional, free-form, never fabricated, with a no-secrets reminder. Marketplace `metadata.version` minor-bumped from `1.42.0` to `1.43.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table to version `1.29.0` with a `v1.29.0+` clause; added a `v1.29.0+` paragraph to the `## stride` section documenting the new `technical_details` field.

### Backward compatibility

Pin-only change for the `stride` plugin; the other three plugin pins (`stride-security-review` `2.3.0`, `stride-ideation` `0.8.0`, `stride-lite` `0.10.0`) are unchanged. The stride v1.29.0 change is documentation-only (see the stride plugin CHANGELOG `1.29.0`) — no `.stride.md`, `.stride_auth.md`, hook, or Stride API wire-shape change; `technical_details` is optional everywhere and never added to any scored-field set, so tasks that omit it behave exactly as before.

## [1.42.0] - 2026-06-15

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride-ideation` plugin pin from `0.7.0` to **`0.8.0`** so `/plugin update stride-ideation@stride-marketplace` pulls the new release. v0.8.0 turns `/stride-ideation:ideate` into a guided, recoverable, human-in-control session with six additive features — (1) a display-only per-round section-completeness recap (the seven gated sections shown `solid`/`thin`/`empty` before every round; never changes the gate, round order, or `<=4`-question budget), (2) an "I'm not sure — propose candidates" uncertainty path on every gated-section and forcing question (proposes 2–4 topic-tailored candidates with rationales; a candidate never satisfies the gate until the human confirms it), (3) a profile recommendation before the rounds when `--profile` is omitted (recommended-first, lean default; explicit `--profile` skips it), (4) a `--input <path>` brain-dump seed read read-only to pre-fill draft sections (distinct from and composable with `--continue`; the file is never modified, moved, or committed), (5) intra-session draft autosave/resume via a new pure-shell `lib/draft.sh` that persists the in-progress draft after every round to a gitignored `.stride/` scratch file and offers resume-or-fresh for the same slug, deleting the scratch after a successful commit (never holds the token), and (6) advisory `requirements-reviewer` findings surfaced as a multi-select human decision with an explicit "Address none — write as-is" choice (at most one refinement round; reviewer never blocks the write). `/stride-ideation:stridify` gains a **preview-and-approval gate** before the POST (renders the decomposed goal/task tree + cross-goal claim order and requires explicit approval; decline stops cleanly with the committed batch JSON intact and no POST) plus a `--yes` / `--auto-approve` bypass for scripted callers. Test coverage grows from 109 to 155 assertions across 11 suites (3 new suites: `test-stridify-preview`, `test-ideate-input`, `test-draft`). The `stride-ideation` marketplace entry description gains a `v0.8.0` clause. Marketplace `metadata.version` minor-bumped from `1.41.0` to `1.42.0`.
- **`README.md`** — Updated the `stride-ideation` row in the `Available Plugins` table to version `0.8.0` with a `v0.8.0` summary; added a `v0.8.0+` paragraph to the `## stride-ideation` section documenting the six `/ideate` session features and the `/stridify` preview-and-approval gate with `--yes` bypass.

### Backward compatibility

Pin-only change for the `stride-ideation` plugin; the other three plugin pins (`stride` `1.28.0`, `stride-security-review` `2.3.0`, `stride-lite` `0.10.0`) are unchanged. The stride-ideation v0.8.0 changes are command/skill-contract + pure-shell-helper additions (see the stride-ideation plugin CHANGELOG `0.8.0`) — no `.stride.md`, `.stride_auth.md`, or Stride API wire-shape change. A flag-free or `--profile=lean` `/ideate` run is byte-for-byte compatible on the happy path; the one automation-affecting change is that a non-interactive `/stride-ideation:stridify` now pauses for human approval unless `--yes` / `--auto-approve` is passed, and `.stride/` is a new gitignored draft-autosave directory.

## [1.41.0] - 2026-06-13

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.25.0` to **`1.28.0`** so `/plugin update stride@stride-marketplace` pulls the new release. v1.28.0 makes the **claim-time `TASK_BASE_REF` always refreshed (G224)**: the PostToolUse hook records the HEAD commit at claim time in `.stride-env-cache` and the `after_doing` flow diffs against it, but the cache was only written when the claim response parsed out of `tool_response.stdout` — an oversized claim response is persisted to a file by Claude Code (only a `Full output saved to:` notice reaches stdout), so the parse failed, the write was skipped, and a stale base ref from a previous claim survived, making `changed_files` span unrelated commits (D60 showed 27 files for a 3-file task). `hooks/stride-hook.sh` (W1086) adds a persisted-output file fallback (validated regular file, parsed with `jq` only, never sourced/eval'd) and makes the base-ref refresh unconditional on every claim (rewrite `TASK_BASE_REF` to current HEAD and clear the stale snapshot even when no JSON parses, preserving `TASK_` identity lines); `hooks/stride-hook.ps1` (W1087) ports the same for Windows parity (the ps1 hook previously never wrote `TASK_BASE_REF`) and fixes a pre-existing `Set-StrictMode` property-access hazard. Bash Test Group 14 and PowerShell Test Group 10 mirror test-for-test (bash 237, PowerShell 172 green). The pin also carries the previously-unreleased plugin v1.26.0 (D65/D66) and v1.27.0 (D67) changes. The stride plugin description in the marketplace entry gains a `v1.28.0+` clause. Marketplace `metadata.version` minor-bumped from `1.40.0` to `1.41.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table to version `1.28.0` and appended a `v1.28.0+` clause; added a `v1.28.0+` paragraph to the `## stride` section documenting the persisted-output fallback, the unconditional claim-time base-ref refresh, the PowerShell parity port, and the StrictMode fix.

### Backward compatibility

Pin-only change for the `stride` plugin; the other three plugin pins (`stride-security-review` `2.3.0`, `stride-ideation` `0.7.0`, `stride-lite` `0.10.0`) are unchanged. The stride v1.28.0 change is hook-script only — no wire-shape, hook-timeout, `.stride.md`, or `.stride_auth.md` change (see the stride plugin CHANGELOG `1.28.0`); a claim whose response parses inline behaves exactly as before, the new behavior only adds recovery paths for responses that previously left the cache stale.

## [1.39.0] - 2026-06-09

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.23.0` to **`1.24.0`** so `/plugin update stride@stride-marketplace` pulls the new release. v1.24.0 makes **every part of a dispatched review report mandatory on every task completion — NO EXCEPTIONS (goal G222)**: all review information the task supplies (`security_considerations`, `testing_strategy`, `patterns_to_follow`, `pitfalls`, `acceptance_criteria`) must be passed to the reviewer, and the reviewer's entire structured output (every project check and every section verdict) must reach the server intact. It is the plugin-side source fix for the recurring defect where a task's security considerations came back **`not_assessed`** and the project-checks list was silently **truncated** (3 of 26 reached the server): `skills/stride-workflow/SKILL.md` (W1072) now passes every review field the task supplies to the reviewer (it previously listed only four, omitting `security_considerations`); `agents/task-reviewer.md` (W1073/W1076) reserves `not_assessed` strictly for a section the task itself left empty and single-sources its input contract; the `reviewer_result` passthrough (W1074) is now a mechanical whole-object copy with a count-checked self-check; and `skills/stride-completing-tasks/SKILL.md` (W1075) adds a mandatory pre-submission self-check hard gate. The stride plugin description in the marketplace entry gains a `v1.24.0+:` paragraph. Marketplace `metadata.version` minor-bumped from `1.38.0` to `1.39.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table to version `1.24.0` and appended a `v1.24.0+` clause; added a `v1.24.0+` paragraph to the `## stride` section documenting the complete-delivery (NO EXCEPTIONS) review-report rule, the count-checked whole-object `reviewer_result` passthrough, and the pre-submission self-check hard gate.

### Backward compatibility

Pin-only change for the `stride` plugin; the other three plugin pins (`stride-security-review` `2.3.0`, `stride-ideation` `0.7.0`, `stride-lite` `0.10.0`) are unchanged. The stride v1.24.0 change is documentation/agent-prompt only — no wire-shape, hook, `.stride.md`, or `.stride_auth.md` change (see the stride plugin CHANGELOG `1.24.0`); it makes the plugin always emit the complete `reviewer_result` the schema already defines (stored as `:jsonb` and persisted verbatim since v1.22.1). The Kanban server independently hard-rejects an incomplete or task-inconsistent report (goal G221), so the two halves enforce completeness from both ends.

## [1.38.0] - 2026-06-08

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.22.1` to **`1.23.0`** so `/plugin update stride@stride-marketplace` pulls the new release. v1.23.0 adds a third `project_checks[]` per-entry `status` value, **`not_applicable`** (alongside `met`/`not_met`), to `agents/task-reviewer.md` and requires the reviewer to emit one entry for **every** top-level `CODE-REVIEW.md` bullet — bullets with no bearing on the diff are marked `not_applicable` with a one-line reason rather than omitted, so the Kanban review queue's "Code review" panel renders the full checklist instead of a partial one. `not_applicable` is approval-neutral (no paired `issues[]` entry, never triggers `changes_requested`); `schema_version` bumps `"1.3"` → `"1.4"`. The stride plugin description in the marketplace entry gains a `v1.23.0+:` paragraph. Marketplace `metadata.version` minor-bumped from `1.37.0` to `1.38.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table to version `1.23.0` and appended a `v1.23.0+` clause surfacing the `not_applicable` project-check status, the full-checklist emission requirement, and the `schema_version` `1.4` bump.

### Backward compatibility

Pin-only change for the `stride` plugin; the other three plugin pins (`stride-security-review` `2.3.0`, `stride-ideation` `0.7.0`, `stride-lite` `0.10.0`) are unchanged. The stride v1.23.0 wire change is additive and forward-compatible (see the stride plugin CHANGELOG `1.23.0`) — the Kanban server stores `reviewer_result` as `:jsonb` and persists it verbatim, so the new `not_applicable` status value flows through untouched; reviewers on an older `schema_version` omit it. The Kanban-server UI half (the review-queue "N/A" pill) ships independently.

## [1.36.0] - 2026-06-07

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.21.0` to **`1.22.0`** so `/plugin update stride@stride-marketplace` pulls the new release. v1.22.0 changes the `after_doing` hook (`hooks/stride-hook.sh` + `hooks/stride-hook.ps1`) to upload the per-file diff snapshot to `/api/tasks/:id/changed_files` as a **transport-encoded envelope** — `{"changed_files":{"encoding":"base64","data":"<single-line-base64>"}}` — instead of the raw `{"changed_files":[...]}` array, so an edge request filter (WAF) in front of the Stride server cannot misread a dense code diff as an attack payload and silently drop the upload (which left `changed_files` empty in the review queue); the server decodes the base64 back to the identical list. The hook falls back to the raw array shape when `base64` is unavailable (the object wrapper is preserved on both paths so the value never lands at `params['_json']` as NULL), and a non-2xx upload response is now surfaced as a stderr warning instead of being discarded (non-fatal to completion; the bearer token is never logged). The stride plugin description in the marketplace entry gains a `v1.22.0+:` paragraph. Marketplace `metadata.version` minor-bumped from `1.35.0` to `1.36.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table to version `1.22.0` and added a `v1.22.0+` paragraph to the `## stride` section documenting the transport-encoded `changed_files` upload, the raw-array fallback, and the non-2xx stderr warning.

### Backward compatibility

Pin-only change for the `stride` plugin; the other plugin pins (`stride-security-review` `2.3.0`, `stride-ideation` `0.7.0`, `stride-lite` `0.10.0`) are unchanged. The stride v1.22.0 wire change is additive (see the stride plugin CHANGELOG `1.22.0`) but requires the Kanban server to accept the `base64` / `gzip+base64` envelope on `/changed_files`, which ships independently in the kanban repo; the raw-array fallback path remains byte-compatible with the prior hook where `base64` is unavailable.

## [1.35.0] - 2026-06-06

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.20.0` to **`1.21.0`** so `/plugin update stride@stride-marketplace` pulls the new release. v1.21.0 elevates `security_considerations` to a first-class, review_queue-scored field at parity with `testing_strategy`: the three task-authoring skills (`stride-creating-tasks`, `stride-enriching-tasks`, `stride-creating-goals`) name it as the 5th scored field (callouts four → five) with an array-of-strings shape; the `task-enricher` and `task-decomposer` agents derive and emit it; `agents/task-reviewer.md` gains a **Security Considerations Alignment** review step plus a `security_considerations` `{ "status": "passed" | "failed" | "not_assessed" }` section-verdict object that confirms the considerations were actually implemented, the `issues[]` `category` enum gains `"security"`, and `schema_version` bumps `"1.2"` → `"1.3"`; and `skills/stride-completing-tasks/SKILL.md` + `skills/stride-workflow/SKILL.md` carry the verdict verbatim into `reviewer_result`. The stride plugin description in the marketplace entry gains a `v1.21.0+:` paragraph. Marketplace `metadata.version` minor-bumped from `1.34.0` to `1.35.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table to version `1.21.0` and refreshed its one-line summary to surface the `security_considerations` elevation, the `task-reviewer` Security Considerations Alignment step, the section verdict, and the `schema_version` `1.3` bump.

### Backward compatibility

Pin-only change for the `stride` plugin; the other three plugin pins (`stride-security-review` `2.3.0`, `stride-ideation` `0.7.0`, `stride-lite` `0.10.0`) are unchanged. The stride v1.21.0 wire changes are additive and forward-compatible (see the stride plugin CHANGELOG `1.21.0`) — the Kanban server stores `reviewer_result` as `:jsonb` and tolerates the new `security_considerations` key; reviewers on an older `schema_version` omit it. The Kanban-server UI half (rendering the security-considerations review result on the Review Queue) ships independently.

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
