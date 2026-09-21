# PHQ-9 authoring source: what was decided rather than transcribed

**Written 2026-09-20 alongside the source.** Every item text, instruction,
option label and severity threshold in `source.json` is transcribed from a
document recorded in its `provenance`. **The things below are not**, and they
are separated here so a reader can tell which is which.

---

## Rebuilt rather than corrected

An earlier six-language authoring source existed. **Nothing was taken from it.**
Its Spanish was a different translation from the distributed form — not in one
item but in nine items, the instruction, all four frequency labels, three of the
four difficulty labels and item 10 — and correcting it item by item would have
anchored the result to choices nobody verified. It was fluent and internally
consistent, which is what made it dangerous: it read like a finished
translation rather than an unverified one.

**Four languages were dropped.** `fr-FR`, `de-DE`, `hi-IN` and `pt-BR` had no
source document. A translation nobody can check against anything is not a
translation this source can carry.

**The language tag follows the document.** The earlier file declared `es-MX`;
the form held is *Spanish for the USA*, so the tag is `es-US`. The document
decides the tag, never the other way round.

## The total and the band are gated on completeness

**This is the decision that matters clinically.**

`missingPolicy` is `propagate` on both scores, and **on this engine that policy
is not consulted for an aggregate at all**. Measured: `sum` over nine items with
three answered returns the sum of those three, under every one of the three
policies. `docs/numeric-semantics.md` §2.1 records the underlying behaviour —
"aggregation already omits missing members unconditionally, at every policy" —
and notes that `missingPolicy` governs only the case where the whole expression
already produced null, which for a `sum` means nothing at all was answered.

**Left alone, that produces a clinically wrong output from a certified
instrument:** a PHQ-9 with three of nine items answered yields a total of those
three, and a band expression would classify it as mild depression. That is what
a clinician reads.

**So both scores are gated.** Each is wrapped in a completeness test —
`countAnswered` over the nine scored items must equal 9 — and emits `null`
otherwise. An incomplete PHQ-9 has **no total and no band**, rather than a
partial total that looks complete. Measured: 3 of 9 and 8 of 9 answered both
yield `total=null, band=null`; 9 of 9 yields both.

**Where the evidence of incompleteness actually lives, corrected 2026-09-20.**
An earlier draft of this file said `absentInputs` records which items were
demanded and missing. **That is wrong for a gated score.** Measured:

```
3 of 9 answered:
  phq9TotalScore    value=null  absentInputs=[]
  phq9SeverityBand  value=null  absentInputs=[]
```

The gate short-circuits on `countAnswered` before the `sum` is evaluated, so the
sum never demands the nine items and nothing records them as absent. **What does
carry it is the refusal**: `finalize` returns one `invalid_item_constraint` per
unanswered item, naming each by path. The evidence is specific and it is on the
refusal rather than on the score basis.

**And that produces a contradiction in the record format, recorded here because
it outlives this instrument.** Completeness is derived from `absentInputs` being
empty — never stored as a flag, precisely so a stored value cannot diverge from
its evidence. A gated score that withheld its value *because the instrument was
incomplete* now carries an empty `absentInputs`, so **by that rule its basis
reads as complete while its value is null.**

**No record reaches that state today**, and the reason is specific to this
instrument rather than general: all nine scored items are `required`, so
`finalize` refuses before a record exists. **The contradiction is in the format,
not in this content**, and it appears the moment an instrument with optional
items uses the same gate — at which point a record would be sealed carrying a
complete-looking basis for a score that was withheld.

It is recorded with the engine escalation below, because they are the same gap.

**No proration.** A PHQ-9 over fewer than nine items is not a PHQ-9 scaled up.

**The limitation is recorded rather than fixed**, because it is still true of
the engine for any instrument that does not write this gate by hand.

### Escalated as engine work: the engine cannot refuse to score

**Two findings, and they are one gap.** The engine has **no first-class notion
of an instrument refusing to score**. Authors express it with a hand-written
gate, and everything below follows from that.

