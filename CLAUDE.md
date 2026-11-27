# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A documentation-only catalog of distributed-systems failure modes. There is no code, no build system, no test suite, and no package manager. All content lives in an index, a directory of category files, and a README:

- `observations.md` — the index: a table of category, file, entry count, and number range. The catalog holds 467 entries in 21 categories (367 on classic distributed systems, 100 on LLM-based agent systems).
- `observations/` — one Markdown file per category, named after the category in lowercase with hyphens, for example `observations/ai-agent-security.md`.
- `README.md` — purpose, per-category entry counts, entry format, and contribution rules.

## Catalog structure

- Entries are **numbered globally and continuously across categories** (the last entry of `observations/network.md` is followed directly by the first entry of `observations/consensus.md`). Numbers do not restart per category.
- Each category file opens with an `## <Name>` heading followed by numbered-list entries. An entry is a `N. <failure mode title>` line plus a paragraph indented by three spaces on the next line.
- The last two categories, **General Software Defects** and **Operational Process Failures**, are deliberately quarantined: they hold failures that are not specific to distributed systems. Keep them out of the core categories.
- The four **AI Agent** categories (Orchestration, Context & Memory, Reliability, Security) sit after `Operational` and before the quarantined categories. An entry there must be specific to agents, such as context limits, non-determinism, tool use, or inter-agent communication.

## Editing rules

- **Adding, removing, or reordering an entry renumbers every later entry**, including the entries in later category files. Renumber with a script, not by hand.
- **Counts must stay in sync**: the per-category table in `README.md`, the table in the index `observations.md`, and the stated totals and category counts in both files must match the actual contents after any change.
- **Entry format is fixed**, in this order: mechanism (what happens and why), impact (what breaks or what is observed), detection and mitigation. One paragraph per entry, written as prose, not bullet points.
- **New entries must be specific and plausible.** Reject tautologies (cause equals effect), vague catch-alls, and bugs that would fail in a single process just as readily as in a distributed system.
- The README shows its format example as a blockquote (`>`); the category files use indented list items. Follow the category files, not the example's quoting.

## Verifying changes

With no automated checks, verification is manual. After an edit, confirm:

1. Entry numbers are sequential with no gaps or duplicates, across category boundaries: each file's first and last numbers match the range in the index, and each range starts one after the previous one ends.
2. Every entry has its indented paragraph directly beneath its title line.
3. The README table and the index table agree, sum to the stated total, and match the number of entries per category file.

A quick way to check counts without reading the files is `grep -cE '^[0-9]+\. ' observations/*.md`, which lists the entry count per category file. To see the first and last number of one file, use `grep -oE '^[0-9]+\. ' observations/<file>.md | sed -n '1p;$p'`.
