# Where TCSlib almost fits: the encoding-mismatch files

Date: 2026-11-04. Corpus: the 259 STOC 2026 challenge files in `challenges/stoc2026_pw` and
`challenges/stoc2026_ps`. Companion to the refactor pass (`batches/refactor_tcslib`), after which
59 files (23%) use TCSlib definitions.

The concept scan flagged 61 files (24%) that mention a TCSlib area but could not be refactored.
Reading each file's definitions splits them into three groups:

| Group | Files | What it means |
|---|---|---|
| A. Scan false positives | 16 | a keyword matched (walk, game, protocol, Gaussian) but the file has no object TCSlib defines |
| B. Missed drop-ins | 10 | the local definition is a TCSlib definition under another name; the target list missed it |
| C. Genuine encoding gaps | 35 | TCSlib has the concept in a shape the paper cannot use without changing the statement |

Group C is the interesting one: each row names the exact gap and the smallest TCSlib change that
would close it. Files are named by stem (`S<row>_<Paper>_node_<id>`).

## A. Scan false positives (16)

No TCSlib object is reinvented here; these belong with the 141 no-overlap files.

- `ADeterministicNearLinear_22` (transport cost of a point walk), `DeterminationOfTheFifth_14`
  (Turing machines), `KrylovMethods_04` (Haar-orthogonal matrices), `TheSubspaceFlatness_05`
  (lattices), `WhyWeCouldntProve_23` (LLL), `ProximalRegret_18` (online gradient descent runs),
  `NoExponentialQuantumSpeedup_05/25/26/32` (short-vector and zero-sum combinatorics),
  `ImprovedBoundsForCoin_18` (adversary transcripts on bit tapes), `SmoothedAnalysis_20`
  (nested-restriction algorithm on measures; "restriction" is a different word),
  `QueryOptimalEstimation_11`, `TheDebiasedKeyls_09` (quantum tomography), `SmoothedAnalysis_12`
  (learners over measures).

## B. Missed drop-ins (10)

Same definition, different local name. A second `refactor` pass (2026-11-04, $3.6) converted the
first nine; the two `NPMembershipForThe` files were left because their literals carry real-interval
semantics and the gain is cosmetic. After it, 68 files (26%) use TCSlib.

| File | Local | TCSlib |
|---|---|---|
| `AD12o1MonotonicityTester_03` | `IsQueryDist (w : Finset (Grid d n) → ℝ)` | `FinDist (Finset (Grid d n))` |
| `FewSingleQubitMeasurements_03/04` | `RandLocalTest.w : Fin m → ℝ` weights; `D : (Fin M → Bool) → ℝ` | `FinDist (Fin m)`, `FinDist (Fin M → Bool)` |
| `SemiStreamingMatchingIn_19/33` | `RandStreamAlg` randomness field + `IsDistribution` | `FinDist` field |
| `RelaxedVsFullLocal_08` | `RandTree.w : Fin r → ℝ` + `IsDist` | `FinDist (Fin r)` (the tree type itself is group C) |
| `FindingBugsInShort_17` | `Lit n := Fin n × Bool` | `Literal n`, `Literal.eval` |
| `SeparatingQMAFromQCMA_04` | `Cube n := Fin n → Bool` (28 uses) | `BoolCube n` |
| `OnZerosAndAlgorithms_06` | nested `Measure.pi … gaussianReal 0 1` | `standardGaussianMeasure` inside the same `Measure.pi` |
| `NPMembershipForThe_15/25` | `structure Lit` with `var`, `pos` (cosmetic; the values are real intervals) | `Literal n` with `neg := !pos` |

## C. Genuine encoding gaps (35)

### C1. Codes indexed by something other than `Fin n` (5 files)

- `AProofThatReed_22/24`: words are `(ι → ZMod 2) → ZMod 2`, indexed by the points of a cube.
- `OptimalTestingOfGeneralized_03`: `relDist` on functions `(Fin n → F) → F`.
- `IdealsMacaulayBasesAnd_05/09`: `relDist` and `distToSat` over an arbitrary finite `α`.
- `ImprovedHardnessOfApproximating_24`: `relDist (w w' : Fin L → A)` with a plain alphabet `A`.

