# SDMX Reuse Decision Gate

**Development plan (revised)** · MODS–OECD–ADB Global Workshop, Seoul 2026

> The first plan is no longer what was built. What changed is set out first,
> then the system as it stands and what is left.

---

## 1. What changed from the first plan

| | First plan | Now | Why |
|---|---|---|---|
| Delivery | MCP tool | single HTML file | File uploads do not reach an MCP tool as arguments. The engine is separate, so MCP wrapping is still open |
| Stores | standard concepts only | standard **and observed usage** | Real institutional data became available. Duplicating inside one institution is a commoner failure than ignoring the standard |
| Verdicts | 5 | **9** | Two stores add a layer, and code-set comparison becomes possible |
| Version lookup | future work | **built** | The registry query turned out to work |
| Input | named concepts | named concepts **and CSV upload** | One export checks dozens of dimensions at once |
| Decisions | recorded | recorded **and exported as JSON** | The hand-off to an authoring tool needs a document |
| Similarity | weighted percentage | **three facts, reported apart** | There is no defensible set of weights |

**What did not change** — the model proposes search terms, code classifies,
the expert decides. No version is invented and no tie is broken quietly.

---

## 2. Position on the gap map

| Producer task | Still waiting to be written |
|---|---|
| Model a structure | reuse-aware modelling that proposes a standard **before inventing** |
| Reuse artefacts | an assistant that suggests existing concepts and codelists **first** |

Two rows, one gap, seen from two angles: *what* to propose, and *when*.

Every existing tool — FMR, the Global Registry, Matrix Generator, SDMX
Constructor, MAIA, the AI Codelist Mapper, pysdmx-mcp — assumes you already
know what to look for. Search for `REF_AREA` and it is found; start from a
column called "지역" and you never learn that it is there to ask for.

---

## 3. Data

### Standard — 136 concepts
`SDMX:CROSS_DOMAIN_CONCEPTS(2.0)` from the SDMX Global Registry public
endpoint. Concepts that describe SDMX itself are filtered out by the `TYPE`
annotation.

Representation splits five ways, and that split is the spine of the branching.

```
codelist + version        14    resolved
codelist, no version      15    must be looked up — never guessed
several codelists          1    EXPENDITURE; no automatic pick
data type                105    TIME_PERIOD and others; create no CL_
none given                 1    IMPUTATION; the scheme gives none
```

**Limit** — the concept scheme carries codelist *references*, not code values.
Code-set comparison against the standard is therefore impossible, and the tool
reports `not loaded` rather than claiming equivalence.

### Observed usage — 73 dimensions, 1,986 code values (1,819 distinct)
Read out of ten SDMX-CSV files published by one central bank. The headers pair
a code column with an English label — `FREQ, Frequency, UNIT_MEASURE, Unit of
Measure…` — which yields both the dimensions and the code values actually in
use.

**Not authoritative** — it shows that an id is in use. It does not establish
ownership, maintenance, or which version is official. That is said on screen
and in the export.

### What the two stores say to each other
```
same id                12    already using a standard concept
  version unknown       5    REF_AREA COUNTERPART_AREA UNIT_MEASURE
                             VALUATION TRANSFORMATION
same name, different id 1    TIME_COLLECT → TIME_PER_COLLECT
```

`TIME_COLLECT` is the case that carries the project. The ids differ, so no
search finds it; the names match once punctuation is normalised, so the tool
does. And it still refuses to say the code sets are the same, because the
standard code values were never loaded.

---

## 4. The system and how it decides

### The whole flow

