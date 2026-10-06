# Sample Run — End-to-End Output of the 2A SBA Question Generator Skill (v1.1, with Phase 8.5 evidence enrichment)

Input: `Papers/Radiographics O-RADS.txt` (from repo covie154/2a_qg), no_dx = 1 (demo).

## Phase 1 verification receipt (produced by an actual run, 06-10-2026)

```
VERIFIED: true
Title:    Benign-appearing Incidental Adnexal Cysts at US, CT, and MRI: Putting the
          ACR, O-RADS, and SRU Guidelines All Together
Authors:  Peter S. Wang, Otto G. Schoeck, Mindy M. Horowitz
Journal:  Radiographics (RSNA), 2022; 42(2):609-624
DOI:      doi.org/10.1148/rg.210091   (source: registry — PubMed/RSNA listings; not printed
          in the txt extract)
Title-match: exact (document abstract matches registry abstract verbatim)
Content-consistency: yes — body discusses SRU 2019, O-RADS US 2020, ACR white paper
Sources: https://pubmed.ncbi.nlm.nih.gov/35061515 | https://pubs.rsna.org/doi/abs/10.1148/rg.210091
```

## Phase 8.5 citation ledger (this demo run)

| id | title | URL | retrieved | method | verbatim quote anchor |
|---|---|---|---|---|---|
| 1 | Wang 2022 source paper (Radiographics) | https://pubmed.ncbi.nlm.nih.gov/35061515 | 2026-10-06 | search-registered, identity verified Phase 1 | "Three major consensus articles on adnexal cystic masses were published between 2019 and 2020" |
| 2 | Levine et al. SRU consensus update, Radiology 2019;293(2):359-371 | https://pubmed.ncbi.nlm.nih.gov/31549945 | 2026-10-06 | web_extract (full abstract) | see Sources block below |
| 3 | Andreotti et al. O-RADS US consensus guideline, Radiology 2020;294(1):168-185 | https://pubmed.ncbi.nlm.nih.gov/31687921 | 2026-10-06 | web_extract (full abstract) | see Sources block below |
| 4 | MDedge: Imaging the endometrioma and mature cystic teratoma | https://www.mdedge.com/content/imaging-endometrioma-and-mature-cystic-teratoma | 2026-10-06 | search snippet only (supports ground-glass feature, nothing more) | see Sources block below |

Provenance note on [4]: cited strictly for the endometrioma echo-texture claim the
snippet literally states — per the skill's snippet rule, no body-level claim rides on it.

## Phase 10 output

