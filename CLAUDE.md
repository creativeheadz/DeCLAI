# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

DeCLAI is a **documentation-only** repository: an academic design proposal for a decentralised GPU-sharing credit system for LLM inference. There is no source code, no build system, no tests, and no package manifest — only Markdown sections, a Bibliography, a README, REVIEW.md (adversarial peer-review findings), KNOWN_LIMITATIONS.md, and a LICENSE. Any "code" present is illustrative snippets embedded inside section Markdown.

Author: Andrei Trimbitas (a.trimbitas@oldforge.tech). Originally published on ResearchGate (DOI 10.13140/RG.2.2.21788.19848); the version in this repository is the 2026-05 revision after an adversarial peer-review pass.

## Recent revision (2026-05)

Substantial revision in May 2026 in response to an adversarial peer review. Key changes:

- §9 retitled from "Experimental Results" to "Projected Performance and Planned Evaluation Protocol"; fabricated pilot deployments removed.
- Fabricated bibliography entries [10], [11], [12] withdrawn; §1.3 rewritten as illustrative scenarios.
- Theorem 4.1 demoted to Property 4.1 (informal); Theorem 7.1 demoted to Conjecture 7.1; Theorem 7.2 restated as Definition 7.1.
- §6 threat model unified to f < n/3 Byzantine; zk-SNARK / MPC / FE claims softened with honest cost callouts.
- §7 carbon math corrected (was ~20× off in per-participant power); economics updated to reflect reputation multipliers and decay.
- Prior art (Petals, Hivemind, SWARM, DiLoCo, Bittensor, Gensyn, Akash) added to §2 and Bibliography ([211]–[224]).
- README headline impact percentages withdrawn or hedged.

See `REVIEW.md` for the full adversarial review findings and `KNOWN_LIMITATIONS.md` for the audit trail.

## Structure and naming gotcha

The paper is split into 10 section files plus `Bibliography.md`. Filename casing is **inconsistent** — sections 1–6 and 9 use `Section_N_...md` (capital S, capital words), while sections 7, 8, and 10 use `section_N_...md` (lowercase). When editing or cross-linking, copy the exact filename rather than guessing; the README's links rely on the existing casing.

Section topics, in order: Introduction → Related Work → System Architecture → Credit Mechanism Design → Distributed Inference Protocol → Security and Privacy → Economic Analysis → Implementation Details → Experimental Results → Use Cases and Social Impact.

Note: the README's "For Researchers" section links to `paper/DeCLAI_Full_Paper.md`, which does **not** exist in the repo — there is no `paper/` directory. Treat that link as aspirational unless creating the combined file is part of the task.

## Citation conventions

References are IEEE-format, numbered, and centralised in `Bibliography.md` (~220 entries after the 2026-05 prior-art additions). In-text citations use bracketed numerals like `[101][102]` — multiple citations are concatenated without separators (no spaces, no commas). When adding a new reference:

1. Append it to `Bibliography.md` with the next sequential number — do not renumber existing entries, since every section's in-text citations are positional. The bibliography has a "Hygiene Notes" appendix listing known duplicates and stale entries that should *not* be renumbered.
2. Insert the `[N]` marker into the prose at the point being cited.

**Withdrawn entries:** [10], [11], [12] are *withdrawn* (see the bibliography). Do not cite these. The withdrawal note is preserved in place to keep downstream numbering stable.

## Editorial tone

The prose is academic register: full sentences, hedged claims, no emoji inside section bodies (emoji appear only in the README, sparingly). Quantitative claims (percentages, dollar figures) are load-bearing — if you change one, check whether it's repeated elsewhere. The 2026-05 revision either (a) shows the derivation with sensitivity, (b) uses qualitative language ("substantially", "order-of-magnitude"), or (c) labels the figure "(projected; not measured)". Avoid reintroducing unhedged percentages — those were the single biggest reviewer complaint.

Code blocks inside sections are **illustrative pseudocode**, not real implementations — Rust/Python/Go snippets exist to demonstrate API shape and design intent. Don't treat them as compilable artifacts or try to "fix" them against a real toolchain.

## Working in this repo

There are no build/lint/test commands. Useful operations are limited to:

- Reading and editing the 10 section Markdown files and `Bibliography.md`.
- Cross-checking citation numbers between section prose and `Bibliography.md`.
- Verifying internal section-to-section references (most are by title, not by anchor link).
