# Changelog

All notable changes to the Stride marketplace pin set will be documented in this file.

## Release record — tags without a GitHub release

*This is a record-keeping note, not a release. It describes no change to this catalog and carries no version.*

A fleet-wide audit found **10 tags** in this repository that are tagged and pushed but have no corresponding GitHub release. **The gap is accepted and will not be backfilled.** It is recorded here so the next release engineer does not rediscover and re-litigate it:

- `v1.9.1` — 2026-04-17
- `v1.12.0` — 2026-05-08
- `v1.13.0` — 2026-05-11
- `v1.15.0` — 2026-05-11
- `v1.15.1` — 2026-05-11
- `v1.15.2` — 2026-05-11
- `v1.21.0` — 2026-05-13
- `v1.22.0` — 2026-05-13
- `v1.23.0` — 2026-05-13
- `v1.25.0` — 2026-05-19

**This is a different gap from the one the `[1.66.0]` and `[1.67.0]` entries record.** Those describe tags with no *changelog entry*; this note describes tags with no *GitHub release*. A tag can be missing either, both, or neither, and the two lists do not coincide.

Why accepted rather than backfilled:

- **Nothing resolved through these releases.** A GitHub release is a human-readable record, not a resolution mechanism — nothing installs *through* one. The missing releases cost nothing at the time and cost nothing now.
- **Backfilling would be worse than the gap.** A release created today against a commit from April or May would be dated today, and would manufacture a record for a state no user ever resolved through — misrepresenting the very history it claims to document.
- **The convention itself is unchanged.** These are omissions from a few release cycles, not a policy shift. Every tag still gets a release going forward.

The audit also found **zero** GitHub releases without a matching tag, so the record is incomplete in only this one direction.

## [1.83.0] - 2026-08-21

Four pins in one release. Every Claude Code plugin in this catalog that had unreleased work shipped on the same day, so they are recorded together rather than as four consecutive catalog numbers.

- **`stride` 1.69.0 -> 1.70.0.** Six hook-executor defects, four of them silent data-loss paths that produced a success-shaped result. **D278:** the capture listed paths without `-z`, so git's octal-escaped display spelling was written into the snapshot as the entry's path *and* fed back as a pathspec that matches nothing — every file under a non-ASCII name, or under a non-ASCII directory component, lost 100% of its diff content on every macOS and Linux agent, silently. A renamed *binary* was separately leaking a raw `Binary files … differ` body. **D279:** sizing each diff by substituting the newlines out goes quadratic on a long line — profiled at 37,063 ms for a 200KB single-line body against 34 ms for `git diff` itself — and the failure is invisible, because the hook is killed at the 120s budget while the completion still succeeds and Review simply shows no diffs. **D275:** the hook-env key filter was a deny-list over an open-ended dangerous set, so `PATH`, `BASH_ENV`, `ENV`, `IFS`, `LD_PRELOAD` and `GIT_SSH_COMMAND` reached both the `set -a` eval and the durable env cache from an API response body; it is now an allow-list of exactly the 17 documented names, with a drift guard in each suite against the table it was copied from. **D281:** the PowerShell executor read the shared env cache as UTF-8 with a REPLACEMENT fallback, so a ps1 write for one key destroyed unrelated records that bash's byte-oriented filter left untouched — now a Latin-1 storage projection, bijective over all 256 byte values, with the boundaries enumerated in the source and a writer that refuses rather than corrupts. **D282:** a record write rewrote the whole cache with no check that anything had moved underneath it, across a window holding a git shell-out, so a concurrent claim was lost wholesale; it now compare-and-swaps immediately before the rename and re-applies on retry against the other writer's content rather than replaying its own stale snapshot.

  **The Windows position has improved and is still not parity, and the pin's own release says so.** `Invoke-WebRequest -SkipHttpErrorCheck` is a 7.0+ parameter and it is now gone from both call sites (**D277**), so the `changed_files` PUT can issue on 5.1 and after_goal detection is no longer a silent no-op under its `catch { return }` — the two consequences the `[1.81.0]` entry recorded as open. What has **not** changed is the harder gap beside them: nothing in that repo has ever *executed* the hook under `powershell.exe`, so the fix is verified by reasoning and by pwsh-7 behaviour rather than by a 5.1 run (**D237**). Recorded in those terms because "Windows works now" would be exactly the overstatement the last two entries were careful to avoid.

  Four follow-ups were filed rather than folded in: **D286** (a ps1 snapshot-parity harness), **D288** (whether the record writers should guard against a binary-refusing grep, which this window makes more reachable rather than less), **D289** (five `Write-EnvCache` callers still passing no compare-and-swap), and **D290** (a fail-open catch on the upload-side filter).

