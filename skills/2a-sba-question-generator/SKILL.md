---
name: 2a-sba-question-generator
description: Use when asked to generate FRCR/FRCT Part 2A-style single-best-answer MCQs from a radiology paper (PDF, link, or pasted text). Verifies paper identity first, then builds the question.
version: 1.3.0
author: covie154
license: CC-BY-NC-SA-4.0 (commercial license reserved to the author)
source: https://github.com/covie154/2a_qg (base/prompts.py, base/class_gen.py)
portable: true   # self-contained prompt; runs in Claude Desktop, ChatGPT, Gemini, or any agent with web access
---

# 2A SBA Question Generator

Turn one radiology paper into exam-grade single-best-answer (SBA) questions, in the
style of FRCR/FRCT Part 2A specimen questions. You (the LLM executing this skill)
run the whole pipeline yourself. No code, no installs — just follow the phases in
order and respect the gates.

## Inputs

| Input | Required | Notes |
|---|---|---|
| Paper | yes | PDF file, URL (arXiv/journal/PubMed/PMC), DOI, or pasted text |
| `no_dx` | no | Number of questions to produce. Default **3** |
| `no_qn_per_dx` | no | Options per question. Fixed at **5** (1 correct + 4 distractors) |
| Topic focus | no | e.g. "pineal region", "sinonasal" — biases diagnosis selection |

**If no paper is supplied** — user just says "surprise me" / "give me questions" with no
PDF, URL, or DOI — route through the companion **2A Surprise-Me Paper Finder** skill
(paste-in: `surprise-me-system-prompt.md`, or in this folder). It searches the web,
applies the user's filters (FRCR 2A **subspecialty** — the six old-syllabus divisions
below —, topic/keyword, modality, paper type, year window, open-access preference,
rarity bias, count), scores candidates, verifies the citation against Crossref/PubMed,
keeps a `used_papers.md` novelty log, and hands the paper to you. If the finder is not
available, ask the user for a paper. Never invent one yourself.

The six FRCR 2A subspecialty divisions (the finder's filter vocabulary; both papers mix
all six, feedback is per division) — source
https://www.radiologycafe.com/exams/2a-exam (40 questions/division/paper; 120 SBA × 2
papers). Use the user's official list if they cite a different one:

1. Cardiothoracic & Vascular
2. Musculoskeletal & Trauma
3. Gastrointestinal
4. Genitourinary, Adrenal, Obstetrics & Gynaecology, Breast
5. Paediatric
6. Central Nervous System & Head & Neck

Even when the finder already verified the citation, **you still run your own Phase 1
identity check independently** — do not inherit the finder's verdict.

## Output modes

The run produces a **question artifact file** by default (Phase 10), built from
`templates/question-artifact-template.md` (bundled as `references/question-artifact-template.md`): question → answer → cited explanation →
sources, with the **source document pinned as citation [1]**. Attach the file; do not also
paste it into chat.

After attaching, you may enter **quiz mode** (Phase 11): pose only the question, take the
user's A–E, then return correct/wrong plus the cited explanation. Enter on request
("quiz me" / "ask me the question" / "interactive") or by a single offer after the
attachment. Never reveal the answer before they commit.

---

## PHASE 0 — ACQUIRE (get full paper text)

