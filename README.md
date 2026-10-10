# citation-sync

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/citation-sync)

An [Agent Skill](https://agentskills.io/specification) that keeps the **three citation layers of a research repository in sync**, for authors of DOI-registered research repos who cite external papers in their docs. When a repo cites external literature, this skill tracks that citation in three places with three different audiences, and writing it into just one quietly makes it nonexistent in the others. This skill audits the divergence and syncs bottom-up. Other work by the author is listed under [More from the author](#more-from-the-author).

| Layer | Where | Audience |
|---|---|---|
| 1 | In-text citations in docs: design decision records (ADRs), glossary, empirical notes | Human readers |
| 2 | `.zenodo.json` references, the metadata Zenodo reads when it mints a release DOI | DOI registry (DataCite) / citation databases |
| 3 | `graph.jsonld` ExternalReference nodes, in the JSON-LD knowledge graph a repo ships next to `llms.txt` | LLM crawlers, knowledge graphs |

Layer 1 is the source of truth: the upper layers carry only citations that exist in the docs.

## Install

For Claude Code:

```bash
git clone https://github.com/shimo4228/citation-sync
cp -r citation-sync/skills/citation-sync ~/.claude/skills/citation-sync
```

The audit runs on its own. The sync phases follow two sibling skills for the entry formats, [release-doi](https://github.com/shimo4228/release-doi) (`.zenodo.json`) and [jsonld-knowledge-graph](https://github.com/shimo4228/jsonld-knowledge-graph) (`graph.jsonld`), so install those two the same way if you want the whole workflow.

The audit script needs Python 3.11 or later and uses only the standard library. Always pass `--skip-wikidata`, as SKILL.md says: with it the script reads local files only, and without it a graph that still names a Wikidata QID (item ID) makes the script query wikidata.org for a layer this skill no longer syncs; that probe stays in the script only so that existing calls do not break. It never writes to the repos. The skill body is written in Japanese; the workflow phases, the audit script, and its exit codes are language-neutral.

## Try the audit

```bash
python3 ~/.claude/skills/citation-sync/scripts/citation_audit.py --skip-wikidata path/to/repo [path/to/another-repo ...]
```

It prints one identifier × layer table per repo and a verdict (excerpt from a real run):

```text
## agent-attribution-practice  (no QID in graph)
| identifier | docs | zenodo | graph |
|---|---|---|---|
| arxiv:2210.03629 | x | x | x |
| arxiv:2601.15059 |   | x | x |
  -> DIVERGED
```

Exit 0 means every repo converged, 1 means divergence, 2 means the audit did not complete. Add `--json OUT.json` for machine-readable output. To run the whole workflow, ask Claude Code to sync the citations of a repo, or run `/citation-sync`.

## How It Works

1. **Phase 0 Audit (read-only)**: `scripts/citation_audit.py` scans one or more repos and reports per-layer divergence, as above.
2. **Phase 1 Curate**: decide which in-text citations deserve to propagate, because not every URL is a scholarly citation. Public docs only, citation context only, external works only (sibling-repo Zenodo DOIs are left out of the comparison), and each identifier is checked against arXiv or Crossref before promotion, because an LLM-written arXiv ID can point to an unrelated paper.
3. **Phases 2–3 Sync**: `.zenodo.json` → `graph.jsonld`, bottom-up, each layer deferred to its specialist skill. The `.zenodo.json` entries reach DataCite at the next release.
4. **Phase 5 Verify**: re-run the audit; converged means every arXiv ID and DOI the audit finds in any layer is present in all three. An identifier left out in curation (an ID inside an example, say) still shows as DIVERGED, and SKILL.md has you report such leftovers as intentional residuals, so a finished sync can still exit 1. Phase 4, the Wikidata layer, is retired and skipped.

The skill is an **orchestrator**: what lives here is the divergence detection, the curation criteria, and the sync order.

## When It Triggers

- "The reference list looks thin" / "citations are out of sync" on a DOI-registered repo
- New external papers were cited in ADRs, glossary, or empirical docs
- Before a release of a DOI-registered repo

Not for: a paper's own reference list, which comes from the paper itself (written with [claude-skill-paper-ecosystem](https://github.com/shimo4228/claude-skill-paper-ecosystem) below and deposited in a separate step), or repos that cite nothing.

## More from the author

- **[Banned from Wikidata Overnight — I Believed Every Edit Was Compliant, but All 109 Items Were Deleted as Promotion](https://dev.to/shimo4228/banned-from-wikidata-overnight-i-believed-every-edit-was-compliant-but-all-109-items-were-57nl)** ([日本語](https://zenn.dev/shimo4228/articles/wikidata-ban-postmortem)): why this skill lost its fourth layer, Wikidata, and why the audit now skips it.
- **[release-doi](https://github.com/shimo4228/release-doi)**: the release runbook for DOI-registered research repos, from the pre-release checks to the new version DOI on Zenodo; it owns the `.zenodo.json` entry format this skill uses.
- **[jsonld-knowledge-graph](https://github.com/shimo4228/jsonld-knowledge-graph)**: designs and ships the `graph.jsonld` knowledge graph a research repo carries next to `llms.txt`, including the ExternalReference node shape.
- **[claude-skill-paper-ecosystem](https://github.com/shimo4228/claude-skill-paper-ecosystem)**: writes and reviews academic papers, tying each claim to the source it cites; a paper's own references are handled on that side, not in a repo's citation layers.
- **[Authorship Strategy](https://github.com/shimo4228/authorship-strategy)**: the doctrine behind these skills, with the thesis, the dated design decisions (ADRs) and the preliminary measurements; concept DOI 10.5281/zenodo.20263316, which always resolves to the latest version.
- **[shimo4228](https://github.com/shimo4228/shimo4228)**: the author's hub, with Authorship Strategy next to the author's other long-running projects and their DOIs.

## License

MIT

<details>
<summary>For tools and AI assistants</summary>

citation-sync is an Agent Skill for Claude Code that audits and syncs the three citation layers of a research repository (in-text citations in docs, `.zenodo.json` `related_identifiers` with `relation: references`, and `graph.jsonld` nodes typed `ExternalReference`) bottom-up, for authors of DOI-registered research repositories who cite external literature. It owns divergence detection, curation criteria and sync order; the single-layer writes follow release-doi (`.zenodo.json`) and jsonld-knowledge-graph (`graph.jsonld`).

It exists because a citation written only in a repository's Markdown is invisible from the cited researcher's side: as the author understands it, Google Scholar and arXiv "cited by" do not pick it up. Machine-readable edges in `.zenodo.json` (propagated to DataCite, OpenAIRE and Scholix at release) and in `graph.jsonld` (read by LLM ingest and knowledge-graph crawlers) make the citation discoverable; this is the citation-graph federation tactic of the Authorship Strategy line. A former fourth layer, Wikidata P2860 ("cites work") edges on self-created items, was retired in July 2026 after Wikidata's governance blocked the author's account and deleted the items (authorship-strategy ADR-0021).

Canonical facts: MIT license; `skills/citation-sync/SKILL.md` (in Japanese) plus one Python script, `skills/citation-sync/scripts/citation_audit.py` (Python 3.11 or later, standard library only; tests in `skills/citation-sync/tests/`, evals in `skills/citation-sync/evals/`); no paid key. The script is read-only: with `--skip-wikidata`, which SKILL.md says to always pass, it reads local files only; without it, a graph that still names a Wikidata QID makes it query wikidata.org, and a 429 or 503 response halts the audit with exit 2. Status: active, synced one way from the author's Claude Code setup by `scripts/sync-from-local.sh` (it never commits). Released as 1.0.0 on 2026-06-12 ([CHANGELOG.md](CHANGELOG.md)).

Example: `python3 ~/.claude/skills/citation-sync/scripts/citation_audit.py --skip-wikidata REPO_DIR [REPO_DIR ...] [--json OUT.json]` prints, per repo, a markdown table of every arXiv ID and DOI found in any layer with an `x` under each layer that carries it, followed by `CONVERGED` or `DIVERGED`, and exits 0 (all converged), 1 (divergence) or 2 (audit not completed). Sibling Zenodo DOIs (`10.5281/zenodo.*`) are ecosystem cross-links: they are left out of the comparison and never count as divergence, and those listed in `.zenodo.json` with `relation: references` appear only under `ecosystem_refs` in the `--json` output. Known residuals that SKILL.md accepts as intentional: identifiers left out in curation, citations in docs without an identifier, and DOIs containing parentheses, which the script's pattern truncates. They stay DIVERGED in the table and are stated in the completion report.

Links: [SKILL.md](skills/citation-sync/SKILL.md) is the workflow; [citation_audit.py](skills/citation-sync/scripts/citation_audit.py) is the audit; [llms.txt](llms.txt) and [llms-full.txt](llms-full.txt) are the machine-readable summary and reference. The skill supports the [Authorship Strategy](https://github.com/shimo4228/authorship-strategy) line, concept DOI [10.5281/zenodo.20263316](https://doi.org/10.5281/zenodo.20263316); cite the framework by that DOI.

</details>