- **`stride-security-review` 2.5.1 -> 2.5.2.** `.stride_auth.md` — a live Stride API token, written into whatever repository the agent workflow runs in — is now ignored. Nothing had committed it; the protection was the absence of a badly-timed `git add -A` rather than a rule, which is a poor arrangement in a plugin whose subject is this exact class of mistake. No change to the command, the agent, or the analysis methodology.

- **`stride-ideation` 0.11.0 -> 0.11.1.** The README defined the default `lean` profile as "byte-for-byte equivalent to v0.3.0 behavior" — a pointer into a changelog rather than a description, and one that decays with every release past it. It now states what `lean` runs: the shared core only, the seven gated sections plus the mandatory framing checkpoint, premortem and challenge gate, with no profile-specific forcing questions, optional document sections or reviewer checks. The other three profile rows already described themselves that way. Same credential ignore as above. No behavior change to `/ideate` or `/stridify`.

- **`stride-exploratory-testing` 0.2.0 -> 0.2.1.** The orchestrator skill routed every SFDIPOT request to `heuristics`, which contains none of it — the coverage lens table has been part of `chartering` since it was written. An agent asked to enumerate a product's coverage surface systematically was loading a skill with no answer to give and improvising from the acronym. All three routing sites are repointed, along with the same wrong pointer in the plugin's own README engines table. Cheat sheets and Tours do live in `heuristics` and are still routed there.

This entry does all four steps: four pins bumped, `metadata.version` 1.82.0 -> 1.83.0, all four README rows and all four prose sections synced, and this entry written. Each section still cites its own pin as its highest version, keeping the D189 invariant.

**Tagged `v1.84.0` rather than `v1.83.0`.** The tag series is independent of `metadata.version` and `v1.83.0` was spent on the previous entry, for the reason that entry records. The gap between the two records is now two and will keep widening; `marketplace.json` remains the authority on what is pinned, and nothing installs through a tag.

## [1.82.0] - 2026-08-20

Two pins, one release: the fleet-wide port-canon work landed in `stride` and `stride-lite` on the same day and is recorded here together.

- **`stride` 1.68.0 -> 1.69.0.** The cross-port rules the fleet had only in agreement become a document and a gate. `docs/port-canon.md` states five rules every Stride port is expected to carry — the failed-verdict `note` rule, the decision matrix as the sole decision point for its columns, row precedence within that matrix, the closed `reason_code` skip vocabulary, and a fence-nesting property — each with a per-port `applies_to` row recorded as data rather than prose, so "this port does not owe this rule" becomes a statement tooling can read rather than a judgement call. `scripts/check-port-canon.sh` enforces it across every port and every vendored catalog copy, reporting each rule cell as ok, missing, stale, unexpected, defect or unverifiable. **The distinction the anchor mechanism exists for** is between *this port carries the rule* and *nobody has checked*: compliance is marked by a comment beside the port's own statement of the rule rather than inferred from a keyword search, and the definition site is excluded from its own scan so a port cannot look compliant on the strength of the file defining compliance. Five adversarial rounds preceded the ship, and the recurring finding was one shape — a `find` succeeding had been read as "the tree was examined", so an unreadable file or an unscaffolded port returned quiet instead of a verdict. The checker now hard-halts on a parse it cannot complete, and **exit `2` means no verdict was possible and must be read as red, not as silence.** This release also fixes two D217-shape nested fences that had been silently truncating shipped instructions.

- **`stride-lite` 0.13.0 -> 0.14.0.** `lib/select_workflow_branch.md` gains the two canon rules that apply to it. The first fixes that file as the only place explore, plan and review are gated. The second states what its first-match reading rule rests on, **leading with the precondition** that a task whose `## Key files` heading is missing or unrecognizable resolves to `full` without reaching the table at all — a reader who skipped to the rows would match on complexity alone and land on `skip-all`. The fleet `reason_code` vocabulary is **deferred here rather than adopted**: this plugin emits no `workflow_steps` object and has no completion endpoint, so the rejection that makes the canonical set closed has nothing to run in, and four of the six values name conditions the loop cannot reach. The drift check consequently still reports that one cell missing for this port, and that is the accurate end state rather than an oversight — an anchor beside a deferral would claim a compliance the port does not have.

