# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] — 2026-06-12

Initial release as a standalone skill repository.

### Added

- `skills/citation-sync/SKILL.md` — orchestrator for auditing and syncing the four citation layers of a research repository (in-text docs → `.zenodo.json` references → `graph.jsonld` ExternalReference → Wikidata P2860), bottom-up, with curation criteria and per-layer deferral to release-doi / jsonld-knowledge-graph / wikidata-federation.
- `skills/citation-sync/scripts/citation_audit.py` — read-only Phase 0 audit across one or more repos (exit 0 converged / 1 divergence / 2 fatal, `--skip-wikidata`, `--json`).
- `skills/citation-sync/evals/` — eval definitions.
- `scripts/sync-from-local.sh` — one-way export from the live Claude Code harness; the harness copy is canonical, this repository is the publication mirror.
