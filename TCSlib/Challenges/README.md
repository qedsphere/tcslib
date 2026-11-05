# TCSlib/Challenges: statement-only challenge problems

This area is a **problem set, not a library layer**. Every file states one claim from a
recent paper and leaves every proof body as `sorry`. Files here are leaves of the import
graph: nothing else in TCSlib may import from `TCSlib.Challenges`.

## Layout

```
TCSlib/Challenges.lean                      area facade
TCSlib/Challenges/STOC2026.lean             conference facade (one Contents line per paper)
TCSlib/Challenges/STOC2026/<Paper>.lean     paper facade (one Contents line per node)
TCSlib/Challenges/STOC2026/<Paper>/NodeNN.lean   one self-contained statement file per claim
TCSlib/Challenges/STOC2026/MANIFEST.tsv     module, paper_id, node_id, prompt_type, difficulty,
                                            parquet, compile status, claim kind, sorry count
TCSlib/Challenges/STOC2026/notes.jsonl      the formalizers' notes: definition map, dropped
                                            aspects, faithfulness notes, lemma stubs
TCSlib/Challenges/STOC2026/build_order.txt  compile order, children before facades
```

`<Paper>` is a CamelCase prefix of the title; the namespace inside each file is
`TCSChallenge.<Paper>.node_NN`, so files never collide and any subset can be imported.

## Provenance

The claims are the `proof_writing` and `proof_strategy` rows of the tcs_v3 dataset's
STOC 2026 split (parquet shards `gpt_5_6_sol-*` and `gpt_6_1_sol-*`, named in each
file's `## References`). A `proof_strategy` row that asks about the same claim as a
`proof_writing` row of the same paper shares that file; only rows without such a twin
got their own file. Rows whose claim is a running-time, kernel-size or oracle-model
statement with no faithful first-order rendering were marked not formalizable; they are
listed in `MANIFEST.tsv` with `status = not_formalizable` and the formalizer's reason.

The files were written by headless Claude formalizer agents, one paper at a time, in the
`reasoning/lean_formalization` pipeline (`scripts/make_packs.py`, `run_agents.py`,
`check_lean.sh`, `lint_fix.py`, `export_to_tcslib.py`). Each file compiled against the
TCSlib pin (Lean v4.25.0, Mathlib `029db123ddaa`) before export; the compile result and
the `sorry` count are recorded in the file's `## Provenance` section and in the manifest.

A second pass (`run_agents.py refactor`, one fresh agent per file) then rewrote 59 of the
files (two passes) to use TCSlib's own definitions instead of local copies: `TCSlib.InformationTheory.FinDist`
(with `uniform`, `map`, `statDist`, `HasMinEntropy`) for finite distributions,
`BooleanAnalysis.BoolCube`/`BooleanFunc`/`boolToSign`/`chiS`/`fourierCoeff`/`influence` and
`KKL.IsJunta` for Boolean functions, `SwitchingLemma.Restriction` with `extend`/`freeVars`,
`DecisionTree`, `BooleanAnalysis.ThresholdFunctions.standardGaussianMeasure`, `Literal`/`Literal.eval`,
`Networks.Network` and `CommunicationComplexity.Deterministic.Protocol`. Seven more files kept
their local definitions for a stated reason (index types that are not `Fin n`, circuits with
constant wires, computed distributions that cannot be bundled without proofs). Every file's
header carries a `TCSlib reuse:` line, and its `## Provenance` section lists what was replaced,
what was kept and how the statement shape changed (agent self-report; not human-reviewed).

## Status against policy.md

| Checklist item | State |
|---|---|
| 1. Area/Topic layout with facades; `TCSlib.lean` export | done (generated) |
| 2. Precise imports, no bare `import Mathlib` | done (each file carries only the imports it uses) |
| 3. `## References` in every file; `[Tag, node]` on every theorem and lemma docstring | done mechanically; the tag is the paper short name and the tcs_v3 node |
| 4. Proof sketch for nontrivial proofs | not applicable: every proof is `sorry`; the linter's "no proof sketch" hits are long *statements* |
| 5. Docstring on every public declaration | `scripts/style_lint.py --area Challenges` lists the remaining undocumented declarations |
| 6. Zero errors per file | every file compiled in isolation; the facades and `TCSlib.lean` were compiled once after export |
| 7. Blueprint | no blueprint entries were written; the dataset pipeline will report these declarations as orphans until it stubs them |
| 8. No new private host | not applicable: no machines or constructions are built here |

**Faithfulness has not been reviewed by a human.** Each file's `## Provenance` section
carries the formalizer's own list of what was dropped or changed and its self-review
notes; the pipeline's back-translation check has not been run. Treat every statement as
a draft until a reviewer has compared it with the paper.

## Effects on the rest of the repository

* `lake build TCSlib` now also builds these modules (about 30 minutes of extra sequential
  compile time on first build; they import only Mathlib and are leaves, so nothing else
  rebuilds).
* `scripts/dataset_pipeline.py` sees them as built declarations without blueprint entries.
  Decide whether they should enter the dataset (as `proof_complete = false` records) or be
  excluded before running it.
* Nothing in `TCSlib/Challenges` is a proved result; do not cite it from library files.