**The drift check is not green, and that is the intended state.** Its tally across ports and vendored copies is `ok 51, missing 1`, the one miss being the deliberate `stride-lite` deferral above. Neither vendored catalog copy appears in its output, which is the condition the sibling catalogs' sync checklists actually test. Recorded so the next release engineer does not read exit `1` here as a blocked release, or "fix" it by editing an `applies_to` row — the canon's own row for that port is a known open item awaiting a human decision, and editing a gate to make it pass proves nothing.

**Divergence found and corrected, per the standing rule to record rather than silently fix.** The `stride-lite` 0.13.0 pin — tagged `v1.81.0` earlier on 2026-08-19 — synced the *Available Plugins* row and nothing else: the per-plugin prose section's newest clause stayed at `v0.12.0+`, and the `marketplace.json` entry description's newest sentence likewise. This is exactly the failure the "Releases and tagging" note predicts of a release that syncs only the row, and it reopened the D189 backlog for one plugin one release after that backlog was closed. Both surfaces are brought current here, which means the `stride-lite` section gains **two** clauses (`v0.13.0+` and `v0.14.0+`) rather than one. `1.82.0` is not a backfill of `v1.81.0`; that release stands as published, and its omission is recorded here instead.

Both plugin sections now cite their own pin as their highest version again, restoring the D189 invariant. This entry does all four steps: both pins bumped, `metadata.version` 1.81.0 -> 1.82.0, both README rows and both prose sections synced, and this entry written. **Tagged `v1.83.0` rather than `v1.82.0`** — the tag series is independent of `metadata.version`, and `v1.82.0` was spent on the previous entry for the reason that entry records.

## [1.81.0] - 2026-08-19

- **`stride` 1.67.0 → 1.68.0.** Two things dominate. **D280:** `stride-hook.sh` loads the env cache with `. "$ENV_CACHE"` under `set -a`, so every value `stride-hook.ps1` wrote there was shell syntax rather than data — three write sites emitted `KEY=value` bare and a task title of `Fix $(touch /tmp/marker)` was a command substitution bash executed at source time, **confirmed executing rather than theorised**. Every writer now escapes, the loader unquotes with an exact inverse, values are flattened so a newline cannot manufacture a second cache line, and the shell-steering fence became an allow-list after a deny-list was found missing `GIT_SSH_COMMAND` and `GIT_EXTERNAL_DIFF`; five review rounds found the first patch incomplete twice, including a flatten that matched LF only while a lone CR planted `BASH_ENV`. **G413 (W2100–W2106):** `stride-hook.ps1` had no capture step at all, so a native-Windows run produced no snapshot with no error — it now builds the snapshot, carries the five per-task record families, runs the same classification engine as bash, evicts per window so a live outer's anchor survives (D268/D274), and replays the capture-time narrowing verdict on retry (D273). Plugin gate: PowerShell suite 961/961 (from 530), bash suite 787/787 with both bash files byte-identical across the port, the 5.1 static gate clean over `hooks/*.ps1` **and** `scripts/*.ps1`, and all three budgeted hot-path skills within budget.

**Not parity, and the pin's own release says so.** On Windows PowerShell 5.1 `Invoke-WebRequest -SkipHttpErrorCheck` is a 7.0+ parameter at two call sites: the diff cannot upload (recorded `000`, indistinguishable from a refused connection) and after_goal detection is a silent no-op under its `catch { return }` (D277). Nothing in that repo has ever executed the hook under `powershell.exe`, so its new compatibility gate is static only (D237). Recorded here because a catalog entry that reads as "Windows works now" would be the same overstatement the plugin's own release spent a task removing from its docs.

This entry does all four steps, and the records agreed going in: `metadata.version` was at `1.80.0`, and the `stride` pin, the Available Plugins row and the prose section's newest clause were all at `1.67.0`, matching the plugin's last release. No reconciliation was needed. Tagged `v1.82.0` rather than `v1.81.0` — the tag series is independent of `metadata.version` and `v1.81.0` was spent earlier the same day on the `stride-lite` 0.13.0 pin.

