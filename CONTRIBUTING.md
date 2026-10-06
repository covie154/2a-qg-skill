# Contributing

The value of these skills sits in their gates, not their prose. Changes that weaken a gate
are not accepted, however well worded.

## What counts as a good contribution

| Area | Welcome | Notes |
|---|---|---|
| Evidence routes | new Phase 8.5 search rows (E6, E7...) | must inherit verbatim-quote + provenance rules; say what the route may support |
| Scoring | finder criteria, point weights, gates | keep the full-text criterion the top-weighted hard gate |
| Question formats | new Phase 5 skeletons (management step, "least likely", causative structure) | one skeleton per format; keep the stem formal and ≤ 3 sentences |
| Failure modes | new failure case + its honest behaviour | "report, don't paper over" is the rule |
| Taxonomy | corrected or newer FRCR 2A division list | cite the official source |
| Output schema | added fields to the question JSON | keep `Source.Verified` and `Citation_Audit` |

## Invariants — pull requests that break these are closed

1. **No question is generated from less than full body text.** Abstracts, summaries,
   figure legends, and model recall never substitute. Accepted sources: PMC, Europe PMC,
   publisher open-access pages, or a user-supplied PDF.
2. **The full-text test is extraction, not optimism.** At least three named body sections
   present in the fetched text, or ≥ 8,000 characters of body prose.
3. **Citation ids and URLs come from retrieval, never from memory.** The model emits
   integers a ledger handed it. A URL that never appeared in a search or extract result
   may not enter the ledger.
4. **Every cited source carries a verbatim quote** copy-pasted from the fetched page, and
   the URL must have been fetched in that session.
5. **Explanation provenance coverage ≥ 0.6**, with unsourceable claims explicitly marked
   `[unverified]` rather than left to look sourced.
6. **Every number traces somewhere.** Age, size threshold, percentage, interval — ledger or
   source paper, or it is searched or marked.
7. **A DOI is never reported unless actually found.** Absent means `none found`.
8. **A title that resolves to nothing is a hallucinated citation.** Drop the candidate.
9. **The source paper stays the exam authority.** External corroboration may flag a
   discrepancy; it may not silently overwrite the paper.
10. **Verification is never inherited.** The generator re-runs Phase 1 even after the
    finder verified the citation.
11. **Fabricated output is the one unacceptable result.** A stated blocker always beats a
    plausible invention.

## Editing rules

- Keep the Hermes frontmatter `description` at **≤ 57 characters** — longer values get
  truncated in the skill index and the routing signal is lost. Detail belongs in the body.
- Keep `SKILL.md` (the agent skill) and its `prompts/` twin in step. The prompt file is the
  frontmatter-stripped, paste-ready form of the same procedure; a change to one is a change
  to the other.
- Preserve the phase numbering. Other documents and users' muscle memory cite phases by
  number (`8.5` is deliberately fractional so it slots between 8 and 9).
- Do not introduce runtime dependencies. If a contribution needs Python, the pipeline is no
  longer drop-in for Claude Desktop or ChatGPT — keep such work in a separate tool.
- Sentence style: active voice, short sentences, no marketing qualifiers. Plain claims.

## Testing a change

Before opening a pull request, run the edited skill end to end in a real session and paste:

1. the **Phase 1 verification receipt** (`VERIFIED`, title, authors, journal, year, DOI with
   its source tag, title-match, content-consistency);
2. the **citation ledger** for the run, including which rows were snippet-provenance;
3. the **`Citation_Audit`** block with coverage and pass status;
4. one **plaintext exam card** as delivered.

Then test at least two failure paths and show the honest output:

- a paper whose DOI will not resolve → must be dropped, not presented;
- a paywalled-only candidate → must be deleted from the shortlist, with the reason recorded
  in `data/used_papers.md` Rejected;
- (recommended) a partial/truncated extract → mirror retried once, then dropped.

A change that produces questions but cannot show these artefacts has not been tested.

## Housekeeping

- `data/used_papers.md` accumulates. Long-running banks may be split by year; keep the
  Handoffs / Rejected split and the novelty-score contract intact.
- Personal run logs and one-off extracts do not belong in the repository. Keep them in your
  own working directory.
- Never commit journal article full text. A DOI, a PMCID, and a title are enough to identify
  a paper; the text stays with the reader's own access.

## Naming

Skill names are lowercase with hyphens, ≤ 64 characters, and the first 57 characters of the
description must read as a self-contained trigger ("Use when ...").
