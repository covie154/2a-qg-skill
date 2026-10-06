---
name: 2a-surprise-me-paper-finder
description: Use when picking a paper for 2A question generation.
version: 1.1.0
author: covie154
license: MIT
metadata:
  hermes:
    tags: [radiology, frcr-2a, paper-discovery, companion-skill, web-search]
    related_skills: [2a-sba-question-generator, grounded-citations]
portable: true
---

# 2A Surprise-Me Paper Finder

## When to Use

- User says "surprise me", "pick a paper for me", "find me a 2A question paper", or
  asks for questions without naming a paper.
- User names only a subspecialty or topic ("give me something on the temporal bone")
  and wants a paper chosen for them.
- Companion to `2a-sba-question-generator`, which makes the questions. This skill only
  FINDS and HANDS OFF the paper; it never writes questions itself.
Do NOT use when: the user already supplied a PDF/URL/DOI → go straight to the
question generator.

## Inputs (all optional — sensible defaults, do not interrogate the user)

| Filter | Default | Values |
|---|---|---|
| Subspecialty | **random** | one of the six FRCR 2A divisions below |
| Topic/keyword | none | free text, e.g. "pineal region", "pulsatile tinnitus" |
| Modality focus | none | plain film / US / CT / MRI / fluoro / nuclear / hybrid |
| Paper type | review | review / case series / pictorial review / consensus / meta-analysis |
| Year window | 2015- | `YYYY-YYYY`; 2A questions age badly past ~10 years |
| Full text | **hard requirement** | open-access full text ONLY — paywalled candidates are
never offered and never handed to the generator. See "FULL-TEXT GATE" below. |
| Rarity bias | on | prefer unusual diagnoses with characteristic imaging, not bread-and-butter |
| Count | 1 | how many papers to return (max 5 shortlist / 1 default handoff) |

If the user gives nothing, run fully random across the six divisions. That is the point
of "surprise me" — do not ask a single question back unless the request is
self-contradictory.

## FULL-TEXT GATE (absolute — no exceptions)

A candidate may enter the shortlist only if the **whole article text is retrievable by
you, in this session, right now**. Rules:

- **Accepted sources of full text** (priority order):
  1. PMC article page — `pmc.ncbi.nlm.nih.gov/articles/PMC<id>` (best; whole body extracts)
  2. Europe PMC — `europepmc.org/articles/PMC<id>` (use when NCBI blocks or is slow)
  3. Publisher open-access page / DOI landing that serves the body (not just abstract)
  4. Journal's own free-review venue page with visible body text
- **Disqualifying — drop immediately, do not present, do not ask the user for a PDF:**
  - abstract-only records (PubMed without a PMCID is abstract-only — the PMCID is the
    tell; no PMCID → assume paywalled until proven open)
  - any page that returns a login wall, "Access through institution", purchase offer,
    or `Failed to fetch` on the article route
  - a DOI that resolves only to a paywalled publisher page
  - preprint-only or "available at [publisher]" stubs
- **The test is extraction, not optimism.** Fetch the article page with your browse/extract
  tool and confirm **at least 3 body sections** are present in the returned text (e.g.
  Introduction + ≥2 named-entity sections + Discussion/Conclusion), or ≥8,000 characters
  of continuous body prose. An abstract + reference list does NOT qualify.
- **Never claim access you did not have.** If the fetch failed, the paper is not a
  candidate — full stop. Do not hand the generator a paper on the strength of an abstract
  and let it "try".
- Record the evidence in the presentation: `Full text: PMC12345678 (verified,
  46,000 chars, sections: Intro/…/Discussion)`.

The user has institutional/PACS access outside this sandbox. That is *their* route, not
yours: if the perfect paper exists but is paywalled, name it, state it is paywalled and
dropped, and pick an open one instead — never make the user's fetch your precondition.

## The six FRCR 2A subspecialties (old-format syllabus divisions)

Both 2A papers are a mix of all six; results feedback is broken down per division.
Use these as the `subspecialty` filter vocabulary:

1. **Cardiothoracic & Vascular** — chest, heart, great vessels, lung, mediastinum
2. **Musculoskeletal & Trauma** — bones, joints, soft tissue tumours, sports, trauma
3. **Gastrointestinal** — bowel, liver, spleen, pancreas, biliary, peritoneum
4. **Genitourinary, Adrenal, Obstetrics & Gynaecology, Breast** — kidney, bladder,
   adrenal, pregnant pelvis, breast screening
5. **Paediatric** — neonate to adolescent, any body region, all modalities
6. **Central Nervous System & Head & Neck** — brain, spine, pituitary/pineal, ear,
   nose, orbit, salivary gland, thyroid, neck nodes, bone

Source for the taxonomy: https://www.radiologycafe.com/exams/2a-exam (40 questions per
division per paper, 120 SBA per paper, 2 papers). Registry may change — if the user
cites a different division list, use theirs.