## [1.80.0] - 2026-08-15

- **`stride` 1.66.0 → 1.67.0.** Progressive disclosure for the three largest agent bodies — `task-reviewer.md` 66,177 → 55,936, `task-enricher.md` 37,338 → 28,865, `task-decomposer.md` 25,521 → 13,684 bytes, **30,551 bytes** off the per-dispatch cost — with only worked examples, edge-case sections and anti-example galleries moved to sibling `docs/` files, every contract kept inline, and `docs/` chosen over `agents/` because every `.md` there registers as a dispatchable agent. Each split was verified by dispatching the slimmed body against a real fixture rather than by reading the diff, which caught three defects a diff review would not have: a returned-summary rendering that existed only in the example being moved, a waived-matrix-row shape stated only by a sentence narrating that example, and a decomposer carrying no treat-task-text-as-data boundary at all. The decomposer's duplicated request envelope was deleted rather than moved, pointing instead at the `stride-creating-goals` skill that owns it. The release also closes the commit-attribution lineage — per-window purity classification (D244), hook-mediated commit ownership (D255), the purity fixpoint (D256), open-window-aware eviction (D268), the outermost-task gate (D271), and the zero-commit-child ratchet pinned rather than fixed after the candidate branch was measured breaking nine pre-existing pins (D272) — and adds a relatedness gate ahead of the Step 5.5 severity/provenance policy (D270). Plugin gate: hook suite 670/670, PowerShell suite 482/482, all three budgeted hot-path skills within budget.

This entry does all four steps, and the records agreed going in: `metadata.version` was at `1.79.0`, and the `stride` pin and the Available Plugins table row were both at `1.66.0`, matching the plugin's last release. No reconciliation was needed.

## [1.79.0] - 2026-08-14