```
    input                    judgement                decision       hand-off
─────────────────  ──────────────────────────  ──────────────  ──────────────

 CSV upload ──┐
 (code/label  │   ┌───────────────────────┐
  pairs, code ├──▶│ 1  observed usage      │──┐
  values      │   └───────────────────────┘  │
  collected)  │   ┌───────────────────────┐  │  ┌───────────┐  ┌───────────┐
              ├──▶│ 2  standard scheme     │──┼─▶│ 9 verdicts│─▶│  expert   │
 typed  ──────┘   └───────────────────────┘  │  │ + evidence│  │  chooses  │
 (split on ,;)    ┌───────────────────────┐  │  │ + refusals│  └─────┬─────┘
                  │ 3  ranked suggestions  │──┘  └───────────┘        │
                  └───────────────────────┘           ▲              ▼
                                                       │       ┌───────────┐
                  ┌───────────────────────┐            │       │ snapshot  │
                  │ registry lookup        │────────────┘       │ recorded  │
                  │ (on request only)      │                    └─────┬─────┘
                  └───────────────────────┘                          │
                                                                      ▼
                                                            ┌──────────────────┐
                                                            │ JSON plan        │
                                                            │ → FMR            │
                                                            │ → pysdmx-mcp     │
                                                            └──────────────────┘
```

### Where the verdicts come from

The first layer that catches a term ends it. With no candidate or several,
representation is never examined.

```
input (label, code values)
│
├─ 1 · observed usage ────────────────────────────────────────────────
│   │
│   ├─ id match ─┬─ same id in standard ─┬─ version ────→ LOCAL_USAGE_FOUND
│   │            │                       ├─ no version ─→ REUSE_LOOKUP_REQUIRED ⊘
│   │            │                       ├─ several CLs → UNRESOLVED ?
│   │            │                       └─ data type ──→ LOCAL_USAGE_FOUND ⊘
│   │            │
│   │            ├─ standard has a near one → STANDARD_ALIGNMENT_CANDIDATE ?
│   │            └─ nothing near ───────────→ LOCAL_USAGE_FOUND
│   │
│   └─ identical code set (different id) ───→ EXACT_CODESET_MATCH ?
│
├─ 2 · standard scheme ───────────────────────────────────────────────
│   │
│   ├─ several candidates ──────────────────→ UNRESOLVED ?
│   │
│   └─ one candidate
│       ├─ found by neither id nor name ────→ SIMILAR_EXISTS ?   ← demoted
│       ├─ version present ──────────────────→ REUSE_CONFIRMED
│       ├─ version absent ───────────────────→ REUSE_LOOKUP_REQUIRED ⊘
│       ├─ several codelists ────────────────→ UNRESOLVED ?
│       └─ data type ────────────────────────→ REUSE_NON_CODE ⊘
│
└─ 3 · ranked suggestions ────────────────────────────────────────────
    ├─ something close (top 3) ──────────────→ SIMILAR_EXISTS ?
    └─ nothing ──────────────────────────────→ SEARCH_OTHER_SCHEME ⊘

    ⊘ a refusal travels with the verdict    ? a question goes back to the expert
```

Seven of the nine do not simply end. The tool asks more often than it answers.

### Who does what

```
        proposes                  classifies               decides
┌──────────────────────┐ ┌──────────────────────┐ ┌──────────────────────┐
│      the model       │ │       the code       │ │      the expert      │
├──────────────────────┤ ├──────────────────────┤ ├──────────────────────┤
│ label → English      │ │ search both stores   │ │ pick a candidate     │
│ search terms         │ │ compute three facts  │ │ create a local one   │
│                      │ │ classify the state   │ │ defer                │
│ ✗ never picks the    │ │ attach the refusals  │ │                      │
│   concept            │ │                      │ │ this alone is the    │
│ ✗ never sees a       │ │ ✗ uses no            │ │ decision             │
│   version            │ │   probabilities      │ │                      │
└──────────────────────┘ └──────────────────────┘ └──────────────────────┘
```

No branch of the classification consults the model. No statistical value is
generated anywhere in the system.

### Four screens
```
Overview    the problem · three steps · nine verdicts · the division of labour
Storage     both stores and what each can and cannot settle · representation mix
Check       upload or type → verdict → ranked suggestions → choose
Decisions   what was recorded · export as JSON
```

