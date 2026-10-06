# 2A SBA Question Generator

> Turn one radiology paper into exam-grade single-best-answer (SBA) questions in the
> style of FRCR/FRCT Part 2A — verified first, cited throughout, no paywalled shortcuts.

Portable, self-contained prompt skills. No Python, no API keys, no installs. Paste them
into **Claude Desktop, ChatGPT, Gemini**, or any agent with web access, and run the whole
pipeline in-model.

Built from the original pipeline at
[covie154/2a_qg](https://github.com/covie154/2a_qg) (`base/prompts.py`, `base/class_gen.py`),
re-expressed as prompts so the logic travels with you instead of sitting in a Django app.

---

## The two skills

| Skill | Role | Load it when |
|---|---|---|
| **2a-sba-question-generator** | Builds the questions | you have a paper (PDF / URL / DOI / pasted text) |
| **2a-surprise-me-paper-finder** | Finds the paper | you have *no* paper and just want questions |

### 1. 2A SBA Question Generator

Ten phases, two hard gates:

```
0  ACQUIRE      full body text from PDF / URL / DOI / pasted text
1  VERIFY       ← HARD GATE — paper identity cross-checked against Crossref/PubMed
2  HARVEST      diagnoses that carry demographics + history + imaging (all three)
3  SELECT      rarest, most characteristic imaging signature
4  SCENARIOS   age, sex, history, modality, findings — every element traceable
5  STEM        fixed skeleton, question line verbatim
6  DISTRACTORS 1 near-miss + 3 paper-listed mimics, each falsifiable
7  ASSEMBLE    assert answer == correct option, shuffle A–E
8  REFINE      strip one non-essential clue, keep answer defensible
   8.5 ENRICH  ← cited external evidence, ledgered, ≥0.6 coverage
9  SELF-VERIFY ← HARD GATE — 12-point audit incl. citation integrity
10 OUTPUT      question JSON + plaintext exam card + Sources block
```

### 2. 2A Surprise-Me Paper Finder

Search → score → **prove you hold the full text** → hand off as extracted text.
Filters: FRCR 2A subspecialty (the six syllabus divisions), topic, modality, paper type,
year window, rarity bias, count. Zero input = fully random draw.

---

## Quick start

**Paste into a chat app** — use the `prompts/` files:

- `prompts/generator-system-prompt.md` → custom instruction / project prompt
- `prompts/surprise-me-system-prompt.md` → paste when you want it to pick the paper

Both are self-contained. Load the finder prompt *and* the generator prompt together for
the full "surprise me → questions" flow.

**Install as Hermes skills:**

```bash
cp -r skills/2a-sba-question-generator  ~/.hermes/skills/
cp -r skills/2a-surprise-me-paper-finder ~/.hermes/skills/
```

Then say *"make me 3 2A questions from <paper>"* or *"surprise me"*.

---

## Repository layout

```
.
├── skills/
│   ├── 2a-sba-question-generator/
│   │   ├── SKILL.md                    full 10-phase spec + gates
│   │   └── references/                 paste-in copies + worked sample
│   └── 2a-surprise-me-paper-finder/
│       ├── SKILL.md                    finder procedure + full-text gate
│       └── references/                 paste-in copy + run log template
├── prompts/                            paste-ready system prompts (no frontmatter)
│   ├── generator-system-prompt.md
│   └── surprise-me-system-prompt.md
├── examples/
│   └── sample-output.md                real run: receipt, ledger, JSON, exam card
├── data/
│   └── used_papers.md                  novelty log + rejected-paper audit trail
├── README.md
├── LICENSE
└── CONTRIBUTING.md
```

---

## The rules that make it trustworthy

These are the parts that matter. The rest is formatting.

### Paper identity is verified, not assumed
Before any question is written, the skill extracts title/authors/journal/year/DOI **from
the document itself**, looks the paper up independently in Crossref/PubMed, and cross-checks
three identities. Mismatch → stop and show both titles side by side. A DOI is never
reported unless actually found; absent means `none found`.

### No paywalled papers. Ever.
Questions may not be built on an abstract, a summary, a figure legend, or model recall.
Full body text must come from **PMC, Europe PMC, a publisher open-access page, or a PDF
you supply**. The finder deletes paywalled candidates before they reach the shortlist —
and the test is *extraction, not optimism*: at least three named body sections must be
present in the fetched text.

The generator re-checks this independently. Your institutional/PACS access is explicitly
*not* an acceptable fallback, because then the output depends on something the skill never
saw.

### Explanations are cited, with receipts
Phase 8.5 runs in **evidence mode** (borrowed from the `grounded-citations` discipline):

- a citation ledger is populated **at retrieval time** — ids and URLs never come from
  memory, the model only emits integers a ledger handed it
- every cited source carries a **verbatim quote** copy-pasted from the fetched page
- every URL must be HTTP-resolvable — fetched in that session
- **≥ 0.6** of explanation sentences must declare provenance (`[n]` or `[unverified]`)
- every number, age, size threshold, and interval traces to the ledger or the paper
- a snippet supports only what the snippet literally says

If web access is unavailable, the run says so and marks `Citation_Audit.pass: false`
with a reason. It does not fabricate citations to satisfy the shape.

### The paper stays the exam authority
When wider literature contradicts the source paper, the paper wins for question purposes
(the question must remain answerable from it) and the discrepancy is flagged in `Notes`
with both sources cited.

### Failure is reported, not papered over
Every failure mode has a defined honest behaviour: identity mismatch → stop; DOI will not
resolve → drop the candidate; partial extract → retry mirror once then drop; self-check
fails twice → drop that question and say why. Hallucinating a plausible-looking result is
the one unacceptable outcome.

---

## Example output

From `examples/sample-output.md` — the O-RADS guideline review, one question:

```
A 34-year-old premenopausal woman undergoes pelvic US for chronic pelvic discomfort.
A 2.4-cm unilocular adnexal cyst is identified with anechoic contents, a thin smooth
wall, and no solid, papillary, or vascular components.

What is the most likely diagnosis?

A. Ovarian cystadenoma
B. Follicle cyst
C. Endometrioma
D. Corpus luteum cyst
E. Cystic ovarian malignancy

******************************************

Correct Option: B

Explanation:
Simple cysts ≤3 cm in premenopausal women are almost certainly follicles — normal
physiologic findings needing no follow-up [1,2]. SRU 2019: exceptionally low cancer
risk; report at 3 cm, follow-up only >5-7 cm [2]. Thin smooth wall, anechoic contents,
absent flow = the paper's simple-cyst definition [1].

Ovarian cystadenoma: cysts that GROW are likely cystadenomas (SRU) [2]; this one is small, simple.
Endometrioma: expect homogeneous low-level 'ground-glass' echoes [4] — not anechoic.
Corpus luteum cyst: expect thick crenulated wall, internal echoes, intense peripheral flow [1].
Cystic ovarian malignancy: O-RADS 3-5 features (solid components etc.) absent [1,3].
```

The full example also shows the verification receipt, the citation ledger, the JSON
schema with `Citations` / `Sources` / `Citation_Audit` / `Notes`, and why the run passes
its own gate (coverage 0.78).

---

## The six FRCR 2A subspecialties

The finder's filter vocabulary. Both 2A papers mix all six; results feedback is broken
down per division.

1. Cardiothoracic & Vascular
2. Musculoskeletal & Trauma
3. Gastrointestinal
4. Genitourinary, Adrenal, Obstetrics & Gynaecology, Breast
5. Paediatric
6. Central Nervous System & Head & Neck

Format: 2 papers × 120 SBA × 3 h; 40 questions per division per paper.
Taxonomy source: [Radiology Cafe](https://www.radiologycafe.com/exams/2a-exam).
If the RCR publishes a different official division list, use theirs — the vocabulary is a
convenience, not doctrine.

---

## Extending it

The skills are deliberately open at the seams:

- **More evidence sources in Phase 8.5** — the enrichment table (E1–E5) takes new rows
  without touching the pipeline. Add a route, say what it may support, and it inherits
  the verbatim-quote and provenance rules automatically.
- **Scoring in the finder** — the 14-point table is a list, not code. Add a criterion,
  set points and a gate.
- **Question formats** — the repo's original `test_qn_gen.py` notes non-diagnosis stems
  (causative structure, "least likely diagnosis", management step). Those slots exist;
  adding them means one new Phase 5 skeleton.
- **Bank building** — run the finder repeatedly; `data/used_papers.md` prevents recycling,
  and the Rejected table keeps a record of what was excluded and why.

Contributions welcome — see `CONTRIBUTING.md`.

---

## Licence

MIT — see `LICENSE`. Text content and prompts; no medical advice, exam material is for
revision practice and remains the property of the RCR.