```json
{
  "Source": {
    "Title": "Benign-appearing Incidental Adnexal Cysts at US, CT, and MRI: Putting the ACR, O-RADS, and SRU Guidelines All Together",
    "Authors": "Wang PS, Schoeck OG, Horowitz MM",
    "Journal": "Radiographics 2022;42(2):609-624",
    "DOI": "doi.org/10.1148/rg.210091",
    "Verified": true
  },
  "Question_Stem": "A 34-year-old premenopausal woman undergoes pelvic US for chronic
    pelvic discomfort. A 2.4-cm unilocular adnexal cyst is identified with anechoic
    contents, a thin smooth wall, and no solid, papillary, or vascular components.
    What is the most likely diagnosis?",
  "Options": [
    "Ovarian cystadenoma",
    "Follicle cyst",
    "Endometrioma",
    "Corpus luteum cyst",
    "Cystic ovarian malignancy"
  ],
  "Correct_Option_Index": 1,
  "Explanation": "Simple cysts 3 cm or smaller in premenopausal women are almost
    certainly follicles — a normal physiologic finding requiring no follow-up.[1][2]
    The SRU consensus update found exceptionally low cancer risk associated with simple
    adnexal cysts, and sets the premenopausal reporting threshold at 3 cm with follow-up
    imaging only above 5-7 cm.[2] The thin smooth wall, anechoic contents, and absent
    flow match the source paper's definition of a simple cyst.[1]",
  "Explanation_Other": [
    "Ovarian cystadenoma: the SRU update notes that simple cysts which GROW are likely
      cystadenomas — a stable 2.4-cm perfectly simple cyst in a 34-year-old is
      overwhelmingly physiologic, not neoplastic.[2] The source paper reserves concern for
      masses with solid or vascular components.[1]",
    "Endometrioma: endometriomas characteristically produce homogeneous low-level
      internal echoes with a 'ground glass' appearance on US[4]; the anechoic contents
      described in this stem exclude it.",
    "Corpus luteum cyst: the source paper describes the classic corpus luteum as a thick
      wall with crenulated inner margins, internal echoes, and intense peripheral colour
      Doppler flow — none present in this stem.[1]",
    "Cystic ovarian malignancy: O-RADS stratifies risk into six categories (0-5) spanning
      normal to high risk of malignancy, with the high-risk categories defined by solid
      components and other worrisome features[3]; none are described here, and the paper's
      central teaching is that incidental simple cystic masses are overwhelmingly benign.[1]"
  ],
  "Citations": {
    "Explanation": [
      ["follicle / no follow-up rule", [1, 2]],
      ["exceptionally low cancer risk; 3 cm report, 5-7 cm follow-up thresholds", [2]],
      ["simple-cyst definition matched by stem", [1]]
    ],
    "Explanation_Other": {
      "Ovarian cystadenoma": [["growth implies cystadenoma", [2]], ["solid/vascular concern", [1]]],
      "Endometrioma": [["ground-glass low-level echoes", [4]]],
      "Corpus luteum cyst": [["crenulated wall + peripheral flow", [1]]],
      "Cystic ovarian malignancy": [["O-RADS 0-5 risk stratification", [3]], ["benign predominance teaching", [1]]]
    }
  },
  "Sources": [
    {"id": 1, "title": "Wang PS et al. Benign-appearing Incidental Adnexal Cysts... Radiographics 2022;42(2):609-624 (source paper)",
     "url": "https://pubmed.ncbi.nlm.nih.gov/35061515", "retrieved": "2026-10-06",
     "quotes": ["Three major consensus articles on adnexal cystic masses were published between 2019 and 2020"]},
    {"id": 2, "title": "Levine D et al. Simple Adnexal Cysts: SRU Consensus Conference Update on Follow-up and Reporting. Radiology 2019;293(2):359-371",
     "url": "https://pubmed.ncbi.nlm.nih.gov/31549945", "retrieved": "2026-10-06",
     "quotes": ["This multidisciplinary consensus update aligns prior Society of Radiologists in Ultrasound (SRU) guidelines on simple adnexal cysts with recent large studies showing exceptionally low risk of cancer associated with simple adnexal cysts.",
                "For simple cysts in premenopausal women, these thresholds are 3 cm for reporting and greater than 5-7 cm for follow-up imaging.",
                "Simple cysts that grow are likely cystadenomas."]},
    {"id": 3, "title": "Andreotti RF et al. O-RADS US Risk Stratification and Management System: A Consensus Guideline from the ACR O-RADS Committee. Radiology 2020;294(1):168-185",
     "url": "https://pubmed.ncbi.nlm.nih.gov/31687921", "retrieved": "2026-10-06",
     "quotes": ["For risk stratification, the O-RADS US system recommends six categories (O-RADS 0-5), incorporating the range of normal to high risk of malignancy"]},
    {"id": 4, "title": "MDedge: Imaging the endometrioma and mature cystic teratoma (provenance: search snippet)",
     "url": "https://www.mdedge.com/content/imaging-endometrioma-and-mature-cystic-teratoma", "retrieved": "2026-10-06",
     "quotes": ["Endometriomas are common, typically benign, cysts that produce homogenous, low-level internal echoes and a \u201cground glass\u201d appearance on ultrasonography. No internal flow is apparent on color Doppler."]}
  ],
  "Citation_Audit": {
    "coverage": 0.78, "min_required": 0.6, "pass": true,
    "unverified_claims": 0, "dead_ends": []
  },
  "Notes": [
    "[2] and [3] are the primary consensus documents that source paper [1] itself
    summarises — independent corroboration of the thresholds, weighted equally; no conflict found.",
    "[4] cited from search snippet only, limited to the ground-glass feature it literally states."
  ]
}
```

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

## Sources

1. Wang PS et al. Benign-appearing Incidental Adnexal Cysts... Radiographics 2022;42(2):609-624
   — https://pubmed.ncbi.nlm.nih.gov/35061515
   > "Three major consensus articles on adnexal cystic masses were published between 2019 and 2020"
2. Levine D et al. Simple Adnexal Cysts: SRU Consensus Conference Update. Radiology 2019;293(2):359-371
   — https://pubmed.ncbi.nlm.nih.gov/31549945
   > "This multidisciplinary consensus update aligns prior ... guidelines on simple adnexal cysts with
     recent large studies showing exceptionally low risk of cancer associated with simple adnexal cysts."
   > "For simple cysts in premenopausal women, these thresholds are 3 cm for reporting and greater than
     5-7 cm for follow-up imaging."
   > "Simple cysts that grow are likely cystadenomas."
3. Andreotti RF et al. O-RADS US Risk Stratification and Management System. Radiology 2020;294(1):168-185
   — https://pubmed.ncbi.nlm.nih.gov/31687921
   > "For risk stratification, the O-RADS US system recommends six categories (O-RADS 0-5),
     incorporating the range of normal to high risk of malignancy"
4. MDedge: Imaging the endometrioma and mature cystic teratoma [search-snippet provenance]
   — https://www.mdedge.com/content/imaging-endometrioma-and-mature-cystic-teratoma
   > "Endometriomas are common, typically benign, cysts that produce homogenous, low-level
     internal echoes and a 'ground glass' appearance on ultrasonography. No internal flow is
     apparent on color Doppler."
```

### Why this passes Phase 9
- Every stem fact traces to [1] (simple-cyst definition; ≤3 cm follicle rule; CL appearance).
- Every explanation number/threshold (3 cm, 5-7 cm, O-RADS 0-5) now carries an external
  citation backed by a verbatim quote — the v1.0 claims that rested on paper + model
  memory alone are now externally grounded.
- Near-miss (corpus luteum) falsified by the stem's wall/Doppler description; endometrioma
  exclusion anchored to a sourced ground-glass feature, not model memory.
- Coverage: 7/9 explanation prose sentences carry `[n]` → 0.78 ≥ 0.6, `pass: true`.
- No `[unverified]` markers needed; the one snippet-only source is flagged honestly.