Gap: `ErrorCorrectingCodes.Codeword n α` is `Fin n → α`, and the section variables demand
`[Field α]` even for `hamming_distance`, `weight`, `hamming_ball`. Fix: state the metric
definitions for `ι → α` with `[Fintype ι] [DecidableEq α]` (Mathlib's `hammingDist` already is),
and add `relDist := hammingDist / card ι`. That unlocks all five.

### C2. Formulas over generic variables, and proof systems (2 files)

- `FindingBugsInShort_15`: `Lit V := V × Bool` for an arbitrary `V`, `Finset` clauses, resolution
  derivations, refutation width.
- `OnSmallDepthFrege_27`: Frege formulas over `Edge m`, evaluation trees, perfect matchings.

Gap: `Literal n` is `Fin n`-indexed; clauses are lists; there is no resolution or Frege proof
system. Fix: `Literal (V : Type)` with a `Finset`-clause reading, plus `Resolution.Derivation`
with `width`; Frege is a larger addition.

### C3. Decision trees that are not Bool-to-Bool (2 files)

- `RelaxedVsFullLocal_08`: `DTree n α`, inputs `Fin n → ZMod 2`, leaves of type `α`, plus a
  randomized tree `RandTree` (weighted finite mixture).
- `FindingBugsInShort_17`: `DT R` querying rewrite instances with leaves in `R`.

Gap: `DecisionTree n` fixes inputs to `Fin n → Bool` and leaves to `Bool`. Fix: generalize to
`DecisionTree (ι : Type) (σ : Type) (α : Type)` with `eval : (ι → σ) → α` and keep the Boolean
case as an abbreviation; add `RandDecisionTree := FinDist` over trees.

### C4. Circuits with features the TCSlib models lack (7 files)

- `PolynomialIdentityTestingAnd_12/24`: arithmetic circuits with `const (c : F)` gates and
  `MvPolynomial` semantics. `Randomized.AlgCircuit` has only `one` (so only integer constants).
- `MonotoneCircuitComplexityOf_03/05`: Boolean circuits with constant wires and inputs indexed by
  `Fin n × Fin n` (edges). `BoolCircuit.DAGCircuit` has neither and carries proof fields, so a
  circuit cannot be built in a definition.
- `MagicAndCommunicationComplexity_09/12`, `QuantumPrecomputationParallelizingCascad_08`: unitary,
  Clifford and magic-gate circuits on qubits. No quantum objects in TCSlib.

Fix: `AlgGate.const (c : R)`; a `DAGGate.const b` kind and an index-type parameter on
`DAGCircuit`; quantum circuits are a new area (states, unitaries, Pauli/Clifford groups,
measurement), which would also serve `FewSingleQubitMeasurements`, `SeparatingQMA` and
`TheDebiasedKeyls` from the no-overlap group.

### C5. Multigraphs with loops or indexed edges (4 files)

- `ADeterministicAlmostLinear_29`: `Multigraph W := List (Sym2 W)`, `addLoops`, degree, volume,
  cut, conductance expander `IsExpanderOn`.
- `LipschitzContinuousAlgorithmsFor_05/21/23`: `ends : Fin m → Sym2 (Fin n)` with distributions
  over edge-index subsets, spanning trees and matchings as `Finset (Fin m)`.

Gap: `KargerMinCut.Multigraph` forbids self-loops (`loopless`), stores edges as a `Multiset`, and
cuts carry nonemptiness proofs. Fix: an index-based multigraph (`ends : Fin m → Sym2 V`) with
`deg`, `vol`, `cutEdges`, `conductance`, `IsSpanningTree`, `IsMatching` on edge-index sets; Karger's
structure can be derived from it.

### C6. Matchings outside `SimpleGraph.Subgraph` (4 files)

- `SemiStreamingMatchingIn_19/33`: matchings in an edge-list stream.
- `ChasingPositiveBodies_24`: fractional matchings `x : L × R → ℝ` on a bipartite edge set.
- `NearOptimalHardnessOf_06`: `k`-dimensional hypergraph matchings.

Gap: TCSlib's `Core.Matchings` works on `G.Subgraph` of a `SimpleGraph` (maximum matchings,
augmenting paths, k-factors). Fix: `IsMatching (M : Finset (Sym2 V))` and
`IsMatching (M : Finset E)` for hyperedges `E → Finset V`, plus `IsFracMatching`.

### C7. Spanning trees and cuts as edge sets (5 files)

- `ThinTreesForLaminar_08/09`: `cut G S : Finset (Sym2 V)`, `IsSpanningTreeOf G S T`, weights.
- `PlanarLengthConstrainedMinimum_33/38`: `IsSpanningTreeOf (G T : SimpleGraph V)`, length-bounded.
- `ChasingPositiveBodies_30`: `IsSpanningTreeOf G (A : Finset (Sym2 (Fin n)))`.

Gap: TCSlib cuts are numbers on capacitated `Networks.Network`s, and no spanning-tree notion
exists (Mathlib has `IsTree`, which every file re-wraps differently). Fix: `cutEdges G S` and one
`IsSpanningTreeOf` on `Finset (Sym2 V)`, with the `SimpleGraph` form as a lemma.

