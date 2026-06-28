# citation-sync

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/citation-sync)

An [Agent Skill](https://agentskills.io/specification) that keeps the **four citation layers of a research repository in sync**. When a repo cites external literature, that citation surfaces in four places with four different audiences — and writing it into just one quietly makes it nonexistent in the other three. This skill audits the divergence and syncs bottom-up.

| Layer | Where | Audience |
|---|---|---|
| 1 | In-text citations in docs (ADRs, glossary, empirical notes) | Human readers |
| 2 | `.zenodo.json` references | DOI registry / citation databases |
| 3 | `graph.jsonld` ExternalReference nodes | LLM crawlers, knowledge graphs |
| 4 | Wikidata P2860 (cites work) | The cited researchers' side — discoverability |

## Install

### Claude Code

```bash
# Copy into your global skills directory
cp -r skills/citation-sync ~/.claude/skills/citation-sync
```

### SkillsMP

```bash
/skills add shimo4228/citation-sync
```

## How It Works

1. **Phase 0 Audit (read-only)** — `scripts/citation_audit.py` scans one or more repos and reports per-layer divergence (exit 0 = converged, 1 = divergence, 2 = fatal); `--skip-wikidata` for offline runs, `--json` for machine-readable output
2. **Phase 1 Curate** — the judgment layer: which in-text citations deserve to propagate (not every URL is a scholarly citation)
3. **Phases 2–4 Sync** — `.zenodo.json` → `graph.jsonld` → Wikidata P2860, bottom-up, each layer deferred to its specialist skill
4. **Phase 5 Verify** — re-run the audit; converged means every curated citation exists in all four layers

The skill is an **orchestrator**: single-layer implementations are deferred to [release-doi](https://github.com/shimo4228/release-doi) (`.zenodo.json`), [jsonld-knowledge-graph](https://github.com/shimo4228/jsonld-knowledge-graph) (graph), and [wikidata-federation](https://github.com/shimo4228/wikidata-federation) (Wikidata). What lives here is the divergence detection, the curation criteria, and the sync order.

## When It Triggers

- "The reference list looks thin" / "citations are out of sync" on a DOI-registered repo
- New external papers were cited in ADRs, glossary, or empirical docs
- Before a release of a DOI-registered repo
- A repo's Wikidata item shows fewer cited works than its docs

Not for: a paper's own reference list (that is paper-deposit / wikidata-federation territory), or repos that cite nothing.

## Syncing from the harness

The canonical copy of this skill lives in the author's live Claude Code harness. This repository is a one-way publication mirror:

```bash
scripts/sync-from-local.sh --dry-run   # report differences only
scripts/sync-from-local.sh             # apply to working tree (never commits)
```

## About this skill

This skill supports the citation-graph federation tactic of the author's research program: external literature cited in a repo should be discoverable *from the cited researcher's side*, which requires machine-readable citation edges (Zenodo references, Wikidata P2860), not just in-repo Markdown. It is maintained by [@shimo4228](https://github.com/shimo4228), whose research lines are the [Agent Knowledge Cycle (AKC)](https://github.com/shimo4228/agent-knowledge-cycle) ([DOI 10.5281/zenodo.19200726](https://doi.org/10.5281/zenodo.19200726)), [Contemplative Agent](https://github.com/shimo4228/contemplative-agent) ([DOI 10.5281/zenodo.19212118](https://doi.org/10.5281/zenodo.19212118)), and [Agent Attribution Practice (AAP)](https://github.com/shimo4228/agent-attribution-practice) ([DOI 10.5281/zenodo.19652013](https://doi.org/10.5281/zenodo.19652013)).

The skill body is written in Japanese; the workflow phases, the audit script, and its exit codes are language-neutral.

## License

MIT
