# Agent Team Model-Boundary Report

## Scope

This PR is the Batch E topic-pack for Issue #4. It refines model-boundary pages for functional choice, pseudopotentials, DFT+U, SOC and vdW / low-dimensional systems:

- `theory-minimum/pseudopotentials.md`
- `theory-minimum/magnetism-soc-dftu.md`
- `physics-judgement/functional-choice-and-sensitivity.md`
- `physics-judgement/pseudopotential-transferability-and-functional-consistency.md`
- `physics-judgement/u-value-provenance-and-boundary.md`
- `physics-judgement/soc-and-symmetry-boundary.md`
- `physics-judgement/vdw-and-low-dimensional-boundary.md`

No fixed U value, fixed functional recommendation, concrete material case, pseudopotential-generation tutorial or vdW-method ranking was added.

## Subagents Used

| Subagent | Role | Target pages | Output |
|---|---|---|---|
| R-E1 Functional / PP Research | Research | PP theory and functional / PP judgement pages | Brief on functional model choice, transferability, NC/USPP/PAW, semicore, cutoff and relativistic treatment |
| R-E2 DFT+U / SOC Research | Research | magnetism/SOC/DFT+U theory, U provenance and SOC boundary pages | Brief on spin, noncollinear SOC, fully relativistic PP, Hubbard manifold and U provenance |
| R-E3 vdW / Low-dimensional Research | Research | `vdw-and-low-dimensional-boundary.md` | Brief on dispersion, vdW correction, low-dimensional vacuum/electrostatics and functional sensitivity |
| W-E1 PP Theory Writer | Writer | `theory-minimum/pseudopotentials.md` | Added model non-equivalence after PP changes and semicore/projector review evidence |
| W-E2 Functional / PP Judgement Writer | Writer | functional and PP judgement pages | Added model-combination sensitivity and semicore/valence branch boundaries |
| W-E3 U / SOC Theory Writer | Writer | `theory-minimum/magnetism-soc-dftu.md` | Added magnetic-configuration comparison and same-U-value non-equivalence across projectors/PPs |
| W-E4 U / SOC Judgement Writer | Writer | U provenance and SOC boundary pages | Added U transfer boundary and SOC/non-SOC comparison data-chain boundary |
| W-E5 vdW Boundary Writer | Writer | `vdw-and-low-dimensional-boundary.md` | Added vdW vs low-dimensional electrostatics separation |
| Q-E1 Physics QA | QA | all seven pages | Checked universal functional, PP, U, SOC and vdW overclaim risks |
| Q-E2 Style / Source QA | QA | all seven pages | Checked local scope, source boundary and handbook style |

## Research Summary

The team converged on one shared rule: these settings are model choices, not scalar accuracy upgrades. Functional, PP, U, SOC and vdW changes alter the Hamiltonian, energy surface, symmetry, projector space or boundary condition. Downstream comparisons are only meaningful when the model branch and file chain are explicit.

## Writing Changes

- Added that changing PP can make the same cutoff, k mesh and downstream scripts non-equivalent because the effective Hamiltonian changes.
- Added semicore / valence configuration as explicit PP and transferability evidence.
- Clarified that identical numerical U values are not the same model when projector, manifold, PP or structure changes.
- Added magnetic-configuration comparison boundaries for spin-polarized workflows.
- Clarified SOC/non-SOC comparisons require separate complete data chains and energy-reference review.
- Clarified that vdW model choice and low-dimensional electrostatics are coupled but distinct boundaries.

## Physics QA Findings

No BLOCK-level physics issue remains after integration.

Accepted boundaries:

- Functional choice is not framed as a universal ranking.
- Pseudopotentials are not treated as interchangeable technical files.
- PP recommended cutoffs are not treated as convergence conclusions.
- U is not written as an element constant.
- SOC is not written as a simple switch independent of PP, symmetry and k-path.
- vdW is not written as a universal improvement and does not replace vacuum/dipole/electrostatic boundary review.

## Style QA Findings

The changes are local and maintain the reference-handbook style. PASS/WARN/BLOCK sections remain judgement gates rather than replacing explanatory text.

## Source QA Findings

No new DOI, BibTeX, tutorial-as-authority or material-specific source was introduced. Parameter definitions remain tied to QE official documentation; method boundaries remain tied to the existing canonical literature spine.

## Validation Results

Validation commands were run after integration:

- `git diff --check`
- full Markdown local-link check
- private-path and document-residue scan
- project-scope drift scan
- fixed material case and universal-parameter scan
- high-risk overclaim scan for functional ranking, PP interchangeability, U constants, SOC switches, vdW universal accuracy and fixed parameters
- PASS / WARN / BLOCK preservation check

## Remaining Gaps

- This PR does not add a full strong-correlation, relativistic DFT or vdW-method tutorial.
- External QE and pseudopotential documentation links were not live-revalidated in this PR.
- Detailed `hp.x` workflow and version-specific Hubbard syntax remain source-documentation topics.

## Merge Recommendation

Ready for draft PR review after final command validation. The diff is scoped to seven model-boundary pages plus this report.

## Final QA Gate

| Gate | Result | Evidence |
|---|---|---|
| Physics accuracy | PASS | Functional, PP, U, SOC and vdW are framed as model/boundary choices; identical U values across projectors/PPs are not treated as equivalent; SOC/non-SOC comparisons require complete data-chain and energy-reference review. |
| Style consistency | PASS | Edits are local, explanatory and scoped to model-boundary wording; no page was rewritten into a tutorial or literature review. |
| Source boundary | PASS | No new DOI, BibTeX or tutorial-as-authority was added; fields remain tied to QE docs and method boundaries to the canonical source spine. |
| Output-review framing | PASS | Added review evidence for semicore/valence, PP branch changes, magnetic configurations, U provenance, SOC/non-SOC chains and low-dimensional electrostatics. |
| PASS/WARN/BLOCK preservation | PASS | All seven target pages retain explicit PASS/WARN/BLOCK sections or judgement gates. |
| Local links | PASS | Full local Markdown link check reported `OK local markdown links checked: 143 files`. |
| Private path / document residue | PASS | Target-file scan found no private machine paths or external document residues. |
| Project-scope drift | PASS | Target-file scan found no unrelated tooling drift; content stays within QE-Learning model-boundary scope. |
| Fixed material case scan | PASS | No material-specific case examples were introduced. |
| Universal parameter scan | PASS | No fixed U, functional, PP, vdW method, cutoff, k-mesh or SOC importance recipe was introduced. |
| High-risk overclaim scan | PASS | Regex hits for element-constant U, SOC switch, PP interchangeability and universal ranking occur in misconception, boundary or report-context wording; no positive overclaim was found. |
| Merge recommendation | READY | No BLOCK gate remains; suitable for draft PR creation and ready/merge after GitHub mergeability check. |