**1. Refusal-to-score should be available without the gate.** An author who does
not write it gets a partial total silently, and the failure is invisible in the
output — the instrument produces a number, and the number is wrong in a way no
reader can see. `missingPolicy` has three values and none of them expresses
*refuse*: measured, all three produce the same partial sum for an aggregate.

**2. The gate is invisible to the completeness basis.** Because refusal is
expressed as an ordinary conditional rather than as a refusal, the score basis
cannot tell *withheld because incomplete* from *computed over everything it
demanded*. Both carry an empty `absentInputs`. **A mechanism the format cannot
see cannot be reasoned about by anything downstream**, including a verifier
reading a sealed record years later.

**What would close both:** a declared way for a score to require completeness of
a named input set, so that the engine — rather than an expression — withholds
the value, and the basis records why. Then completeness stays derivable from
evidence, and an author cannot forget it.

**An abstract statement of the underlying defect already existed** in
`docs/numeric-semantics.md` §2.1, including the measurement that aggregation
ignores the policy. It did not prevent this, because it was recorded as a
naming defect rather than followed through to what it means for a banded
instrument. **The PHQ-9 is the concrete case**: a severity classification over
three of nine answered items, which is the output a clinician reads.

## The band is an ordinal, not a string

`phq9SeverityBand` emits `0`–`4` rather than `"minimal"`–`"severe"`.

**Two reasons, and the second is the better one.**

The grammar forces it: `op: "null"` is typed `num?`, and branches must unify on
base type, so a string band cannot express "or nothing" and therefore cannot be
withheld on incompleteness. A band that cannot be withheld is the defect above.

**And a string band would be worse even if it were possible.** A string-valued
score writes display text into the record regardless of session language, so a
Spanish session would store an English band name. **The ordinal is
language-neutral**; the band names live in each language's score label, where
they are translated like everything else a person reads.

Mapping, from Table 4: `0` none-minimal · `1` mild · `2` moderate ·
`3` moderately severe · `4` severe.

**The first band keeps the manual's own name.** Table 4 says "None-minimal", not
"minimal", and shortening it would be an edit rather than a transcription.

**The top band is bounded at 27 rather than left open.** A total above 27 is
impossible, and an open `else` would classify an impossible value as severe. It
emits `null` instead, so an impossible total fails loudly rather than quietly
producing the most alarming band.

## Item 10 is unscored, and the exclusion is sourced

The manual states it directly on page 2: *"This single patient-rated difficulty
item is not used in calculating any PHQ score or diagnosis but rather represents
the patient's global impression of symptom-related impairment."*

The `helpText` uses the manual's own wording rather than a paraphrase. **The
Spanish `helpText` is a translation of that sentence and is not transcribed from
any document** — no Spanish-language manual was held. It is the one piece of
Spanish in this source that is not from the form, and it is marked here rather
than left to be assumed.

## Group labels carry the form's own headings

The forms have no group headings. Rather than invent section names — the earlier
file used "Symptom frequency over the past two weeks", which appears on no
document — each group's label is the form's own instruction text for that
block, transcribed. The instruction is not a separate display item: on the form
it is a heading above the table, not an item in it.

## What is owed

**The permission statement is not carried into definitions, and it should be.**
The manual states, page 8: *"All of the measures included in Table 1 are in the
public domain. No permission is required to reproduce, translate, display or
distribute."* The form carries only the second sentence; the public-domain
sentence is the half a legal reviewer needs and it is stronger.

**It is source-only in this work because carrying it further is a format
change.** The canonical schema's `assessment.authority` is
`additionalProperties: false` with `name` and `identifier`, and `authority`
passes straight through into every generated definition — so a permission field
would touch every definition and every record binding one. That is a real format
change and it should not ride along with content. Recorded as owed.

**Verification is owed.** Both languages carry `verifiedBy: { verified: false }`.
The transcription was produced by the engine track and nobody has read it back
against the documents. That is a normal state and it is stated rather than left
blank, because a transcription that verifies itself is a check grading its own
work.