## Procedure

### ① Pick the division
Random selection unless filtered. For genuine spread, weight toward divisions the user
has NOT drawn recently — read the run log first (step ⑤).

### ② Search (fan out, then score)
Query patterns that work well, in priority order:

| Route | Query shape | Why |
|---|---|---|
| A | `site:pmc.ncbi.nlm.nih.gov "<topic/division>" review imaging differential diagnosis` | PMC is the only guaranteed full-text corpus; lead here |
| B | `"<topic/division>" "available in PMC" OR PMCID imaging review` | surfaces open-access copies directly |
| C | `Radiographics OR "Insights into Imaging" "<topic>" imaging review` | teaching-style reviews; then prove OA status via PMC/DOI |
| D | `"<division>" imaging findings differential diagnosis pictorial review` | mimics the specimen-question shape |
| E | `"<rare dx>" case report OR case series imaging <division> PMCID` | rarity bias, still open-access filtered |

Collect 8-15 candidates across routes, then run every one through the FULL-TEXT GATE
before scoring. Candidates that fail the gate are deleted from the list, not annotated.

### ③ Score candidates (pick the best, announce the scores)

| Criterion | Points | Gate |
|---|---|---|
| Full text extracted and verified this session (per FULL-TEXT GATE) | 4 | NOT extracted / partial / abstract-only → DROP |
| Teaches ≥3 diagnoses with demographics + history + imaging (the generator's validity rule) | 3 | 0 → drop |
| Contains rare/characteristic-appearance entities (good SBA material) | 2 | |
| Review or pictorial-review type, not a raw technical/physics paper | 1 | physics-only → drop |
| Within year window | 1 | |
| Modality variety in the paper (CT + MR + US) | 1 | |
| Not in the run log (novelty) | 2 | repeat → -2 |

Take the top scorer. Score 11+ = strong candidate; 8-10 = acceptable, say it's a
second-tier pick; below 8 → search again with a broader query before settling.
Everything on the scored list is paywalled-free by construction — you extracted it.

### ④ Verify before handoff (do NOT skip)
Before proposing, confirm the paper is real and reachable:
- Re-fetch the full-text page and confirm the body you scored is the same article the
  registry describes. Resolve the DOI (`https://doi.org/<doi>` or Crossref
  `https://api.crossref.org/works/doi:<doi>`) and check title/journal/year match the
  search hit. No DOI and no registry record → treat as suspect; a title that resolves to
  nothing is a hallucinated citation, not a candidate.
- Confirm the extracted body actually teaches imaging of the claimed subspecialty (not a
  clinical/surgical paper with one figure, not a physics/AI-methods paper).
- Register every consulted URL in the citation ledger as you go (grounded-citations rule:
  ids come from retrieval, never from memory). Hand the generator the **extracted full
  text**, not a link it must chase.

### ⑤ Log the run (so "surprise me" stays surprising)
Append to `data/used_papers.md` (create on first use; table:
`date | subspecialty | title | first author | year | journal | DOI | questions generated`),
then read it back at the start of every run for the noveltty score. Never hand back a
paper already in the log unless the user asks for a repeat.

### ⑥ Present + hand off
Report in this shape, then continue into the question generator in the SAME turn — do
not stop and ask permission:

```
Picked: <Title>
        <First author et al>. <Journal> <Year>;<vol>(<issue>):<pages>.
        DOI: doi.org/<doi>   [registry-confirmed]
        Subspecialty: <one of six>
        Why this one: <2 lines — what makes it good 2A material>
        Score: <n>/14
        Full text: <PMC id / OA URL> — verified extracted, <n> chars, sections seen:
                   <list>  [open access required; paywalled never presented]
```

Then invoke `2a-sba-question-generator` on the extracted text, inheriting any filters
the user set (question count, rarity bias). Hand it the text you already pulled, plus the
canonical URL/DOI. The generator re-runs its own Phase 1 identity check — do not tell
it the paper is verified; let it verify independently.

## Failure modes — report, don't paper over

| Situation | Behaviour |
|---|---|
| Division/topic yields only paywalled hits | Say so plainly, name the rejected paywalled papers with the reason, then widen the query (older years, neighbouring topic, other journals, Europe PMC) until an open-access paper passes the gate. Never fall back to "ask the user for the PDF" as the default outcome. |
| Fetch returns partial/truncated body | Retry via the Europe PMC mirror once. Still partial → drop; a partial extract is not full text. |
| DOI will not resolve | Drop the candidate; never hand over an unverifiable citation |
| Division yields nothing good | Widen to a neighbouring topic, say what you widened and why |
| Log missing/corrupt | Rebuild it from nothing and continue; note it |
| User's filters contradict (e.g. paediatric + breast screening + year <2010) | Say so and offer the nearest workable combination |

Portable paste-in version: `references/surprise-me-system-prompt.md`, and
`prompts/surprise-me-system-prompt.md`.
