# PHQ-SADS authoring source: what was decided rather than transcribed

**Written 2026-10-07 alongside the source.** `source.json` declares `en-US`
only. Every item, heading, option label and score, the opening paragraph and the
footer are transcribed from `sources/PHQ-SADS-English_1.pdf` (commit `0c54df3`).
**The things below are not.**

**This is demonstration content.** It was transcribed from PDF by a language
model, without human review, and nobody has verified it.

---

## Deviation: the printed skip at C a is not authored

**Recorded 2026-10-07, decided by Vasu. This is a deviation from the printed
form, and it is deliberate.**

**What the form prints.** Under C a: *"If you checked “NO”, go to question E."*
Section D, which follows, is the PHQ-9. **Read literally, the instruction skips
the whole PHQ-9 for every respondent who has not had an anxiety attack.**

**What the literal reading produces.** Measured at capsule `babc9b6` on a
scratch variant carrying that rule, with every D item at "Nearly every day" and
C a NO:

```
as authored (no rule):        phq9Score=27
with "go to question E":      phq9Score=0     absentInputs=[]
```

**A respondent with no panic attack gets a depression score of 0**, with a basis
that reads complete. That reads as no depression, and it could be
catastrophically wrong: the most severe possible PHQ-9 is reported as the least
severe. Recorded as platform work in `../ESCALATIONS.md`, escalation 1, under "Sums".

**Why the deviation is unauthored rather than corrected.** Every option deviates
from the printed form, so the question is which deviation is safe:

- **Authored as printed:** the zero above. **Unsafe.**
- **Unauthored, as here:** C b-e are asked of everyone, including respondents the
  form would have skipped past. **That costs a minute and harms nobody.**
- **Changed to "go to D":** that corrects the instrument on our own authority,
  **which we do not do**, however likely it is to be what was meant.

**The resolution belongs to the instrument owner**, not to Truvex: either the
form is corrected at its source, or the owner confirms the intended target and
this source follows.

**C b-e are required**, like the rest of the form. The opening paragraph asks for
every question to be answered, and this form, unlike the PHQ, prints no "unless
you are requested to skip".

## Scores

| Score | Printed | Authored |
|---|---|---|
| `phq15Score` | "PHQ-15 Score = ___ + ___" under A | sum of A1-15 option scores |
| `gad7Score` | "GAD-7 Score = ___ + ___ + ___" under B | sum of B1-7 |
| `phq9Score` | "PHQ-9 Score = ___ + ___ + ___" under D | sum of D1-9 |

**The option scores are transcribed**, from the column headings: (0) to (2) for A,
and (0) to (3) for B and D. The labels are the printed score names.

**The printed form is a sum of column subtotals**, written as one sum over the
items. That is the same arithmetic.

**No severity bands are authored.** None are printed on this form, and authoring
them from another document or from memory would not be a transcription.

**A sum withholds when any item is blank.** That is correct for a total, and
escalation 1 keeps it: a sum must state a magnitude and cannot bound one. Only
threshold counts answer with blank members, and only where the blanks cannot
change the result.

**Section C has no score.** None is printed.

## A6 is optional, and the PHQ-15 total withholds when it is blank

A6, "Menstrual cramps or other problems with your periods", does not apply to
every respondent. The form prints no rule for it, so none is authored.
**It is optional, because making it required forces an answer that does not
apply.**

The consequence is real and stated: **a respondent who leaves A6 blank gets no
PHQ-15 total.** Measured: `phq15Score=null`, `absentInputs=[a6]`. No published
missing-data rule for the PHQ-15 is held, so the engine default (withhold)
stands. **If a published rule exists, it is the fix**, through the cited
declaration in the capsule's numeric semantics §2.3.

A7 has the same shape and is required, because the form gives no basis for
treating it differently. Flagged for clinical review.

**E is optional**, being conditional ("If you checked off any problems").

## Carried as printed, errors included

- **B1:** "Feeling nervous anxiety or on edge", with no commas.
- **D9:** "Thoughts that you would be beter off dead of or hurting yourself in
  some way". "beter" and "dead of or" are printed that way.
- **The opening paragraph** ends "to the best of your ability", with no full stop.

**Correcting any of these would be an edit.** They are flagged for a reviewer,
because they are what a respondent reads.

**Dashes:** D6 prints an em dash and D8 an en dash, and both are kept. C a and C
c read U+23AF in the text layer and print an em dash, so they are carried as
U+2014.

## Not carried

Dot leaders, the score boxes and blanks, the skip instruction text, and the
footer's second printing (it appears at the top and the foot of page 2). The
footer is carried once, as `instrument.attribution`.

## Provenance

Identical in shape and gaps to the PHQ source: see `../phq/DECISIONS.md`,
"Provenance: what is recorded, and what could not be".

- **The transcriber is a tool:** id `claude-opus-5-5`.
- `verifiedBy` is `{ verified: false }`.
- There is no per-item locus and no rules slot, and the scoring transcriber is
  in `notes`.
- **Publisher:** no publisher is named. `publisher` records what the document
  states, its developer line: "Developed by Drs. Robert L. Spitzer, Janet B.W.
  Williams, Kurt Kroenke and colleagues, with an educational grant from Pfizer
  Inc." The retrieval location is unknown and stated as such.

**The model name collides with the capsule repository's rule** if a package
generated from this source is ever kept there. The full note is in
`../phq/DECISIONS.md`, "The model name in provenance collides with the capsule
repository's rule", and it applies here unchanged.

**Delivery content needs a real transcriber and a real verifier.**

## Validation

`check-source.mjs`: **VALID** at capsule `babc9b6`, from a clean clone built in
scratch. The generator also produces the definition at that commit.
