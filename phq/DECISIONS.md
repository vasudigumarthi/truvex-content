# PHQ authoring source: what was decided rather than transcribed

**Written 2026-10-07 alongside the source.** `source.json` declares `en-US` and
`it-IT`. Every item, heading, option label, the opening paragraph and the footer
are transcribed from the two forms committed in `sources/` (commit `0c54df3`).
**The things below are not**, and they are separated so a reader can tell which
is which.

**This is demonstration content.** It was transcribed from PDF by a language
model, without human review, and nobody has verified it. See
[Provenance](#provenance-what-is-recorded-and-what-could-not-be).

---

## The algorithms are the form's office coding, and three of them cannot always evaluate

Each form prints the office-coding lines (in English on both forms). Each is
authored as a boolean score named `officeCoding…` after the form's own
abbreviation, so a value reads as "the printed coding rule is met", not as a
diagnosis.

| Score | Printed rule | Status |
|---|---|---|
| `officeCodingSomDisSymptomCriterion` | Som Dis if at least 3 of #1a-m are "a lot" **and lack an adequate biol explanation** | **Partial, by decision.** Only the symptom count is authored |
| `officeCodingMajDepSyn` | #2a or b, and five or more of #2a-i at least "More than half the days" (count #2i if present at all) | Authored |
| `officeCodingOtherDepSyn` | #2a or b, and two, three or four of #2a-i at least "More than half the days" (count #2i if present at all) | Authored; reading below |
| `officeCodingPanSyn` | all of #3a-d YES and four or more of #4a-k YES | Authored |
| `officeCodingOtherAnxSyn` | #5a and three or more of #5b-g "More than half the days" | Authored; reading below |
| `officeCodingBulNer` | #6a, b and c and #8 all YES | Authored |
| `officeCodingBinEatDis` | the same, but #8 either NO **or left blank** | **Cannot evaluate the blank case** |
| `officeCodingAlcAbu` | any of #10a-e YES | Authored |

`depressiveItemsAtThreshold` (not emitted) is the count shared by the two
depression rules, authored once.

### Som Dis needs a judgment nobody enters

"Lack an adequate biological explanation" is a clinician judgment. It is not on
the form, and no input carries it. **Only the computable part is authored**, as
a criterion, never named after the disorder. Its label says the second criterion
is not evaluated. Two rejected options: authoring nothing, which loses a
computable criterion the form prints; and adding a clinician-entered item, which
is not a transcription and would seal a judgment nobody entered as a response.

### A blank answer cancels the algorithm that reads it

**On paper, a blank does not meet a criterion. In this engine, a blank that a
score reads means the score has no value.** Measured on the generated `en-US`
definition at capsule `babc9b6`:

- #2e blank, every other answer negative: `officeCodingMajDepSyn` and
  `officeCodingOtherDepSyn` have **no value**, `absentInputs=[q2e]`, although
  #2a and #2b are "Not at all" and the paper answer is plainly no.
- #1d blank, three of #1a-m "a lot": the Som Dis criterion has **no value**,
  `absentInputs=[q1d]`.
- #6a-c YES, #8 blank: **both** `officeCodingBinEatDis` and `officeCodingBulNer`
  have no value, `absentInputs=[q8]`. On paper Bul Ner is no and Bin Eat Dis is
  yes.

**This is escalated as platform work**, with these cases as its evidence:
`../ESCALATIONS.md`, escalation 1. **Nothing here is authored around it.**

**Under that rule a count answers only where the blank cannot change the result**,
and declines where it can. The first case becomes `false`, and the second
`true`. #2e blank with four others meeting stays at no value, correctly, because
the blank decides it. The rule is deliberately more conservative than the paper,
which assumes a complete form. **The #8 case is a conjunction, not a count**, and
escalation 1 does not change it.

### #8 "either NO or left blank"

The expression tests only `#8 = NO`, because a score may not test for a blank:
`isAnswered`, `coalesce` and `isNull` are refused inside scores
(`absence_operator_in_score`). With #8 visible and blank the score has no value,
and its basis names `q8`. **The gap is visible in every record, and the label
says so.**

**#8 is not required.** Making it required would change the instrument to fit
the engine, and the change would be invisible in the output. **Not authored as
"no YES among [#8]" either**: that reads a blank as not-YES, and it would look
faithful to anyone reviewing it later. Escalation 2 in `../ESCALATIONS.md`
records that the refusal is too broad.

### Two readings, stated as readings

- **Other Dep Syn, "#2a or b"**: the third line gives #2a or b no threshold of its
  own. Read with the second line's threshold, at least "More than half the
  days".
- **Other Anx Syn, "if #5a and answers to three or more of #5b-g are More than half
  the days"**: read as #5a also at "More than half the days", the top option of
  #5.

Both follow the grammar of the Maj Dep Syn line. **They are interpretations, not
transcriptions**, and a clinical reviewer should confirm them.

### Alc Abu is a count, so that a skipped #10 counts as none

"Any of #10a-e is YES" is authored as a count of at least 1. Rules withdraw #10
after #9 NO, and a count excludes withdrawn items, so a non-drinker gets
`false`. A plain `or` over the five items would give no value instead, because
withdrawn items read as null.

## Counting items under different criteria

The depression count includes #2i at "Several days" or above and its neighbours
at "More than half the days" or above. Each count is a `countWhere` over its
items, with a predicate that is an `or` of one test per member. The predicate
runs once per member with only that member visible, so each member's own test is
the one that decides. Measured: 2a-d at "More than half the days" with 2i at
"Several days" counts 5 (Maj Dep Syn); the same with 2i at "Not at all" counts 4
(Other Dep Syn).

## Item 1d is optional, and no rule skips it

The form prints a Sex field but never prints a rule skipping #1d, and inventing
one would be interpretation. **#1d is optional, because making it required
forces an answer that does not apply.** Today a blank #1d
cancels the Som Dis criterion. **Escalation 1 is its fix wherever #1d is not
decisive.** With three others "a lot" the criterion is met. With two, #1d
decides it, and the criterion correctly has no value.

**#1e has the same shape and is required**, because the form gives no basis for
treating it differently. Flagged for clinical review.

## Required items

The opening paragraph asks for every question to be answered "unless you are
requested to skip over a question", so items are required, and items withdrawn
by a skip rule are exempt. **Three are optional:** #1d (above); #8, which the
form itself treats as possibly blank; and #11, which is conditional ("If you
checked off any problems"), as in the PHQ-9 source.

## Skip rules

Four rules implement the printed skip instructions:

| Rule | When | Withdraws |
|---|---|---|
| `skipFrom3a` | #3a NO | #3b-d, all of #4 |
| `skipFrom5a` | #5a "Not at all" | #5b-g |
| `skipFrom6aOr6b` | #6a or #6b NO | #6c, all of #7, #8 |
| `skipFrom9` | #9 NO | all of #10 |

**The printed instructions themselves are not carried.** "If you checked NO, go
to question #5" tells a respondent how to navigate paper; on screen the rule
does it. This follows the check-mark instruction decision in the PHQ-9 source.
**The rules have no provenance slot in this schema**, so their source (the
highlighted instructions on pages 2 and 3 of each form) is recorded only here.

## Not carried

- **The Name / Age / Sex / Date line** (Nome / Età / Sesso / Data). Decided
  2026-10-07 (Vasu), for three reasons:
  - **No algorithm reads them.**
  - **The host already knows who the subject is.** Identity belongs to the
    deployment, not to the instrument.
  - **Carrying Sex would put a patient attribute into a record** that exists to
    carry responses and scores.

  **Consequence:** item 1d's applicability has no basis anywhere in the form as
  carried. That is the gap already recorded under
  [Item 1d](#item-1d-is-optional-and-no-rule-skips-it), not a new one.
- **The office-coding lines as text.** They are an administrative annotation,
  and their content is in the scores.
- **Underlining, page numbers and the line break in the Italian title.**

## Text layer versus image

Where they disagree, the printed glyph is followed and the site is recorded in
the language's provenance `notes`:

- **English:** #3a, #3c and #7c read `––` in the text layer and print one em dash,
  so they are carried as U+2014. #2f and #2h already carry U+2014 in the text
  layer.
- **Italian:** most apostrophes read U+201F in the text layer and print U+2019
  (for example *d’ansia*, *nell’altro*, *l’effetto*), so U+2019 is carried. #7c
  is handled as in English.

## Italian

**The Italian provenance entry cites the Italian document and its own digest.**
Scoring provenance cites the English document. The Italian form prints the same
office-coding lines in English at the same places, and scoring provenance has a
single document slot.

**Product text, not transcription: every Italian score label.** The form prints
no score labels in Italian. They are Truvex wording around the printed English
abbreviations, as are the English labels' surrounding words. **A qualified
bilingual clinician should review them before any clinical use.**

`identicalByDesign` declares one slot: the `NO` option label, which both forms
print identically.

## Provenance: what is recorded, and what could not be

Recorded per language: document, digest, retrieval, transcriber and method.
Recorded per score: document, locus and digest.

**What the schema could not record, in the session's own words:** our
provenance schema cannot record where an item was transcribed from within a
document, so transcription cannot be evidenced at the level our own instruments
require. Specifically:

- **No per-item locus.** Item locations are recoverable only from the item
  numbering, by reading the form.
- **No transcriber on scoring provenance.** It is stated in each entry's `notes`.
- **No provenance slot for rules.** It is recorded in the skip-rules table above.

Escalated as a schema change: `../ESCALATIONS.md`, escalation 3.

**The transcriber is a tool, and the record says so.**
`transcribedBy.role` is "language model, transcribed from PDF without human
review", `id` is the model identifier `claude-opus-5-5`, and `method` is
`textLayerAndImage`. **The id is not a quality-records identifier**, and none was
invented.

**`verifiedBy` is `{ verified: false }` with a note, not absent.** The schema
requires the field on a sourced language, and `false` is its value for "nobody
verified this". **No human verifier is recorded.**

**Publisher: what each document itself states.** Neither form names a
publisher. Each prints a developer statement, and `publisher` records that
statement in the document's own language, quoted from the footer:

- **English:** "Developed by Drs. Robert L. Spitzer, Janet B.W. Williams, Kurt
  Kroenke and colleagues, with an educational grant from Pfizer Inc."
- **Italian:** "Elaborato dai dottori Robert L. Spitzer, Janet B.W. Williams, Kurt
  Kroenke e colleghi, con un finanziamento da parte della Pfizer Inc."

**Not known, and stated as unknown: the retrieval location.** It was not
recorded when the documents were placed. `retrievedFrom` says so and names the
committed file; `retrievedAt` is the date they were placed (2026-10-07).

### The model name in provenance collides with the capsule repository's rule

**Recorded 2026-10-07.** `transcribedBy` names the model, because that is the
truth about who transcribed this. The authoring source's `provenance` is carried
into each generated package's provenance document, restricted to that package's
language.

**If content authored this way is ever generated into packages kept in the
capsule repository, the model name in the package provenance collides with that
repository's rule against naming AI tooling anywhere.** That rule does not
govern this repository, where the name is the honest record.

**Resolve it before any such package is committed there.** Do not resolve it by
removing the name: that would make the provenance false. Keep such packages out
of that repository, or settle how the rule treats provenance that must name its
tool.

**Delivery content needs a real transcriber and a real verifier.** This source is
for demonstration and platform testing only.

## Validation

`check-source.mjs`: **VALID** at capsule `babc9b6`, run from a clean clone built
in scratch, not from the live working tree. The generator also produces both
definitions at that commit.