### Evidence, not confidence
Three facts are computed and reported separately. There is no combined score.

```
Code set   exact · partial · none · not loaded · not comparable
Name       normalised match · tokens overlap · different
ID         same · different
```

Ordering is lexicographic: a non-empty exact code set, then overlap, then a
normalised name match, then name tokens, then id tokens.

### Decisions
```
pick a candidate / Accept this   REUSE
Keep local / create new          CREATE_LOCAL
Defer                            DEFER     ← not a resolution; stays in the count
```

With nothing found, `Accept this` is not offered. Recording the typed label as
a target would invent a standard concept — which is precisely what that verdict
was warning about.

The verdict, the evidence and the time are snapshotted when the expert chooses,
and the export never recomputes them.

### Version lookup
A button on the 15 `REUSE_LOOKUP_REQUIRED` verdicts. Called only on request.

```
/codelist/all/{id}/all?detail=referencestubs&references=none&format=sdmx-2.1
```

**Measured** — `CL_AREA` is maintained by ESTAT (1.8), SDMX (2.0) and
IAEG-SDGs. The concept scheme gave the id but not the agency. The version was
not the only thing missing; so was the agency.

The newest is never selected automatically. Results are grouped by agency, then
newest within each — ordering across agencies by version alone would imply a
ranking that does not exist.

Where the fetch is blocked — offline, CSP, CORS — the verdict is untouched and
the query URL is shown as a link. The tool cannot fetch it, but it can still
say where to look.

---

## 5. Verification

Building and checking were kept apart: one side wrote the tool, the other ran
every input through a headless browser and audited the result. **Six P0 defects
surfaced that the builder's own checks had not.**

### P0 found and fixed
| | Symptom | Cause |
|---|---|---|
| 1 | 83 standard ids containing `_` never resolved to their own concept | the search term's `_` became a space; the id it was compared against kept its `_` |
| 2 | one shared word in a description produced a reuse verdict, version and all | `TIME_SOURCE → FREQ_COLL / SDMX:CL_FREQ(2.0)` |
| 3 | a refresh lost the reason for a decision; re-checking attached a different verdict | the export looked the verdict up in volatile memory |
| 4 | the placeholder's own example returned "not found" for three concepts that exist | the whole string was evaluated as one term |
| 5 | accepting a "not in this scheme" verdict recorded the typed label as a reuse target | the accept branch fell back to the label when nothing was found |
| 6 | the contamination filter discarded four kinds of sound decision | a candidate pick has no single hit, so `found` is empty |

### Vulnerability
Quotes escaped their attribute context. Registry responses, CSV headers and
file names all travel that path, so the escaping was fixed for all of them.

### Regressions introduced while fixing, then caught
- normalisation was asymmetric, so 48 names containing punctuation stopped matching
- keeping only the strongest tier let one concept's id hide another's name

### Safety invariants that hold
```
nothing found means nothing can be recorded as reused
identical code values are evidence, never proof of the same meaning
no code-set match is claimed where the standard values were never loaded
a missing version is never invented
several candidates are never resolved silently
no codelist is proposed for a data type
absence from this scheme is never reported as absence from SDMX
nothing the expert did not choose is exported as a decision
a stored decision keeps the reason it was made
a failed network call never changes a verdict
```

**Regression suite** — 726 inputs: 136 standard ids, 136 standard names, 73
observed dimensions, 17 seed terms, 6 presets and the rest.

---

## 6. Against the judging criteria

| Criterion | This prototype | Why |
|---|---|---|
| **Trustworthiness** | strong | No version is invented, no tie is broken quietly, and nothing found means nothing can be recorded as reused. Only the expert decides |
| **Transparency** | strong | Three facts reported apart, no probabilities, and the verdict is snapshotted when the expert chooses — never recomputed on export |
| **Impact** | see below | Three before-and-after pairs |
| **Usefulness** | strong | Two boxes on the gap map. `TIME_COLLECT` was found in real institutional data |
| **Continuation** | see §8 | Feeds the design stage of a 2027 build |

