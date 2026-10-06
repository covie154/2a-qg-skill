# Question Artifact — Template

Every generated question is delivered as **one self-contained markdown file**. Copy this
skeleton exactly; replace the `<angle-bracket>` slots. Order of blocks is fixed:
question first, answer second, explanation third, sources fourth.

```markdown
# 2A SBA Question — <subspecialty>
**Source:** <FirstAuthor et al>. <Title>. <Journal>. <Year>;<vol>(<issue>):<pages>. DOI: <doi>
**Generated:** <DD-MM-YYYY> | **Verified:** <true | partial> | **Citation coverage:** <0.00>

---

## Question

<A 34-year-old premenopausal woman presents with <clinical history>. <Modality> showed
<findings> >

**What is the most likely diagnosis?**

A. <option A>
B. <option B>
C. <option C>
D. <option D>
E. <option E>

---

## Answer

**Correct option: <B>** — <Diagnosis name>

---

## Explanation

<2-5 sentences on why the correct answer is best, each load-bearing claim carrying a
bracketed citation id. Claims taken from the source document itself MUST cite [1].>

| Option | Verdict | Why |
|---|---|---|
| A. <name> | Incorrect | <falsified by which stated stem feature> [ids] |
| B. <name> | **Correct** | <decisive supporting feature> [ids] |
| C. <name> | Incorrect | <reason> [ids] |
| D. <name> | Incorrect | <reason> [ids] |
| E. <name> | Incorrect | <reason> [ids] |

---

## Sources

1. **<Source document — always citation [1]>** — <FirstAuthor et al>. <Title>. <Journal>
   <Year>;<vol>(<issue>):<pages>. DOI: doi.org/<doi> — <canonical URL or PMCID>
   — retrieved <DD-MM-YYYY>, <N> chars of body text extracted.
   > "<verbatim sentence from the source paper that carries the question's teaching point>"
2. <corroborating guideline / review> — <full citation> — <URL>, retrieved <date>
   > "<verbatim supporting quote>"
3. <further source> …

---

*Produced by the 2A SBA Question Generator skill. Answer keys and explanations are cited
to the source paper and to independently retrieved open-access evidence; the source
document is always [1].*
```

## Hard rules

1. **Citation [1] is the source document** — the paper the question was built from. It
   appears first in `Sources`, carries a verbatim quote of the teaching sentence, and is
   cited by every explanation claim drawn from that paper.
2. Numbers (age, size thresholds, percentages, intervals) appear only if traceable to a
   source in the block or marked `[unverified]`.
3. The `## Answer` and `## Explanation` blocks sit **below** the question so a reader can
   self-test before scrolling.
4. File name: `<YYYY-MM-DD>_<first-author-surname>_<subspecialty-slug>_qn<N>.md`
   (e.g. `2026-10-07_Wang_gu-adnexal_qn1.md`). Number questions within a run sequentially.
5. **Quiz variant:** when the user asks to be quizzed, emit a *separate* file containing
   only up to `## Question` — no Answer, no Explanation, no Sources with revealing names.
   Name it with a `_QUIZ` suffix (`..._qn1_QUIZ.md`). The answer key stays server-side in
   your working context until the user commits an answer.