- **PDF provided** → extract the full text (use your app's file-read/PDF tool).
- **URL provided** → fetch the page. Prefer the free full text: PMC, arXiv, or the
  publisher's open-access page. If a paywall blocks full text, do NOT guess content
  from the abstract alone — tell the user and ask for the PDF.
- **DOI only** → resolve via `https://doi.org/<doi>` and the Crossref API
  (`https://api.crossref.org/?doi=<doi>`), then fetch full text if open access.
- **Pasted text** → use as-is.

If extracted text is fragmentary or garbled (broken OCR, tables only), say so and
stop. A question built on a mangled extract is worse than no question.

**No-paywall rule (hard):** you may not build a question set on anything short of the
full body text. An abstract, a summary, a figure legend, or your own recall of the
paper is NEVER a substitute — every Phase 4 element must trace to a sentence in the
extracted body. For papers the Surprise-Me finder supplied, the body text arrives with
the handoff (it only presents papers whose full text it extracted from PMC / Europe PMC
/ publisher open access; paywalled candidates are deleted before you ever see them). For
user-named papers, if the body is paywalled and the user has not supplied the PDF, stop
at this gate and request the PDF — do not proceed on the abstract and do not let the
user's own institutional access become your unstated precondition.

## PHASE 1 — VERIFY PAPER IDENTITY (hard gate)

Do this BEFORE writing any question. The question set must provably belong to the
paper the user named.

1. Extract from the text itself: **title, author(s), journal, year**, and the **DOI**
   if printed in the document (formats like `10.xxxx/...` or `doi.org/10....`).
2. Independently look it up — web search the exact title, and/or query Crossref:
   `https://api.crossref.org/works/doi:<claimed-doi>`.
3. Cross-check the claimed identity (user's title/link) against document-extracted
   identity against the registry record.
4. Record a **verification receipt**:

   ```
   VERIFIED: true | false | partial
   Title: ...
   Authors: ...  Journal: ... Year: ...
   DOI: ... (source: printed | registry | none found)
   Title-match: exact | close | mismatch
   Content-consistency: does the body match the claimed topic/figures captions? yes|no
   ```

**GATE RULES**
- `false` (mismatch) → STOP. Report the mismatch with both titles side by side.
  Never continue on the "wrong paper".
- `partial` (no DOI anywhere, or registry record missing) → proceed ONLY if title and
  content are consistent; state the caveat in your output.
- Never report a DOI you did not actually find. No DOI = write `none found`.

## PHASE 2 — HARVEST DIAGNOSES

From the full text, list every diagnosis the paper actually covers. A diagnosis is
**valid** only if the paper provides ALL of:
- demographic information (typical age range / sex predilection),
- clinical history (presentation / symptoms),
- imaging features (modality-specific findings).

Exclusions:
- Diagnoses listed only as differential afterthoughts with imaging features but no
  clinical history → invalid, drop them.
- Drop the paper's own case-report diagnosis if it is a one-off with no teaching
  generalisation.

Output (internal working JSON, not shown to user unless asked):
`{"Diagnoses": ["...", ...]}`

If fewer valid diagnoses than `no_dx`, reduce `no_dx` to what exists and tell the user.

## PHASE 3 — SELECT RARE/CHARACTERISTIC DIAGNOSES

Rank valid diagnoses by (a) rarity, (b) distinctiveness of imaging signature,
(c) separability from plausible mimics. Select the top `no_dx + 2` candidates.
Prefer diagnoses with a classic "textbook" appearance — a good SBA has one clearly
best answer. Avoid diagnoses whose imaging is non-specific.

## PHASE 4 — BUILD SCENARIOS

For each selected diagnosis, derive from THIS paper (never from generic memory):

```json
{
  "Patient_Age": 0,
  "Patient_Gender": "M or F",
  "Clinical_History": "presenting complaint + key clinical context",
  "Imaging_Modality": "e.g. MRI with contrast, CT, DSA",
  "Imaging_Findings": "findings as described in the paper, no diagnosis words",
  "Diagnosis": "the answer"
}
```

Rules:
- Age/sex must sit inside the demographic range the paper gives for that diagnosis.
- `Imaging_Findings` may describe appearance, location, enhancement, signal — but
  must not contain the diagnosis name or a pathognomonic eponym.
- Every element must be traceable to a sentence in the paper.

## PHASE 5 — STEM

Prose stem, formal concise English, exactly this skeleton:

> A `<Patient_Age>`-year-old `<male/female>` presents with `<Clinical_History>`.
> `<Imaging_Modality>` showed `<Imaging_Findings>`.
>
> **What is the most likely diagnosis?**

- M → "male", F → "female". Sentence count ≤ 3 before the question line.
- Keep the question line verbatim: "What is the most likely diagnosis?"

## PHASE 6 — DISTRACTORS (reasoned, not random)

Produce 4 distractors. Deliberate design, not filler:
1. **Near-miss**: same region/modality, differs by one decisive feature the stem
   already excludes (age, side, enhancement pattern, clinical context).
2–4. **Plausible mimics**: real differentials the paper itself mentions, each
   falsifiable by a stated finding.

All 5 options must be: same grammatical form, same granularity, no "all of the
above", no hints toward one option in the stem, no overlap/umbrella terms
(e.g. never include both "schwannoma" and "vestibular schwannoma").

For EACH option write: `Name` + `Explanation` — why it is correct / why it is
wrong, argued **from the paper's own content** (cite the feature, e.g. "the paper
notes X occurs in children, unlike this 40-year-old...").

## PHASE 7 — ASSEMBLE + SHUFFLE

1. Assert `Diagnosis == Option_Correct.Name` (case-insensitive). If not equal → fix before continuing.
2. Shuffle options into order A–E.
3. `Correct_Option_Index` = position of the correct option after shuffle.

## PHASE 8 — DIFFICULTY REFINEMENT

Redo the stem once to make it harder while keeping the answer defensible:
- Remove one non-essential clue (keep the decisive one).
- Tighten ambiguous wording so the answer stays uniquely best.
- Never leak the answer into the stem; never remove the last differentiating feature
  (a question with two defensible answers is a BAD question).
- Keep "What is the most likely diagnosis?" verbatim.

## PHASE 9 — SELF-VERIFICATION (hard gate, per question)

Check every item; if any check fails, regenerate that question once, then report the
failure honestly:

- [ ] Every stem fact appears in the paper (no hallucinated findings/epidemiology).
- [ ] Correct answer is unambiguously best given the stem.
- [ ] Each distractor is falsifiable by a stated stem feature and its explanation is correct.
- [ ] No answer leaks in the stem; no clue-word overlap that gives away the answer.
- [ ] Options homogeneous (same form/granularity); no umbrella overlaps.
- [ ] Age/sex consistent with paper demographics.
- [ ] Verification receipt from Phase 1 is still `true`/`partial` (re-check if you switched sources).
- [ ] Paper-content explanations cite the paper ([1]); no outside opinion presented as paper content.
- [ ] **Citation audit passes:** every `[n]` resolves to a ledger row retrieved this session;
      every cited ledger row carries a verbatim quote; no hand-typed URLs; explanation
      coverage ≥ 0.6; every figure/threshold/percentage is sourced or `[unverified]`;
      `Citation_Audit.pass: true` (or honestly `false` with a reason, offline fallback only).
- [ ] Sources block rendered from the ledger (not retyped); ids in prose match `Citations`.
- [ ] If web access was used at any point, each external claim is cited with its source.

## PHASE 10 — OUTPUT (the artifact is the default deliverable)

**The default output of a run is one artifact FILE per question**, not chat prose. Build it
from `templates/question-artifact-template.md` (bundled as `references/question-artifact-template.md`), write it to disk, and deliver it to the
user (attach it in chat apps; in an agent runtime, also state the path). The chat message
carries only a one-line summary plus the file — the file holds the question, the answer,
and the cited explanation.

Every artifact, in this order:
1. `## Question` — stem + options A–E + "What is the most likely diagnosis?"
2. `## Answer` — the correct letter and diagnosis name
3. `## Explanation` — why the answer is best, plus the five-option verdict table; every
   load-bearing claim carries a bracketed citation id
4. `## Sources` — rendered from the ledger. **Citation [1] is the source document** the
   question was built from: full citation + DOI/URL + retrieval date + a verbatim quote of
   the sentence carrying the teaching point. Everything else is corroboration, numbered
   [2], [3], …

Hard rules: the answer never appears above the question line; every number in the
explanation is sourced or marked `[unverified]`; no hand-typed URLs. File name
`<YYYY-MM-DD>_<first-author-surname>_<subspecialty-slug>_qn<N>.md`.

Also keep the machine-readable record (in the file footer or a sidecar JSON, not the chat
body) so banks can be reprocessed:

```json
{
  "Source": {"Title": "...", "Authors": "...", "Journal": "...", "Year": 0,
             "DOI": "doi.org/10.xxxx/... | none found", "Verified": true},
  "Question_Stem": "...",
  "Options": ["...", "...", "...", "...", "..."],
  "Correct_Option_Index": 0,
  "Explanation": "why the answer is correct, grounded in the paper",
  "Explanation_Other": ["Name: why wrong", "Name: why wrong", "Name: why wrong", "Name: why wrong"],
  "Citations": {"Explanation": [["fragment", [1]], ["fragment", [2,3]]], "Explanation_Other": {"Name": [["fragment",[ids]]]}},
  "Sources": [{"id": 1, "title": "<source document>", "url": "...", "retrieved": "DD-MM-YYYY", "quotes": ["verbatim"]}],
  "Citation_Audit": {"coverage": 0.0, "min_required": 0.6, "pass": true, "unverified_claims": 0, "dead_ends": []},
  "Notes": []
}
```

The artifact is the attachment. Do not paste the whole artifact into the chat as well —
that duplicates it and hides the file.

## PHASE 11 — QUIZ MODE (interactive self-test)

Two ways to enter quiz mode:
- **User-requested:** the user says "quiz me", "ask me the question", "interactive",
  "don't show me the answer" → go straight to the quiz flow.
- **Offered:** after you attach the full artifact, ASK ONCE whether they'd like to be
  posed the question before seeing the answer. If they decline or don't reply, stop — the
  artifact already stands on its own. Do not push it twice.

Quiz flow, once entered:
1. Emit a **QUIZ artifact** (same file, `_QUIZ` suffix) and/or post in chat containing
   ONLY the stem, options A–E, and the question line — no answer, no explanation, no
   spoiler-bearing sources list. Hold the answer key in your working context.
2. Ask the user to reply with a single letter (A–E). Wait; do not answer for them.
3. On their reply, judge it against the held key and respond:
   - correct → "**Correct.** <diagnosis>" then the full cited explanation + Sources block.
   - wrong → "**Not quite — the best answer is <letter>, <diagnosis>.**" then the full
     cited explanation, and specifically why *their* chosen option is falsifiable by a
     stated stem feature (use that option's row from the verdict table).
   - unrecognised input (not A–E, or blank) → re-ask once, clarifying the format; do not
     guess their intent.
4. Reveal nothing about the answer before they commit. A user who asks for a hint mid-quiz
   may get one *nudge* toward a stem feature, never the diagnosis name.

Multi-question runs: quiz them one at a time — present Q1, take the answer, give the
verdict and explanation, then move to Q2. Track the running score only if the user is
clearly doing a self-assessment session; report it at the end, unprompted narration
between questions is noise.

---

## PHASE 8.5 — EVIDENCE ENRICHMENT (cited explanations; required when web access exists)

Explanations must NOT rest on the paper alone or on model memory. After the question is
built (Phase 8) and BEFORE self-verification (Phase 9), go get complementary external
evidence and cite every load-bearing claim inline. This follows the house
`grounded-citations` discipline: **ids and URLs come from retrieval, never from memory;
the model only emits small integers it was handed.**

### 8.5.0 Ledger rules (the anti-hallucination contract)

1. Open a **citation ledger** for this run: a table `id | title | URL | retrieved | verbatim quote`.
   - **Agent runtime with shell** (Hermes/Codex/Claude-Code-like): use the
     `grounded-citations` script —
     `S=~/.hermes/skills/research/grounded-citations/scripts/sources.py;
      python "$S" reset; python "$S" add <url> --title T` → it prints `[n]`;
     `python "$S" quote <id> --text "…" --from page.txt`;
     `python "$S" render --style evidence --replace-in <draft>`.
   - **Portable runtime** (Claude Desktop/ChatGPT with browsing): maintain the table by
     hand in your working context. Add a row at the moment of retrieval, not later.
     A URL that never appeared in a search/extract result may not enter the ledger.
2. Register at retrieval time. Registering after drafting = the failure mode this exists
   to prevent.
3. Citation syntax: bracketed ids immediately after the sentence, no preceding space,
   one id per bracket, max 3 ids per sentence:
   `Simple cysts ≤3 cm in premenopausal women are follicles and need no follow-up.[2][3]`
4. Never invent an id, a URL, or a DOI. Never retype a URL into a Sources block from
   memory — regenerate it from the ledger.
5. Register the **source paper itself** as ledger id [1] at Phase 1 (its verified
   DOI/landing URL). All paper-sourced sentences cite [1]; enrichment sentences cite
   the added ids.

### 8.5.1 Enrichment searches (run all that apply per question)

| ID | Search | Purpose / what it may support |
|---|---|---|
| E1 | DOI/Crossref/PubMed registry record of the source paper | Canonical citation metadata into `Source`; never a content claim |
| E2 | The stem's key discriminating fact (prevalence, age/sex rule, threshold) | Corroborates or flags the paper's teaching; primary guidelines > single case reports |
| E3 | `"<correct dx>" vs "<each distractor>"` differentiating features | Upgrades the distractor explanations from "not this" to "this specific feature excludes it" |
| E4 | The distractor entities' own defining imaging features | Confirms each distractor is a real, describable entity (kills LLM-invented distractors) |
| E5 | Guideline/consensus statement governing the topic (ACR/SRU/IOTA/O-RADS/BI-RADS/LI-RADS etc.) | Gives the explanation the management/standard-of-care anchor the 2A exam expects |

Retrieval quality rules:
- Prefer the free full text (PMC, DOI landing, guideline PDF). A search **snippet** supports
  only what the snippet literally says; if the claim needs the body, extract the page first.
- 3–6 enrichment sources per question is the working range; more than 10 usually means
  over-citing.
- Log dead ends honestly ("no source found for X") instead of smoothing them over.

### 8.5.2 Evidence mode (hard gate — this is the house quality bar)

Because these questions are medico-legal-adjacent teaching artefacts, enrichment runs in
**evidence mode**, not just citation mode:

- Attach a **verbatim quote** from the extracted page to every ledger row that a claim
  depends on. Copy-paste from retrieved text; never retype or paraphrase into a quote.
- Any load-bearing claim you could not source gets an explicit `[unverified]` marker —
  that is declared provenance, not a dodge. If most sentences carry `[unverified]`, the
  run needed more retrieval; say so.
- Every URL must be **HTTP-resolvable** — you fetched it this session.
- **Coverage gate:** ≥ **0.6** of explanation prose sentences must declare provenance
  (`[n]` or `[unverified]`). In an agent runtime, enforce it mechanically:
  `python "$S" verify <draft> --evidence --min-coverage 0.6` and fix until green.
  In a portable runtime, self-check the ratio and state it in `Citation_Audit`.
- Numbers (percentages, sizes, ages, thresholds, intervals) may only appear in an
  explanation if they trace to ledger evidence or to the source paper. A recalled figure
  with no source is either searched or marked `[unverified]`.

### 8.5.3 Conflict handling

If wider literature contradicts the source paper: **the paper stays the exam authority**
(the question must remain answerable from it), and you add a `Notes` entry naming the
discrepancy with both ids cited. One source is reporting; two independent sources are
corroboration — say which you weight and why.

### 8.5.4 Output additions

Per question JSON gains:

```json
"Citations": {
  "Explanation": [["sentence fragment", [1, 5]], ["...", [3]]],
  "Explanation_Other": { "<DistractorName>": [[fragment, [ids]], ...] }
},
"Sources": [
  {"id": 1, "title": "<source paper>", "url": "https://doi.org/10....",
   "retrieved": "YYYY-MM-DD", "quotes": ["verbatim supporting sentence"]},
  {"id": 2, "...": "..."}
],
"Citation_Audit": {
  "coverage": 0.75, "min_required": 0.6, "pass": true,
  "unverified_claims": 0, "dead_ends": ["no source found for X"]
},
"Notes": ["paper vs guideline discrepancy on threshold X, weighted [1] as exam authority"]
```

Append a rendered `## Sources` block after the plaintext card — ids, titles, live URLs,
and the verbatim quote under each (evidence style). Never hand-type that block; generate
it from the ledger.

**Offline fallback:** no web access → skip 8.5, mark all enrichment claims `[unverified]`,
set `Citation_Audit.pass: false` with `"reason": "no web access"`, and say in your reply
that the explanations are un-enriched. Do not fake citations to satisfy the shape.

---

## JSON HYGIENE (applies to every internal step)

If you must emit JSON from a model call and it breaks: extract the outermost
`{ ... }` with greedy match, repair common defects (trailing commas, smart quotes,
unescaped newlines), retry once. If it still fails, report the raw text — do not
fabricate the missing data.

## MULTI-QUESTION RULES

- Repeat Phases 4–9 per diagnosis. Options sets may not reuse distractors across
  questions in the same batch unless the correct answer differs.
- Vary modality focus across questions where the paper supports it (CT vs MR vs US).
- Return all question JSONs as an array; then all plaintext cards.

## FAILURE MODES — report, do not paper over

| Situation | Behaviour |
|---|---|
| Paper identity mismatch | Stop, show both identities |
| Paywall, no full text | Ask for the PDF |
| < no_dx valid diagnoses | Reduce count, say why |
| OCR garbage | Stop, ask for clean text |
| Web lookup fails | Mark DOI `none found`, continue offline path |
| Self-check fails twice for one question | Drop it, report it failed and why |
