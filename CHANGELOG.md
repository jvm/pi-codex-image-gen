# Changelog

All notable changes to this project will be documented in this file.

This project follows the spirit of [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and uses semantic versioning for releases.

## [Unreleased]

## [0.1.4] - 2026-05-07

### Fixed

- Added `repository.url` to `package.json`, required by npm provenance (sigstore) verification.

## [0.1.3] - 2026-05-07

### Changed

- Migrated peer dependencies from `@mariozechner` to `@earendil-works` scope (`@earendil-works/pi-ai`, `@earendil-works/pi-coding-agent` v0.74.0).

### Added

- `.github/workflows/publish.yml`: OIDC-based npm publish triggered on GitHub release.
- `scripts.check` and `scripts.pack:dry-run` in `package.json`.

## [0.1.2] - 2025

### Changed

- Moved extension entry point to `index.ts` at the package root for a cleaner npm package name display in Pi.

## [0.1.1] - 2025

### Fixed

- Fixed extension identifier to use the package name instead of `package:file`.

## [0.1.0] - 2025

### Added

- Initial release.
- `codex_generate_image` Pi extension using the OpenAI Codex ChatGPT backend (`gpt-image-2`). Piggybacks on Pi's `openai-codex` login — no `OPENAI_API_KEY` required.
- `skills/imagegen` Pi skill with prompting playbook, chroma-key transparency workflow, and Python CLI fallback (`image_gen.py`).
- Save-mode configuration (`none`, `project`, `global`, `custom`) with env and config-file overrides.
- SSE stream parsing with exponential backoff retry on transient errors.
