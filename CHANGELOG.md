# Changelog

All notable changes to the Stride marketplace pin set will be documented in this file.

## [1.27.1] - 2026-05-20

### Changed

- **`.claude-plugin/marketplace.json`** — Bumped the `stride` plugin pin from `1.14.0` to `1.14.1` so `/plugin update stride@stride-marketplace` picks up the W748 documentation surfacing edits (canonical API Request Format example body now includes `changed_files`; pre-completion verification checklist now contains the "Did you embed `.stride-changed-files.json` into the payload as `changed_files`?" item). Marketplace `metadata.version` patch-bumped from `1.27.0` to `1.27.1` to match. Stride plugin `description` field extended with a `v1.14.1+:` clause noting the surfacing edits; v1.14.0+ feature description preserved.

### Source

Stride plugin release: https://github.com/cheezy/stride/releases/tag/v1.14.1 — patch release for W748 (SKILL.md surfacing edits). No functional change in the plugin; pure documentation update so installed plugins prompt for `changed_files` during normal payload assembly.
