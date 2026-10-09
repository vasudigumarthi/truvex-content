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

## An incomplete PHQ-9 has no total and no band

**This is the decision that matters clinically, and since 2026-10-09 the engine
holds it rather than an expression in this source.**

**What happens, measured 2026-10-09** on this source's own expressions, with the
capsule's generator and engine at `dfa8e13`, content model 2.0:

```
items 1 to 6 answered, 7 to 9 blank:
  phq9TotalScore    value=null  undefinedReason=absentInput  absent: items 7, 8, 9
  phq9SeverityBand  value=null  undefinedReason=absentInput  absent: items 7, 8, 9
nothing answered:
  both              value=null  undefinedReason=absentInput  absent: items 1 to 9
all nine answered:
  phq9TotalScore    the sum      phq9SeverityBand  its band
```

A score that touches an absent input has no total and lists what was absent,
and a score read from it, the band, inherits both. **So a partial PHQ-9 can
never show a partial total or a classification**, and the evidence of why is on
the score basis, item by item. Both cases are vector cases (below), so an engine
that put a partial total on an incomplete form would refuse to start with this
content.

**No proration.** A PHQ-9 over fewer than nine items is not a PHQ-9 scaled up.
The source declares no published missing-data rule, which is the one thing that
could let content model 2.0 emit a total over absent inputs.

### History: the hand-written gate, 2026-09-20 to 2026-10-09

**On the engine of 2026-09-20 (content model 1.x)**, `sum` over nine items with
three answered returned the sum of those three, under every value of
`missingPolicy`, and a band would have classified it as mild depression. So both
scores were wrapped in a completeness test, `countAnswered` over the nine items
equal to 9, else a bare `null`. That worked, and it left two gaps, escalated as
engine work at the time: the engine could not refuse to score on its own, and a
gated score withheld its value with an empty `absentInputs`, so its basis read as
complete while its value was null.

**Both closed in the engine on 2026-10-05** (capsule `311b4d0`, content model
2.0.0, "the missing-input repair"): an absent input withholds the total and
records `absentInput` with the list, a score reading another inherits it, and
`missingPolicy` was removed. Measured on 2026-10-09 (above): the absent items are
now named on both scores.

**The gate was deleted on 2026-10-09, not rewritten.** It is no longer
load-bearing, and its bare `null` is refused by content model 2.0
(`null_literal_in_score`). The band's last arm, "if the total is at most 27 then 4,
else `null`", became plain 4 at the same time: a total of nine items scored 0 to
3 cannot exceed 27, so the `null` was unreachable and is refused for the same
reason. No reachable result changed.

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

Mapping, from Table 4: `0` None-minimal (0–4) · `1` Mild (5–9) · `2` Moderate
(10–14) · `3` Moderately Severe (15–19) · `4` Severe (20–27).

**The first band keeps the manual's own name.** Table 4 says "None-minimal", not
"minimal", and shortening it would be an edit rather than a transcription.