- **`stride` 1.65.0 → 1.66.0.** The hot-path re-extraction with byte budgets: `stride-workflow/SKILL.md` back to 89,647 bytes and `stride-completing-tasks/SKILL.md` from 100,085 to 56,750, each with cold material moved into gated sibling files (pointers at every original site; every gate, matrix, Decision Summary and prompt-injection framing rule inline; the completion contract re-verified against the server's validation modules with no field requirement, schema shape or enum value changed), plus `scripts/check-skill-budgets.sh` as hook-suite Group 28 — budgets 12-13% above post-extraction sizes, so ordinary edits pass and only sustained regrowth trips. Also: slim-view adoption on the index/tree GETs and the complete acknowledgement, dispatcher mode's projected `fields=status,needs_review` read, and the completed-status confirmation gate in `reference.md`'s dispatcher summaries. Claim and next deliberately stay full.

This entry does all four steps, and the records agreed going in: `metadata.version` was at `1.78.0`, the `stride` pin and the Available Plugins table row were both at `1.65.0`, and `v1.78.0` carries a CHANGELOG entry. No divergence to reconcile.

## [1.78.0] - 2026-08-12

- **`stride` 1.64.0 → 1.65.0.** An optional `workflow_steps[].reason_code` beside the free-text `reason`, so the compliance skip breakdown aggregates instead of producing one row per entry — six values derived by classifying every skipped entry on a real board (73 entries had produced 58 distinct prose strings averaging 145 characters), optional and omittable so agents that predate it complete unchanged on any runtime. Plus write-then-summarise for all three report-producing agents: `task-reviewer`, `task-explorer` and the generic `Plan` dispatch now persist their full output under `.stride/` and return a bounded summary naming the path — 24 lines / 2,000 characters for the reviewer, a deliberately looser 60 lines / 6,000 characters for the explorer and planner, whose summaries are implemented from rather than parsed. A dispatch that supplies no path writes nothing and emits inline, so older orchestrators keep working.

This entry does all four steps, and the records agreed going in: `metadata.version` was at `1.77.0`, the `stride` pin and the Available Plugins table row were both at `1.64.0`, and `v1.77.0` carries a CHANGELOG entry. No divergence to reconcile this time — the `[1.77.0]` entry closed the `v1.76.0` two-of-four gap, and nothing has drifted since.

## [1.77.0] - 2026-08-12

- **`stride` 1.63.0 → 1.64.0.** Task-attributed `changed_files` (a task's snapshot carries the commits that task made, not its nested tasks'), a one-document hook stdout contract so harness-facing fields are actually read, durable per-hook result files, a required substantive note on a `"failed"` section verdict, and hook test suites that calibrate machine load so their failure count no longer depends on what else is running.

### Divergence recorded rather than backfilled

`v1.76.0` was a **two-of-four** release: it bumped `metadata.version` and the `stride` pin in `marketplace.json`, but left the Available Plugins table row at **1.62.0** (one release stale even before this one) and wrote no CHANGELOG entry. Per this catalog's own rule the published tag is left as it stands and the gap is reconciled here — this entry does all four steps, and the table row moves 1.62.0 → 1.64.0 in one jump, skipping 1.63.0, which is why the jump is larger than a single release.

## [1.75.0] - 2026-08-07

Sync the `stride` pin to **1.62.0**.

That release cuts the orchestrator skill's per-task context footprint by **42%** — `skills/stride-workflow/SKILL.md` goes 139,972 → 80,523 bytes — by moving gated and cold content into sibling reference files loaded on demand. Steps 5.5 and 5.6 keep their gate, Decision Summary and an explicit load instruction inline while their bodies move out; every non-Claude-Code branch consolidates into one file loaded at Platform Detection; and six cold lookup sections move to a reference file. Content was **moved, never deleted** — each extraction was proved lossless by reconstructing the pre-change file from its parts and diffing byte-for-byte against git.

**The pin is worth taking for the defect, not the size.** Step 7's worked `reviewer_result` example had fallen behind the contract it illustrated, listing five keys where the server requires eight structured sections on any dispatched review and rejects on them **unconditionally, regardless of the grace flag**. Any agent building its completion payload from that example was getting a hard `422`. Installs on 1.61.0 or earlier are exposed to it.

The release also specifies how Step 5 derives the diff it hands the reviewer — previously left to improvisation, most naturally a bare `git diff` that silently omits staged hunks and untracked files and returns nothing inside a nested subrepo — and adds the data-not-instructions framing and redaction rules that dispatch had been missing.

**Recorded because the catalog should not overstate it:** that release's own goal targeted under 40KB and did not reach it, and says so in its notes. The remaining bulk is hot-path procedure that cannot be extracted without deleting rules.

Per this README's rule that a table cell carries a short clause and the prose carries the detail, the *Available Plugins* row gains a one-clause summary and the `## stride` section gains a full `v1.62.0` callout, so the section's highest cited version again equals its pin.

All four steps done in this one commit: the `stride` pin and description in `marketplace.json`, `metadata.version`, the README row **and** prose, and this entry.

## [1.74.0] - 2026-08-07

Sync the `stride-lite` pin to **0.12.0**.

That release adds three optional gated sub-steps to `stride-lite-workflow`, each dispatching an agent from a **different** plugin — `stride-exploratory-testing:explorer` per charter (Step 6a), `/harden` on confirmed findings (Step 6b), and `stride-security-review:security-reviewer` in considerations mode (Step 6c). All three skip cleanly when that plugin is absent and none can fail a task, so an install without either companion behaves exactly as before. It also ships `SECURITY.md`, which states the trust boundary the plugin had never written down (it executes arbitrary user-authored shell commands from `.stride_lite.md`, unvalidated) and records that the activation marker is a forgeable coordination signal and never an authorization.

Four defects fixed, three of them found by test suites that release introduced. Two had left the native-Windows hook path non-functional: the executor read stdin via `@($input)`, empty for an OS-level pipe, so the documented auto-fire had never worked and exited 0 without reporting it (D215); and `Start-Process -ArgumentList` re-split every command, so `bash` received only its first token (D218). The other two are workflow-integrity fixes — a bullet or numbered `## Key files` list counted zero and sent a multi-file task to `skip-all` with no review (D216), and a mispaired code fence rendered part of an agent contract as sample output (D217).

Per this README's rule that a table cell carries a short clause and the prose carries the detail, the *Available Plugins* row gains a one-clause summary and the `## stride-lite` section gains a full `v0.12.0+` callout, so the section's highest cited version again equals its pin.

All four steps done in this one commit: the `stride-lite` pin and description in `marketplace.json`, `metadata.version`, the README row **and** prose, and this entry.

## [1.73.0] - 2026-07-31

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` pin from `1.53.0` to **`1.61.0`** and extended the entry's description with a `New in v1.61.0:` clause. `1.54.0`–`1.61.0` are a consistency sweep over the plugin's two optional gated integrations, each item found by an exploratory session run against the integration itself and filed as its own defect (D201–D208). **Reachability is the theme:** neither Step 5.5's gate (`manual_tests` + plugin) nor the deep security sub-step's (`security_considerations` + plugin) has a review precondition, so a small 0-1 `key_files` task reaches both — but nine routing artifacts across the three skills said otherwise, including the deep sub-step's own position as a level-4 child of the reviewer-dispatch section, a subtree a small task never enters. All nine now route through both, the sub-step fixed by a pointer rather than a promotion so its three positional cross-file citations keep resolving. The completion gate's verdict-present checks are **scoped to payloads where a structured review block was actually parsed**, so a self-reported skip or an unparseable review no longer demands a re-review the decision matrix forbids or a verdict that would be fabricated. A **blocked** exploratory session is recorded as *not performed* rather than counted as coverage, with its obstacle recorded as an obstacle instead of a severity-bearing finding. Both optional dispatches now state where their wall-clock goes in `workflow_steps` — folded into the existing `reviewer` entry, never a seventh name, and recorded in `completion_notes` where no reviewer ran. And the exploratory gate's surface list and never-execute-untrusted-plugin-content prohibition are stated identically in both orchestrator skills, so the security gate's cross-reference to "the same detection" resolves to a gate that contains the rule. Marketplace `metadata.version` bumped to `1.73.0` — see the divergence note below for why it was at `1.71.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table from `1.47.0` to **`1.61.0`** with a short `v1.48.0–v1.61.0` clause, and added two narrative paragraphs to the `## stride` prose section: one for `v1.54.0–v1.61.0` and one for `v1.48.0–v1.53.0`, the latter covering the exploratory-testing second wave (severity mapping and Critical escalation, non-interactive dispatch surfaces, explicit session budget, harmed-party summaries, pre-session gitignore guidance, and the optional Step 5.6 `/harden`). The section now cites its pin as its highest version again.

### Divergence recorded, not silently corrected

Per this catalog's own rule, the state `1.73.0` reconciles is written down rather than quietly fixed:

- **`v1.72.0` was a one-of-four release.** Its commit (`867adab`, "Sync the stride pin to 1.53.0 (G392)") touched `marketplace.json` alone — it bumped the `stride` pin and its description, but left `metadata.version` at `1.71.0`, left the README table row at `1.47.0` and the prose at `v1.47.0+`, and wrote no changelog entry. It is tagged and published on GitHub, so it is a real release; it is simply an incomplete one, of exactly the shape the README's four-step rule calls "a half-finished release, not a release."
- **It reopened the prose-sync gap D189 closed.** The README's own known-gaps note warns that "a release that syncs only the row reopens this gap" — `v1.72.0` synced neither the row nor the prose, so both had drifted two pin bumps behind. `1.73.0` closes it again by syncing the row, the prose and `metadata.version` together.
- **`1.72.0` is not backfilled and not renumbered.** The release exists on GitHub against a state users could resolve through, so manufacturing a retroactive entry for it would misrepresent the record — the same reasoning `[1.66.0]` and `[1.67.0]` applied to `v1.64.0` and `v1.65.0`. The skipped `metadata.version` value of `1.72.0` is therefore never occupied in `marketplace.json`; this entry is where that is recorded.

## [1.71.0] - 2026-07-30

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride-exploratory-testing` pin from `0.1.0` to **`0.2.0`**, its first release since the initial publication, and rewrote the entry's surface description (five skills / five commands → six skills / seven commands). `0.2.0` is a capability sweep from the plugin's own 2026-07 improvement research, delivered as six tasks: a sixth skill **`bug-advocacy`** encoding Kaner's RIMGEA plus the **severity** definition the explorer's contract always required and the plugin never had — a four-level `Critical > High > Moderate > Minor` rubric built as an impact ladder of explicit clauses with aggravating-only modifiers, so two sessions rating the same bug from the same evidence land on the same level; **`/pair`**, the plugin's first interactive mode and the inversion of `/explore`, where the human drives the application and Claude suggests the next probe, names the `heuristics` lens that produced it, and sweeps three ledgers to volunteer what the session has neglected — its division of labour enforced by a tool allowlist that holds nothing capable of reaching the app; **`/harden`**, the first path from *Explored* back to *Checked*, which detects the project's own test framework rather than assuming one, drafts a regression check per confirmed bug from its `minimal_repro` (RIMGEA's Isolate step already produces exactly what a minimal test case needs), reports what it could not convert under seven named categories rather than guessing, stages drafts under `.exploratory/checks/` instead of writing into the user's suite, and never claims a draft passes because it holds no test runner; **`.exploratory/` session artifacts** — per-run debriefs and sheets, an append-only charter backlog, and a coverage map that deliberately carries no percentage — which `/charter` reads back so charter generation stops re-proposing covered ground; and `SFDPOT` corrected to **`SFDIPOT`** with the missing Interfaces lens. **Breaking for downstream consumers of the `explorer` agent's JSON:** `session_sheet` drops `duration` and `tbs` (wall-clock measures an agent cannot honestly take, whose presence contradicted the agent's own hard rule against fabricating a result) in favour of counts plus an agent-native probe budget, `bugs[]` gains the four RIMGEA products, and `charter-generator` gains an optional `deprioritized` key and accepts `interfaces` as a `lens` value. Marketplace `metadata.version` bumped from `1.70.0` to `1.71.0`.
- **`README.md`** — Updated the `stride-exploratory-testing` row in the `Available Plugins` table to version `0.2.0` with the new surface counts and a `v0.2.0+` clause, corrected the `## stride-exploratory-testing` prose section's Surface bullets (six skills, seven slash commands), and added a `v0.2.0+` narrative paragraph so the section cites its pin as its highest version — closing the "carries no version narrative yet" gap the release-record note flagged against this plugin.

## [1.70.0] - 2026-07-29

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.44.0` to **`1.47.0`** so `/plugin update stride@stride-marketplace` pulls the three releases cut since the last sync. `1.45.0` stops the `task-reviewer` worked example echoing a matrix row `passing` for a behaviour the same review proves defective — the Lifecycle / wiring row claimed the move broadcasts exactly once while three other legs of the same example state it broadcasts twice, so under the file's own Mismatch definition the row was a Mismatch echoed `passing`, inside an example prefaced "Mimic this shape exactly" (D198). `1.46.0` moves the three-value to two-value severity collapse out of worked-example commentary and into the two normative loci — review step 1 and the `acceptance_criteria` hard rule — so a reviewer working the numbered steps in order no longer meets a flat "If any criterion is Not Met, flag it as a Critical issue" ~90 lines before the reconciliation that qualifies it, and the hard rule finally covers the wholly-absent case and its `critical` severity (D199). `1.47.0` splits the completion gate's ~826-word bidirectional checkbox into a ~201-word scannable check plus a named, explicitly-binding sub-block, moving every clause **verbatim** — the move was proven character-for-character rather than asserted, with zero characters lost (D200). Marketplace `metadata.version` bumped from `1.69.0` to `1.70.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table to version `1.47.0` with a short three-clause addition, and added `v1.45.0+`, `v1.46.0+` and `v1.47.0+` narrative clauses to the `## stride` prose section so it again cites its pin as its highest version.

### Note on the catalog-only releases in this window

Four commits in this cycle (**D194**–**D197**) changed this README alone and carry **no** `metadata.version` bump and no pin change, so they ship under the `1.69.0` number rather than taking one of their own. They correct the catalog's own accuracy rather than what it distributes: the intro sentence said "Five plugins" against a six-entry manifest and omitted `stride-exploratory-testing` entirely (D194); the `## stride` **Agents:** list named four agents where `stride/agents/` ships five, with `task-enricher` missing even though the same section referred to it twice in prose (D195); the *Automatic Hook Execution* event table had been frozen at four rows since `v1.5.0` and omitted `after_goal`, contradicting three other statements in the same section (D196); and the *Known gaps* tag figure read "32 of 63 tags" where the repo measures 64, with "Most" overstating what is at most half (D197). Each was rewritten count-free where a count was the thing that rotted, per this section's own preference.

### Backward compatibility

Pin-and-documentation changes only. No `.stride.md`, `.stride_auth.md`, hook, wire-shape, or reviewer-schema change — `schema_version` stays `1.6` across all three `stride` releases, and no plugin pin other than `stride` is touched.

## [1.69.0] - 2026-07-29

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.43.0` to **`1.44.0`** so `/plugin update stride@stride-marketplace` pulls the new release. v1.44.0 resolves two one-directional rule gaps in the reviewer schema of record. A reviewer finding a genuine vulnerability on a task that supplied no `security_considerations` was caught between the empty-section rule (emit `not_assessed`) and the Consistency rule (emit `failed`), with no documented winner — and the cheapest escape from an unresolvable constraint is to suppress the finding. The four-tile verdict rule now states that a real finding outranks `not_assessed` in all four sections, names suppression and re-labelling as the worse defect, and reframes the v1.41.0 credential carve-out as the worked instance of that general rule rather than the sole exception. Review step 5 now assesses its dimensions whether or not the task listed considerations. Both nested-array escalation rules gain a reverse direction, deliberately opposite: `failed` beside an all-`mitigated` `considerations[]` is legitimate (the failure can originate outside the task's list), while `failed` beside no `failing` matrix row is a defect (`rows[]` enumerates everything that verdict can be about). Marketplace `metadata.version` bumped from `1.68.0` to `1.69.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table to version `1.44.0` with a short `v1.44.0+` clause, and added the matching `v1.44.0+` clause to the `## stride` prose section, keeping both surfaces in sync per the four-step rule.

### Backward compatibility

Prompt-and-documentation changes only. No `.stride.md`, `.stride_auth.md`, hook, wire-shape, or reviewer-schema change — `schema_version` stays `1.6`, and neither the three-state section-status enum nor the seven-value `issues[].category` enum changes. No plugin pin other than `stride` is touched.

## [1.68.0] - 2026-07-29

### Updated

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.42.0` to **`1.43.0`** so `/plugin update stride@stride-marketplace` pulls the new release. v1.43.0 gives the `task-reviewer` worked example's `not_met` acceptance criterion a backing issue. The criterion's `evidence` had ended "…see the critical issue above" while no such issue existed, so the cross-reference dangled and the example demonstrated the opposite of what review step 1 and the `acceptance_criteria` hard rule both require; `acceptance_criteria` was another category mandated in prose and demonstrated in no worked instance, the same defect class `1.42.0` closed for `security`. It is not the last one — `pattern` remains undemonstrated and is recorded in the plugin's own entry as deliberately deferred. The example now carries an `important`-severity `category: "acceptance_criteria"` entry over a synthetic double broadcast, with `issue_counts`, the `summary`, and the criterion's `evidence` moved to match, and the preamble's two enumerations reconciled — the criterion had been listed as a sixth item outside the five-issue enumeration. The preamble also resolves the Critical-vs-Important question the two rules left open, by documenting that review step 1's three-value working scale (Met / Partially Met / Not Met) collapses into the two-value emitted `status` enum, so a partially-satisfied criterion is emitted `"not_met"` while keeping `important` severity. Marketplace `metadata.version` bumped from `1.67.0` to `1.68.0`.
- **`README.md`** — Updated the `stride` row in the `Available Plugins` table to version `1.43.0` with a short `v1.43.0+` clause, and added the matching `v1.43.0+` clause to the `## stride` prose section, keeping both surfaces in sync per the four-step rule.

### Backward compatibility

Prompt-and-documentation changes only. No `.stride.md`, `.stride_auth.md`, hook, wire-shape, or reviewer-schema change — `schema_version` stays `1.6`, and the seven-value `issues[].category` enum the example draws from was already legal. No plugin pin other than `stride` is touched.

### Note on the prose surface (D189)

The per-plugin prose backlog recorded as a known gap in `[1.67.0]`'s fifth bullet was closed by **D189**, immediately before this release: `## stride`, `## stride-ideation`, `## stride-lite` and `## stride-security-review` were all brought up to their pins, the stale "64 fixtures" count was replaced with count-free phrasing, and the README's `Known gaps` block was rewritten to describe the resulting state. D189 kept the two-surface convention rather than declaring one canonical, and recorded that decision plus the remaining problem — the `stride` table cell is a single line of roughly 14 KB — as its own gap. This release is the first to follow the resulting guidance: the table-cell addition above is deliberately short, with the detail carried in the prose section.

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
