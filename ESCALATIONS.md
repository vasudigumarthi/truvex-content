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

## 1. A threshold count bounds itself and answers only when the bounds agree; a sum withholds

**Decided 2026-10-07 (Vasu). This replaces two earlier escalations**:

- the first escalation 1, "a count treats a blank as not meeting the criterion";
- the first escalation 4, "an aggregate with zero contributing members never
  returns a value".

**Neither was right as written**, and the conflict between them was the symptom.
Their evidence is kept below.

**The capsule thread implements this in the engine. Content cannot express it,
and should not try.** Nothing in the authoring source or the expression grammar
can tell a decisive blank from an irrelevant one, and no authoring declaration
is to be added for it.

### Why "blank does not match" was wrong

Take the depression count: eight items answered, four meet the threshold, #2e
blank, threshold five. "Blank does not match" reports `false`. **But a match on
#2e would have made it five.** The blank is decisive, and that rule answers
anyway. **It is the same class of error as a PHQ-9 of 27 becoming 0, only
quieter.**

### The rule for a threshold count

**Classify every member:**

| Member | Class |
|---|---|
| Answered and matching | definite match |
| Answered and not matching | definite non-match |
| Shown but blank | unknown |
| Excluded because an **answered** input made it inapplicable | definite non-match: the respondent told us it does not apply |
| Excluded with no answered cause, including a section never performed and a gate left blank | unknown: nobody asked, so anything is possible |

**Then bound the count:**

- **minimum** = definite matches;
- **maximum** = definite matches + unknowns.

**Answer only when the bounds agree:**

| Bounds | Result |
|---|---|
| minimum meets the threshold | `true` |
| maximum fails the threshold | `false` |
| otherwise | **no value**, because the unknowns decide it |

**The result records the three numbers:** definite matches, definite non-matches
and unknowns. A reader can then see which case produced the answer instead of
inferring it.

**The test of "answered cause" is derived, not declared.** The engine knows which
rule excluded a member and whether that rule's own condition was answered.

### Sums

**Unchanged in principle.** A sum must state a magnitude and cannot bound one:

- **any unknown member means the sum has no value;**
- **a sum over zero contributing members always has no value**, whatever emptied
  it. With no values there is no magnitude, and 0 states something nobody
  produced.

D1 (record-3 break plan §43) covers a section excluded by a *declared gate*. **The
zero-member case is broader:** ordinary skip rules can empty a total with no
gate involved. D1's other half stands unchanged: a *partially* excluded section
withholds only when the author declares it a gate.

### Strictly more conservative than the printed algorithms

**This rule is strictly more conservative than the printed algorithms, which
assume a complete form.** Where the paper implies an answer from a form with a
decisive blank, the engine declines to answer. **That is the correct direction to
be wrong in:** declining is visible in the record; a wrong answer is not.

### Every case we have

Results under the rule are derived by applying it. "Today" is measured at capsule
`babc9b6` on the generated `en-US` PHQ definition, except where marked.

**Alc Abu** (count of #10a-e YES, at least 1):

| Case | min / max | Under the rule | Paper | Today |
|---|---|---|---|---|
| #9 NO, #10 excluded by that answer | 0 / 0 | `false` | false | `false` |
| #9 YES, #10a-e all NO | 0 / 0 | `false` | false | `false` |
| #9 blank, #10 shown and blank | 0 / 5 | **no value** | (assumes complete) | no value, `absentInputs=[q10a..q10e]` |
| #9 blank, #10 hidden | 0 / 5 | **no value** | — | not reachable: a rule whose condition is unanswered does not fire |

**The earlier conflict disappears instead of needing a decision.** A blank gate's
members are unknown whether the engine shows them or hides them, so the two
rows give the same answer.

**Maj Dep Syn** (count of #2a-i at threshold, at least 5) and **Other Dep Syn**
(2 to 4), with #2e blank:

| Others meeting | min / max | Maj Dep Syn | Other Dep Syn | Today (both) |
|---|---|---|---|---|
| 5 of 8 | 5 / 6 | `true` | `false` | no value, `absentInputs=[q2e]` |
| 4 of 8 | 4 / 5 | **no value**: the blank decides it | **no value** | no value |
| 3 of 8 | 3 / 4 | `false` | `true` | no value |
| 0 of 8, #2a and #2b "Not at all" | 0 / 1 | `false` | `false` | no value |

The 4-of-8 row is the one the first escalation 1 would have answered `false`.

**Other Dep Syn's threshold is a range.** The rule then reads: `true` when every
count in [min, max] lies inside 2 to 4, `false` when none does, no value
otherwise. **The PHQ source also computes the count as its own score
(`depressiveItemsAtThreshold`) and compares it in two others through
`scoreRef`**, so the bounds must survive a `scoreRef` and the comparison applied
to it. How is the capsule thread's call. The PHQ needs it.

**Som Dis symptom criterion** (count of #1a-m "a lot", at least 3), with #1d
blank:

| Case | min / max | Under the rule | Today |
|---|---|---|---|
| three others "a lot" | 3 / 4 | `true` | no value, `absentInputs=[q1d]` |
| two others "a lot" | 2 / 3 | **no value**: #1d decides it | no value |

**Pan Syn and Other Anx Syn:** unaffected. The gating item's definite `false`
already decides each `and`.

**Sums, PHQ-SADS:** under C a the form prints "If you checked NO, go to question
E", which taken literally skips section D, the PHQ-9. On a scratch variant
carrying that rule, with every D item at "Nearly every day" and C a NO:

```
without the rule:   phq9Score=27
with the rule:      phq9Score=0    absentInputs=[]
```

**The most severe possible total is reported as the least severe, with a basis
that reads complete.** It follows from two settled positions applied together: a
withdrawn item is not demanded (numeric semantics §2.2), and `sum` over an empty
set is 0. **Under the rule, it has no value.** The source does not author the
literal skip; that is a recorded deviation in `phq-sads/DECISIONS.md`.

**Also measured, A6 blank:** `phq15Score` has no value, `absentInputs=[a6]`. That
is correct for a sum, and unchanged.

### What the rule does not cover

**Bul Ner and Bin Eat Dis are conjunctions, not counts.** With #6a-c YES and #8
blank, both have no value today (`absentInputs=[q8]`). Three-valued logic over an
unknown #8 gives the same result, which is correct for Bul Ner. **Bin Eat Dis's
"either NO or left blank" names the blank as a condition. That is escalation 2,
and this rule does not reach it.**

### What it touches beyond the evaluator

- **The record format.** Completeness is derived today from an empty
  `absentInputs`, and a non-empty list means no value. A count that answers with
  unknown members needs its three numbers recorded, or a record would carry a
  value beside a non-empty absent list and contradict its own rule. That is an
  `assessment-record` change.
- **The vector generator.** It refuses a comparison inside a `countWhere`
  predicate (`vector_path_unmeasurable`), and its recorded trigger to revisit is
  now met. The PHQ source uses `in`, not a comparison, and generates today.
  Vectors were not attempted.

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

## 4. Merged into escalation 1

**Replaced 2026-10-07.** The rule raised here (an aggregate with zero
contributing members never returns a value, with a refinement for counts) and
its evidence, `phq9Score=0` and Alc Abu, are now part of escalation 1's single
rule. **The number is kept so that references to it still resolve.**