**The English label matches Table 4's casing exactly**, corrected 2026-10-03: it
previously lowercased every band name. The Spanish label is unchanged; see
[Spanish text with no published source](#spanish-text-with-no-published-source).

**The top band is bounded at 27 rather than left open.** A total above 27 is
impossible, and an open `else` would classify an impossible value as severe. It
emits `null` instead, so an impossible total fails loudly rather than quietly
producing the most alarming band.

### An instruction to change the bands, checked against the document and withdrawn

**Recorded 2026-10-03.** An instruction asked for the manual's labels as single
words — dropping "None-" from band 0, to be marked as needing confirmation
against Table 4 — and for a recorded deviation, on the understanding that the
published first band runs 1 to 4 where this source covers 0 to 4.

**The instruction came from a recollection of the published table, not from the
document.** Reading Table 4 in the held manual (digest `sha256:6ed2f00f…`,
page 7) settled it: the first band is printed "0 – 4" and named "None-minimal".
The stored labels and ranges already matched. **No confirmation item and no
deviation is recorded, because there is none.** Only the casing above changed.

## Item 10 is unscored, and the exclusion is sourced

The manual states it directly on page 2: *"This single patient-rated difficulty
item is not used in calculating any PHQ score or diagnosis but rather represents
the patient's global impression of symptom-related impairment."*

The `helpText` uses the manual's own wording rather than a paraphrase. **The
Spanish `helpText` is a translation of that sentence and is not transcribed from
any document**; see
[Spanish text with no published source](#spanish-text-with-no-published-source).

## Spanish text with no published source

**Recorded 2026-10-03. Agreed in conversation on 2026-09-21 and not written down
until now**, which is the failure this project keeps finding in code, here in
its own decision record.

**The Spanish severity band labels and the Spanish item 10 `helpText` have no
published source.** The Spanish form prints neither, and the instruction manual
is English only. **They are Truvex translations, recorded as product text rather
than as transcription**, and a qualified bilingual clinician reviews them before
any clinical use.

**The English equivalents are not product text.** The band names are transcribed
from Table 4 and the `helpText` from the manual's page 2. **The asymmetry is the
point:** the same slot is a transcription in one language and Truvex's own
wording in the other, and a reviewer must not read the Spanish with the
authority the English carries.

**Corrected in the same entry:** this file previously called the Spanish
`helpText` "the one piece of Spanish in this source that is not from the form".
That was false. The band labels are not from the form either, and neither are
the two score labels' surrounding wording — *"PHQ-9 total score"*, *"PHQ-9
depression severity band (…)"* and their Spanish counterparts — which are
Truvex wording in both languages, around band names that are transcribed in
English only.

## Group labels carry the form's own headings

The forms have no group headings. Rather than invent section names — the earlier
file used "Symptom frequency over the past two weeks", which appears on no
document — each group's label is the form's own instruction text for that
block, transcribed. The instruction is not a separate display item: on the form
it is a heading above the table, not an item in it.

### Item 10's text rendered twice, and content was not at fault

**Measured 2026-10-03**, by rendering each generated definition through the
platform's browser renderer into a DOM: item 10's text appears twice in both
languages, once as the `functionalImpact` group's heading and once as the
item's prompt.

**The cause is three facts together:** the schema requires every group to carry
a label; this group holds one item whose prompt is that same text; and the
printed form has no heading for that block. The content states what the form
states.

**Rejected: inventing a heading for this group.** It would put Truvex words
into a clinical form where the published form has none. The decision above
stands.

**Closed in the platform** by `c2a9c80` and amended by `9efc20c`, both in
`truvex-capsule`: the renderer omits a group heading when the group holds
exactly one item, has no child groups, and that item's prompt is identical to
the label. It is instrument-agnostic and needed nothing from content. Measured
after `9efc20c`: item 10's text renders once in both languages.

## The printed footer is carried verbatim, in the attribution slot

Each form prints an attribution and permission statement at the foot of the
page. **Since 2026-10-04 it is carried as `instrument.attribution`**, one plain
string per language, exactly as printed, and each generated definition carries
its own language's text as `assessment.attribution`:

- English: *"Developed by Drs. Robert L. Spitzer, Janet B.W. Williams, Kurt
  Kroenke and colleagues, with an educational grant from Pfizer Inc. No
  permission required to reproduce, translate, display or distribute."*
- Spanish: *"Elaborado por los doctores Robert L. Spitzer, Janet B.W. Williams,
  Kurt Kroenke y colegas, mediante una subvención educativa otorgada por Pfizer
  Inc. No se requiere permiso para reproducir, traducir, presentar o
  distribuir."*

The line break each form prints mid-sentence is layout and is not carried. **One
string, not split into attribution and permission**: splitting it would be
interpretation, and the decision was to store what the form prints.

**It renders where the form prints it.** Measured 2026-10-04 on both generated
definitions: `h1 > symptomItems > functionalImpact > footer`, the footer's text
equal to the stored attribution. It maps to FHIR `Questionnaire.copyright`, and
an ODM export reports it lost, ODM having no element for it.

**The definitions now declare content model revision 1.1.0**, the revision that
introduced the field, stamped by the generator from the field's presence. A
deployment whose engine supports only 1.0.0 refuses them by name.

### History: a display item, 2026-10-03 to 2026-10-04

The footer was first carried as an ungrouped display item, `printedFooter`,
because the format had no slot for it. Ungrouped items render before groups, so
it rendered near the top, under the title — **a recorded deviation from the
printed layout**, now resolved. Placing it in `functionalImpact` after item 10
was rejected then: it would have needed the renderer's single-item heading rule
widened to fit content added in the same session. The text moved from the
display item to the slot byte for byte, checked in both languages.

### Still a gap: `authority.name` is not localizable

`instrument.authority.name` is a single string and cannot hold a Spanish
authority. The attribution slot carries the printed text in each language, which
is what was needed; **the authority field itself is unchanged**, keeps the
English attribution it held, and localizing it would be its own format change.

## Item 9 raises no alert

**Recorded 2026-10-03 as a prototype-stage decision, for review before any
clinical use.**

Capsule captures the response to item 9 and scores it like any other item. **It
raises no alert, flag or notification on any answer.** Any protocol for a
positive answer — who is told, how fast, what follows — belongs to the
deploying party, and is theirs to define and operate.

**This matches how the paper instrument is administered.** The printed form
carries no instruction for a positive answer; the manual (page 2) says a final
decision about the risk of self-harm requires a clinical interview, and points
to a separate follow-up screener. Both sit with the clinician, not the form.

**The same statement is in the platform's integration guide**, so a deploying
party cannot assume the product alerts.

## The check-mark instruction is not carried

Each form prints a marking instruction under the first block's heading:
*(Use "✔" to indicate your answer)* and *(Marque con un "…" para indicar su
respuesta)*, where the Spanish form's mark is a symbol-font glyph its text layer
does not carry. **It is omitted deliberately.** It tells a respondent how to mark
paper, which does not apply on screen, where the response control is the
instruction. Recorded so the absence is a decision rather than a silence.

## The manual's retrieval date

**The instruction manual was retrieved 2026-09-20.** Basis: the held file's
timestamp is 2026-09-20 21:54, beside the two forms' 21:45 and 21:46, and its
sha256 matches the digest recorded in `provenance.scoring`. The date is recorded
here rather than in the source because `transcribedFrom` carries
`document`, `locus` and `digest` by design: a citation is not a transcription,
and the digest is what makes the citation checkable. That shape is intended, not
incomplete.

## What is owed

**The public-domain statement is not carried into definitions, and it should
be.** The manual states, page 8: *"All of the measures included in Table 1 are in
the public domain. No permission is required to reproduce, translate, display or
distribute."* The form carries only the second sentence, and since 2026-10-03
that sentence reaches definitions as part of the printed footer. The
public-domain sentence is the half a legal reviewer needs and it is stronger.

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

## Migrated to authoring source 2.0, except its transcriber

**Migrated 2026-10-09**, on the trigger recorded on 2026-10-07: the source is
needed, as the content the demonstration's assembly step names.

- `sourceVersion: "1.1.0"` became `authoringSchemaVersion: "2.0.0"`. The old
  field named the source format's version, not the content's.
- `versions` added, `1.0.0` for both `en-US` and `es-US`. Under 1.x the content
  version was supplied at generation and none was ever recorded for this source;
  `1.0.0` matches the sibling sources authored at 2.0.
- `missingPolicy` deleted from both scores. Its value, `propagate`, is what
  content model 2.0 always does.
- The completeness gate deleted, and the band's unreachable `null` arm made plain
  (see *An incomplete PHQ-9 has no total and no band*).
- Vector cases authored (below).

**`transcribedBy` decided 2026-10-09 (Vasu): role `transcriber`, identifier
`Truvex`, method `textLayerAndImage`**, on both languages. From the migration until
then it was left unset, so the source failed validation by name rather than carry
an invented value. The value before migration, `{ "who": "Truvex engine track",
"method": "textLayerAndImage" }`, named neither a role nor an identifier.

**`verifiedBy` is required by authoring source 2.0, and stays `{ "verified": false }`
with its note**, the schema's own form for *nobody has checked it yet*, carried
since 2026-09-20. Nothing about the verification changed.

## Vector cases

Authored, each named for what it proves. The generator adds the rest: every one
of the total's 28 values is pinned (measured: 13 reached by authored cases, 15
generated, `valueCoverage` full).

| Case | Proves |
|---|---|
| complete: every item answered, item 10 too | total 27, band 4; item 10 answered and unscored |
| incomplete: items 1 to 6 answered, 7 to 9 blank | no total and no band, items 7 to 9 named absent on both |
| nothing answered | no total and no band, every item absent somewhere |
| band boundary 4, 9, 14 and 19: below, at, above | each band edge: the total one below, at and one above the cut, with its band |
