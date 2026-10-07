# Escalations to the platform, from content authoring

**Raised 2026-10-07 while authoring `phq/` and `phq-sads/`.** These are capsule
work: schema, engine, generator and record format. They are written here
because this thread does not touch that repository.

> **Before carrying any of this into truvex-capsule:** that repository forbids
> instrument names anywhere, commit messages included. Every instrument named
> below must become a generic description there ("a screening form whose office
> coding counts items against a threshold"). The evidence stays here, by
> reference.

All measurements below are on definitions generated at capsule `babc9b6`, from a
clean clone built in scratch, evaluated with `resolveRules` and
`evaluateScores`.

---

## 1. A count of items meeting a criterion must treat a blank as not meeting it

**The finding.** The forms' office-coding algorithms treat a blank as a definite
"does not meet the criterion". The engine treats any absent input a score reads
as "cannot compute". **Both are right in their own place, and only one is
applied.**

- **For a sum**, withholding is correct: a missing item makes the total wrong.
- **For a count of items meeting a criterion**, an unanswered item simply does not
  meet it, and the count is still meaningful. That is how the paper algorithm
  works and how a human office coder applies it.

The engine collapses the two cases.

**Required behaviour.** A count-with-threshold tolerates absence, treating an
unanswered member as not matching, and **reports alongside its result how many
members were unanswered**, so a reader knows how much of the form was blank.
Sums keep withholding.

**Evidence** (PHQ, `phq/source.json`):

| Responses | Paper answer | Engine |
|---|---|---|
| #2e blank, all else negative, #2a and #2b "Not at all" | Maj Dep Syn no, Other Dep Syn no | both **no value**, `absentInputs=[q2e]` |
| #1d blank, three of #1a-m "a lot" | Som Dis symptom criterion met | **no value**, `absentInputs=[q1d]` |
| #6a-c YES, #8 blank | Bul Ner no, Bin Eat Dis yes | both **no value**, `absentInputs=[q8]` |

**What it touches beyond the evaluator:**

- **The record format.** Completeness is derived from an empty `absentInputs`, and
  a non-empty list means no value. A tolerant count needs its own field for its
  unanswered members, or a record would carry a value beside a non-empty absent
  list and contradict its own rule. That is an `assessment-record` change.
- **The vector generator.** It refuses a comparison inside a `countWhere`
  predicate (`vector_path_unmeasurable`). Its recorded trigger, "when an
  instrument needs a comparison inside a countWhere predicate", is now met. The
  sources here use `in`, not a comparison, and generate today; vectors were not
  attempted.

**Do not author around it in content.** The PHQ source does not.

## 2. `absence_operator_in_score` is too broad

It refuses `isAnswered`, `coalesce` and `isNull` anywhere inside a score. **A real
instrument tests for a blank:** the PHQ's Bin Eat Dis is "#8 either NO or left
blank". The refusal exists to close hand-written proration and zero
substitution, and it does that. It also forbids a published rule that names a
blank as a condition.

**Wanted:** a way to express "this item was left blank" as a condition in a
score, which is not a route to substituting a value for it. Until then, the PHQ
authors only "#8 = NO", records that the blank case cannot evaluate, and does
not make #8 required.

## 3. Instance: the provenance schema cannot evidence a transcription at the level our instruments require

**Finding, stated plainly:** our provenance schema cannot record where an item
was transcribed from within a document, so transcription cannot be evidenced at
the level our own instruments require. **In a product whose pitch is provenance,
that is a real gap.** The content brief claimed per-item provenance, and the
schema cannot hold it.

**What is missing** (authoring source 2.0, `provenance`):

- **A per-item locus.** `languages.<tag>` records document, digest, transcriber
  and method, with no locus and no per-item entries (`additionalProperties:
  false`). Only free `notes` remain, capped at 1024 characters.
- **A transcriber on scoring provenance.** `scoring[]` has `transcribedFrom`
  with no `transcribedBy`.
- **A provenance slot for rules.** None exists. A skip rule's source cannot be
  recorded.

**Scoped change, estimated by reading the code (not measured):**

- **Per-text location:** `languages.<tag>.loci` maps each piece of displayed text
  (addressed as `identicalByDesign` addresses it) to a locus. A `productText`
  entry, with a reason, declares Truvex wording. For a sourced language, the
  generator refuses any displayed text that has neither. This makes "transcribed
  here, Truvex wording there" structural rather than prose.
- **Scoring:** `scoring[].transcribedBy` becomes required. No in-repo sample
  carries scoring provenance, so nothing breaks.
- **Rules:** a new `rules[]` with `{target, transcribedFrom, transcribedBy,
  notes?}`, shared across languages like scoring.
- **Coverage:** when provenance is present, every score and rule has an entry and
  every target exists, matching the existing "partial provenance is refused"
  rule.
- **Versions:** authoring source 2.0 → 2.1, and content provenance 1.0.0 → 1.1.0.
  Both patterns already admit these. The change is additive to the package
  provenance document, which is signed through the content manifest's
  `evidence`. **It is not a record-format change.**
- **Footprint:**
  - `tools/build/derive-schemas.mjs`, which authors `PROVENANCE` and derives
    content provenance from it
  - two regenerated schemas and their compiled validators
  - `format-versions.ts`
  - `generate.ts` and `generate-refusals.ts` (about 4 new refusal codes)
  - the source-schema and generator tests, with one planted-violation probe per
    new refusal
  - record 3.0.0 plan §25
  - **About 10 hand-edited files and roughly 300–450 lines including tests.** No
    sample regenerates: all five are `sourced: false`.

**Owner:** the capsule thread (schema and generator code).
**Trigger:** the first instrument authored for **delivery** rather than
demonstration.
**Until then:** the PHQ and PHQ-SADS sources record what the schema allows, at
document level, and their DECISIONS files state what could not be recorded.

## 4. An aggregate with zero contributing members never returns a value, whatever emptied it

**This is the same defect as D1, reached by a different path, and the rule needs
stating more broadly than D1 states it.**

**D1** (record-3 break plan §43, decided 2026-10-07) says: a section excluded by
a *declared gate* never returns a score; it is withheld, with a named reason
identifying the gate. Undeclared groups behave as today.

**The broader path:** ordinary skip rules can empty a total with no gate
involved. Today that total returns 0.

**Evidence.** The PHQ-SADS prints, under C a: "If you checked NO, go to question
E". Taken literally, that skips section D, the PHQ-9. On a scratch variant
carrying that rule, with every D item answered "Nearly every day" and C a NO:

```
without the rule:   phq9Score=27
with the rule:      phq9Score=0    absentInputs=[]
```

**The most severe possible total is reported as the least severe, and the basis
reads complete.** No gate is declared anywhere, so D1 does not reach it. It
follows from two settled positions applied together:

- a withdrawn item is not demanded (numeric semantics §2.2);
- `sum` over an empty set is 0 (§2.3, empty-set rules).

**The rule to carry: an aggregate with zero contributing members never returns a
value, whatever emptied it** — a declared gate, skip rules, or anything else.
It is withheld, with a reason.

**This does not conflict with D1's other half.** A *partially* excluded section
withholds only when the author declares it a gate. Branching that skips some
members leaves the score valid. **That decision stands.** The broader rule
covers only the case where *no* member contributes.

**Not yet settled: do counts fall under it?** Derived by reading the PHQ source,
not measured. Under the rule as stated, `count` and `countWhere` with zero
contributing members would also return no value. In the PHQ that changes one
algorithm:

- **Alc Abu** is a count of #10a-e YES, at least 1. After #9 NO, every member is
  withdrawn. Today the count is 0 and the result `false`, which is the paper's
  answer. Under the broad rule it would have no value for every non-drinker.
- **Pan Syn and Other Anx Syn** are unaffected. Their counts can also be emptied
  by a skip, but each sits in an `and` beside the gating item, and that item's
  definite `false` still decides.

**Questions for the capsule thread:**

- Is "zero members, no value" right for counts as well as for totals?
- Should a count emptied by logic say so through the same reason, so content
  like Alc Abu can rely on it?

**This is the same territory as escalation 1**, which already asks counts to
treat absence differently from sums.

**Content is not affected today.** The PHQ-SADS source does not author the
literal skip, as a recorded deviation (`phq-sads/DECISIONS.md`). **Any
instrument whose skip logic can withdraw every member of a total is exposed
until the rule lands.**