### C8. Property testers (3 files; also 3 partially refactored ones)

- `TestingNoisyLowDegree_09/10`: `Tester ℓ n m := (queries) → (answers) → ℝ` as an acceptance
  functional, with measurability and noise.
- `AD12o1MonotonicityTester_07`: influence on the hypergrid `Fin d → Fin n`, up/down walks.
- `AStrongCompositionTheorem_19/21/35` (refactored for distributions; `Tester` kept): an inductive
  query tree with `acceptProb`, tolerant and zero-sided variants.

Gap: TCSlib has BLR and low-degree tests but no tester type. Fix: `Tester (X Y : Type)` =
distribution over query trees, `acceptProb`, `IsTester P ε q`, and influence over product spaces.

### C9. Expanders by conductance or existence (2 files)

- `ADeterministicAlmostLinear_29`: `IsExpanderOn H B φ` by conductance.
- `NearOptimalHardnessOf_35`: `ExpanderFact C`, an existence statement for `D`-regular expanders.

Gap: TCSlib's expander notion is spectral (`lambda` of a symmetric stochastic matrix). Fix:
conductance, Cheeger's inequality, and an explicit-expander existence theorem.

### C10. Communication protocols beyond two parties (2 files)

- `StreamingLowerBoundsAnd_09`: `k`-party number-in-hand randomized protocols with per-player
  bit budgets.
- `MagicAndCommunicationComplexity_13`: private-coin protocols whose messages are biased coins.

Gap: TCSlib has deterministic, public-coin and private-coin two-party protocols with explicit
tapes. Fix: a `k`-party protocol tree with a message budget per player; biased-coin messages are a
`FinDist`-valued variant of `Protocol.alice`.

### C11. Hermite functions and codes with explicit algebra (2 files)

- `EfficientQuantumHermiteTransform_20`: physicists' Hermite `2^(n/2) He_n(√2 x)`; TCSlib has the
  probabilists' `normalizedHermitePolynomial`. Fix: define the physicists' family and the
  rescaling lemma.
- `TheWeakRankPrinciple_32`: Reed–Solomon-style `CodeData` with Lagrange interpolation. TCSlib has
  no Reed–Solomon or Reed–Muller code. Fix: `ReedSolomon` and `ReedMuller` as `Code`s.

### C12. Total variation on measures (2 files)

- `QueryLowerBoundsFor_03`, `SparsifyingSumsOfNorms_29`: `tvDist` on `Measure`s.

Gap: `statDist` is finite only. Fix: `tvDist (μ ν : Measure α)` with the `FinDist` case as a lemma.

### C13. Distributions that are computed, not given (1 file here, 4 partially refactored)

- `StrongSpatialMixingFor_04`: subprobability vectors and marginals computed from counts.
- Partially kept in `MixingOfGeneralBiased_05`, `PseudorandomHashingForSpace_11`,
  `TwoSourceExtractorsFor_38`, `UniquenessAndRapidMixing_16`: kernel powers, conditioning,
  `k`-fold products.

Gap: `FinDist` bundles nonnegativity and normalisation proofs, so a definition cannot build one
without proving them. Fix: constructors `FinDist.ofSubtypeUniform (s : Finset α) (hs)`,
`FinDist.cond`, `FinDist.kernelPow`, `FinDist.pi (k)`; or a parallel predicate API
`IsFinDist (f : α → ℝ)` with `statDist'` on plain vectors.

### C14. Learning beyond Hedge and minimax (2 files)

- `SmoothedAnalysisOfLearning_12` (learners over measures with smoothness), `ProximalRegret_15`
  (`Φ`-regret).

Gap: `LearningTheory` has external regret for finite experts, minimax, Johnson–Lindenstrauss and
mistake bounds. Fix: `phiRegret`, and a PAC learner type over `Measure`s.

## What would move the needle

Ranked by files unlocked per definition added:

1. Index-generic codes and `relDist` (C1): 5 files, a few lines.
2. One `IsSpanningTreeOf` and `cutEdges` on edge finsets (C7): 5 files.
3. Edge-finset and hypergraph matchings (C6): 4 files.
4. An index-based multigraph with loops (C5): 4 files, and it subsumes Karger's.
5. `AlgGate.const` and constant wires in `DAGCircuit` (C4, classical half): 4 files.
6. `FinDist` constructors for computed laws (C13): finishes 5 files already half-converted.
7. A tester type (C8): 6 files.

A quantum area (states, Clifford circuits, measurements) is the single largest gap by file count
across both the mismatch and no-overlap groups (about 15 files) but is a new library, not a fix.
