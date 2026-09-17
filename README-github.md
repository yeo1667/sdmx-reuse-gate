# SDMX Reuse Decision Gate

**Check before you create.** Before a new SDMX structure is built, this
tells you whether the concepts and codelists you are about to define
already exist — in the SDMX standard, or in your own institution.

> Search is not the problem.
> Knowing what to search for — and when not to decide — is.

**[Open the tool →](reuse-gate.html)**

---

## The problem

SDMX asks you to reuse standard concepts and codelists. Every tool can find
`REF_AREA` — once you know to ask for it. Starting from a column called
"지역", you never learn that it is there, so you invent `COUNTRY`, and
interoperability is gone before the DSD is written.

Three more failures happen at the same moment:

- **Invented versions.** Of 136 cross-domain concepts, 15 name a codelist
  without a version. Ask an assistant to fill in a DSD and it will supply one.
- **Codelists that should not exist.** `TIME_PERIOD` is a data type.
  `CL_TIME_PERIOD` is plausible and wrong.
- **Silent disambiguation.** "Frequency" is three concepts — of observation,
  of collection, of dissemination. Choosing one quietly is worse than
  choosing none.

## What it does

Upload a DSD export or a data file, or name the concepts. Every component
comes back with a verdict, the evidence behind it, and — where it matters —
a question rather than an answer.

| Verdict | Meaning |
|---|---|
| `REUSE CONFIRMED` | Concept found; its codelist carries agency, id and version |
| `DEFINED IN A DSD` | Your institution already defines this component |
| `LOCAL CODELIST · STANDARD EXISTS` | A local codelist where a standard one also exists |
| `LOCAL USAGE FOUND` | The id appears in published data — use, not ownership |
| `STANDARD ALIGNMENT CANDIDATE` | Nothing of this id, but something close |
| `VERSION LOOKUP REQUIRED` | Codelist named, version absent — resolve it, never guess |
| `EXACT CODE SET · REVIEW` | Same values elsewhere. Evidence, not proof of same meaning |
| `SIMILAR EXISTS` | Closest matches, ordered by evidence |
| `UNRESOLVED` | Several candidates with equal evidence. The tool refuses to pick |
| `SEARCH ANOTHER SCHEME` | Not in what was searched — not the same as absent |

## How it decides

```
0  registered DSDs      what the institution has defined
1  observed usage       ids and code values read from published data
2  standard scheme      SDMX cross-domain concepts
3  ranked suggestions   closest matches by evidence
```

The first layer that catches a term ends it — but a local hit never mutes a
standard signal. A missing version is still missing; a data type is still not
a codelist.

**Evidence, not confidence.** Three facts are computed and reported
separately. There is no combined score, because there is no defensible set
of weights.

```
Code set   exact · partial · none · not loaded · not comparable
Name       normalised match · tokens overlap · different
ID         same · different
```

## What it never does

- invent a codelist version
- pick one of several candidates quietly
- propose a codelist for a data type
- call two things the same because their code values match
  (`CONFIDENTIALITY` and `PROVISIONAL` can both be Y/N)
- report absence from one scheme as absence from SDMX
- record a reuse when nothing was found to reuse

## Try it

Two DSD exports are included.

1. Drop **`EXR.csv`** on the Check screen — it is judged immediately
2. Press **Keep this DSD in storage**
3. Drop **`BKN.csv`** — 13 of its 18 components are already in `EXR`,
   and three of those use a local codelist where a standard one exists

On the `region` card, try **Look up versions in the registry**.
`CL_AREA` is maintained by ESTAT, SDMX and IAEG-SDGs: the concept scheme
gave the id but not the agency. The version was not the only thing missing.

## Running it

One file. Open `reuse-gate.html` in a browser — no install, no server, no
account. The data is inside it.

To use different data, replace the `<script id="evidence-store">` block;
counts, listings and matching all follow.

## Where it stops

At the decision. Authoring belongs to FMR, Matrix Generator and
`pysdmx-mcp`. **Download plan (JSON)** is the hand-off.

```
the model proposes what to search for
the registry supplies the evidence
rules classify the state
the expert decides
an authoring tool builds the structure
```

## Limits

The tool searches one concept scheme and reports what is in it, nothing
more. The concept scheme carries codelist *references*, not code values, so
code-set comparison against the standard is not possible — and the tool says
`not loaded` rather than claiming equivalence.

The CSV adapter reads FusionXL DSD exports and data files that pair a code
column with an English label. It should not be described as reading every
SDMX-CSV.

Observed usage was read out of published data. It shows that an id is in
use; it does not establish ownership, maintenance, or which version is
official.

---

Built at the **MODS–OECD–ADB Global Workshop**, Seoul, 15–17 September 2026.
See [`development-plan-en.md`](development-plan-en.md) for the design, the
verification, and what is deliberately not built.