### Impact — three before-and-after pairs

**1 · An assistant answering alone, and the same question through the Gate**

```
before   "Which SDMX concept and codelist should a region dimension use?"
         → Use REF_AREA with CL_AREA(2.0)
                              ↑ the version was invented

after    Gate → REUSE_LOOKUP_REQUIRED
         ⊘ Do not invent a version.
         lookup → ESTAT(1.8) · SDMX(2.0) · IAEG-SDGs
         not only the version was missing — so was the agency
```

**2 · Searching, and being offered**

```
before   search the registry for "지역" → nothing
         you never learn that REF_AREA is there to ask for

after    type "region" → REF_AREA offered, with the version warning
```

**3 · What actually happened during the build**

```
before   TIME_SOURCE → REUSE_CONFIRMED · FREQ_COLL · SDMX:CL_FREQ(2.0)
         one word shared with a description, and a version was settled

after    TIME_SOURCE → itself · String · ⊘ Do not create CL_TIME_SOURCE
```

The third says most about what this prototype is. Being confidently wrong was
not a hypothesis — it happened to this tool, and it was caught because building
and checking were kept apart.

---

## 7. What is left

| | Item | Judgement |
|---|---|---|
| P1 | CSV headers can be paired wrongly without saying so (other formats) | Bound the supported shape in the documentation; the code fix is optional |
| P1 | file name and input mode are absent from the export | Traceability; no effect on the demonstration |
| P2 | `match_tier` is null when a candidate was picked | Correct in meaning, inconsistent in form |
| P2 | more than three candidates are not counted on screen | Zero cases in this data |
| P2 | Google Fonts is requested despite "no internet required" | A wording fix is enough |

**Deliberately not built**
- searching existing DSDs and dataflows — the largest overlap with FMR and Matrix Generator
- writing into FMR — `sdmx-audit-mcp` and `pysdmx-mcp` already do it
- generating DSD XML — authoring is a separate step
- extending another agency's codelist — not permissible
- a confidence score — no defensible weights, and it invites the automatic acceptance the tool exists to prevent

---

## 8. Limits

The tool searches one concept scheme and reports what is in it, nothing more.
It does not know whether a concept exists elsewhere, whether a codelist covers
the codes a dataset needs, or whether a standard concept means what a local
column means.

The CSV adapter reads the code-and-English-label shape of this evidence data.
It should not be described as reading every SDMX-CSV.

Korean input works when a connected chatbot derives the search terms. The
browser page alone takes ids and English labels.

Observed usage was read out of published data. Whether that institution
registered those artefacts, and which version is official, was not checked.

---

## 9. Continuation

This is not meant to be shown once and put away.

**Inside the institution** — an AI-friendly statistical metadata programme
enters its build stage in 2027. Placing this decision layer in front of
structure design means the standard is consulted before a local concept is
invented. The slot already exists.

**Swapping the data** — replacing the `<script id="evidence-store">` block is
enough; counts, listings and matching all follow. Moving to another
institution or another concept scheme needs no code change.

**MCP wrapping** — the engine is separate from the screen, and the connection
to the workshop lab's chatbot has already been verified. Turning it into a tool
called in plain language is a few days' work, and that is where Korean input
starts working.

**Collaboration** — a proposal arrived to use Israel's Business Tendency Survey
and Korea's Business Survey Index as test data. The two cover the same concepts
under different classifications and different numerical conventions, which
suits testing codelist extension and unit transformation. If the corresponding
DSD is registered in the Israeli registry, that adds an authoritative store.

---

## 10. Where the tool stops

At the decision. Authoring belongs to FMR, Matrix Generator and `pysdmx-mcp`.
`Download plan (JSON)` is the hand-off.

```
the model proposes what to search for
the registry supplies the evidence
rules classify the state
the expert decides
an authoring tool builds the structure
```

> Search is not the problem.
> Knowing what to search for — and when not to decide — is.
