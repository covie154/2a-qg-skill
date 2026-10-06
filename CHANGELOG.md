# Changelog

All notable changes to these skills. Versions are shared across the generator and finder
so the paired prompts always travel togetha.

## [1.2.1] — 06-10-2026

### Added
- `README.md`, `CONTRIBUTING.md`, `CHANGELOG.md`, `.gitignore`, `LICENSE` (MIT).
- Repository layout split into `skills/`, `prompts/`, `examples/`, `data/`.
- Rejected-paper audit trail in `data/used_papers.md` (separate from handoffs).

### Changed
- **Full text is now a hard gate at both ends.** The finder deletes paywalled candidates
  before the shortlist and must prove in-session extraction (≥ 3 body sections or
  ≥ 8,000 characters of prose). The generator refuses to build questions on an abstract,
  summary, figure legend, or model recall.
- Finder full-text criterion re-weighted to 4 points (was 3) with a hard drop; scoring
  maximum is now 14, thresholds 11+ strong / 8-10 second-tier.
- Finder search routes re-ordered to lead with `site:pmc.ncbi.nlm.nih.gov`, since PMC is the
  only guaranteed full-text corpus.
- Presentation block must now state the extraction proof: PMC id or OA URL, character
  count, and the section headings actually seen.
- Handoff passes extracted body text to the generator instead of a link it must chase.

## [1.2.0] — 06-10-2026

### Added
- Companion skill **2a-surprise-me-paper-finder**: web search, scoring, verification,
  novelty log, and automatic handoff to the generator in the same turn.
- Six FRCR 2A subspecialty divisions as the filter vocabulary (Cardiothoracic & Vascular,
  Musculoskeletal & Trauma, Gastrointestinal, GU/Adrenal/OB-GYN/Breast, Paediatric,
  CNS & Head & Neck).
- Filters: subspecialty, topic/keyword, modality, paper type, year window, rarity bias,
  count. Empty input runs fully random.

## [1.1.0] — 06-10-2026

### Added
- **Phase 8.5 evidence enrichment.** Citation ledger registered at retrieval time;
  verbatim quote per cited row; ≥ 0.6 provenance coverage on explanations; every number
  traced or marked `[unverified]`; snippet-only claims limited to what the snippet states.
- Enrichment routes E1-E5: registry metadata, stem discriminating fact, correct-vs-distractor
  differentiators, distractor defining features, governing guideline/consensus.
- Output fields `Citations`, `Sources`, `Citation_Audit`, `Notes`.
- Phase 9 grew three citation-integrity checks; Sources block rendered from the ledger,
  never retyped.
- Offline fallback: audit reports `pass: false` with a reason. No fabricated citations.

## [1.0.x] — 06-10-2026

### Added
- **2a-sba-question-generator**: ten-phase portable pipeline with two hard gates
  (Phase 1 identity verification, Phase 9 self-verification).
- Phase 1 verification receipt: title, authors, journal, year, DOI with source tag
  (printed | registry | none found), title-match grade, content-consistency.
- Diagnosis validity rule: a diagnosis qualifies only with demographics **and** clinical
  history **and** imaging features.
- Distractor design: one near-miss plus three paper-listed mimics, each falsifiable by a
  stated stem feature; homogeneous options, no umbrella overlap.
- Output as question JSON plus a plaintext exam card.
- No code, no API keys: the same pipeline runs in Claude Desktop, ChatGPT, Gemini, or any
  agent with web access.

### Derived from
- Original Python/Django pipeline at `covie154/2a_qg` — `base/prompts.py` prompt library
  (`create_dx`, `choose_rare_dx`, `create_qn`, `create_text`, `cot_*`, `refine_qn_cot_*`,
  `get_doi`, `output_format`) and `base/class_gen.py` orchestration, re-expressed as
  prose phases so the logic is portable.
