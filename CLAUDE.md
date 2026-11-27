# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A documentation-only catalog of distributed-systems failure modes. There is no code, no build system, no test suite, and no package manager. All content lives in two Markdown files:

- `observations.md` — the catalog itself: 475 entries in 21 categories (375 on classic distributed systems, 100 on LLM-based agent systems).
- `README.md` — purpose, per-category entry counts, entry format, and contribution rules.

## Catalog structure

- Entries are **numbered globally and continuously across categories** (entry 72 is the last Network entry, entry 73 is the first Consensus entry). Numbers do not restart per category.
- Each category is an `## <Name>` heading followed by numbered-list entries. An entry is a `N. <failure mode title>` line plus a paragraph indented by three spaces on the next line.
- The last two categories, **General Software Defects** and **Operational Process Failures**, are deliberately quarantined: they hold failures that are not specific to distributed systems. Keep them out of the core categories.
- The four **AI Agent** categories (Orchestration, Context & Memory, Reliability, Security) sit after `Operational` and before the quarantined categories. An entry there must be specific to agents, such as context limits, non-determinism, tool use, or inter-agent communication.

## Editing rules

- **Adding, removing, or reordering an entry renumbers every later entry** in `observations.md`.
- **Counts must stay in sync**: the per-category table in `README.md` and the stated totals and category counts in both files must match the actual contents after any change.
- **Entry format is fixed**, in this order: mechanism (what happens and why), impact (what breaks or what is observed), detection and mitigation. One paragraph per entry, written as prose, not bullet points.
- **New entries must be specific and plausible.** Reject tautologies (cause equals effect), vague catch-alls, and bugs that would fail in a single process just as readily as in a distributed system.
- The README shows its format example as a blockquote (`>`); the real file uses indented list items. Follow the real file, not the example's quoting.

## Verifying changes

With no automated checks, verification is manual. After an edit, confirm:

1. Entry numbers are sequential with no gaps or duplicates, across category boundaries.
2. Every entry has its indented paragraph directly beneath its title line.
3. The README table sums to the stated total and matches the number of entries per category heading.

A quick way to check numbering without reading the file is to list the entry title lines, for example `grep -nE '^[0-9]+\. ' observations.md`.
