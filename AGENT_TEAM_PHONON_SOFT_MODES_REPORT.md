# Agent Team Phonon / Soft-Mode Report

## Scope

This PR is the Batch C topic-pack for Issue #4. It refines only the phonon theory, soft-mode judgement and phonon-debugging workflow pages:

- `theory-minimum/dfpt-phonons.md`
- `physics-judgement/09-phonons-soft-modes-and-dynamical-stability.md`
- `workflows/phonon/phonon-debugging.md`

No EPC, Wannier, thermodynamics, broad workflow expansion, concrete material case or universal parameter recommendation was added.

## Subagents Used

| Subagent | Role | Target pages | Output |
|---|---|---|---|
| R-C1 Phonon physical-picture Research | Research | `theory-minimum/dfpt-phonons.md` | Brief on harmonic approximation, Born-Oppenheimer energy surface, force constants, dynamical matrix and acoustic/optical modes |
| R-C2 Soft-mode Research | Research | `physics-judgement/09-phonons-soft-modes-and-dynamical-stability.md`, `workflows/phonon/phonon-debugging.md` | Brief on imaginary frequency, negative curvature, ASR, q-grid sensitivity, eigenvectors and real instability triage |
| W-C1 DFPT Phonon Writer | Writer | `theory-minimum/dfpt-phonons.md` | Added eigenvector, direct q-point vs interpolation and q-grid/IFC physical context |
| W-C2 Soft-mode Judgement Writer | Writer | `physics-judgement/09-phonons-soft-modes-and-dynamical-stability.md` | Added `omega^2` interpretation, evidence layers, virtual-frequency categories and stronger PASS/WARN/BLOCK conditions |
| W-C3 Phonon Debugging Writer | Writer | `workflows/phonon/phonon-debugging.md` | Added operational triage order and direct q-point / eigenvector review steps |
| Q-C1 Physics QA | QA | all three pages | Checked Gamma stability, ASR, imaginary-frequency and curvature statements |
| Q-C2 Style QA | QA | all three pages | Checked that theory, judgement and workflow pages stay distinct |
| Q-C3 Source / Link QA | QA | all three pages | Checked source boundary and local links |

## Research Summary

The team converged on four boundaries:

- Phonon frequencies are evidence about the curvature of the Born-Oppenheimer energy surface; imaginary frequencies represent negative `omega^2`, not a separate experimentally observed oscillation.
- A suspicious mode needs both frequency and eigenvector evidence. The frequency gives curvature; the eigenvector gives the collective displacement pattern.
- `ph.x` direct q-point results and `matdyn.x` interpolated path results are different evidence layers. A soft mode seen only after interpolation should trigger q-grid/IFC/file-chain review.
- ASR is a diagnostic and constraint for translational invariance. It cannot repair unconverged structures, wrong files or failed response calculations.

## Writing Changes

- Strengthened `dfpt-phonons.md` with eigenvector / displacement-pattern interpretation and the difference between direct q-point DFPT evidence and interpolated `matdyn.x` evidence.
- Added q-grid / IFC truncation language to explain why acoustic drift, local virtual modes or phonon DOS low-frequency artifacts can appear.
- Expanded the soft-mode judgement page with a compact `omega^2` theory layer, evidence-layer table and imaginary-frequency classification table.
- Tightened PASS/WARN/BLOCK rules so direct q-point evidence, eigenvectors, q-grid/IFC review and ASR boundaries are explicit.
- Made the phonon-debugging page more operational by adding a seven-step triage order from structure/SCF to controlled rerun.

## Physics QA Findings

No BLOCK-level physics issue remains after integration.

Accepted boundaries:

- Gamma phonon is not presented as full Brillouin-zone stability.
- ASR is described as diagnostic/constraint, not a repair for bad upstream data.
- Imaginary frequency is tied to negative curvature and requires triage before a real-instability claim.
- Direct q-point, interpolation, eigenvector and file-chain evidence are separated.

## Style QA Findings

The three pages keep distinct roles:

- `dfpt-phonons.md` gives theory-minimum background.
- `09-phonons-soft-modes-and-dynamical-stability.md` gives physical judgement rules.
- `phonon-debugging.md` gives workflow triage steps.

The added content uses explanatory paragraphs and compact tables rather than a checklist-only rewrite.

## Source QA Findings

No new DOI, BibTeX, tutorial-as-authority or material-specific source was introduced. Parameter and program behavior remain tied to QE official INPUT references and the PHonon user guide; physical-method boundaries remain tied to the existing DFPT canonical literature spine.

## Validation Results

Validation commands were run after integration:

- `git diff --check`
- full Markdown local-link check
- private-path and document-residue scan
- project-scope drift scan
- fixed material case and universal-parameter scan
- high-risk overclaim scan for Gamma stability, ASR, imaginary frequency and direct/interpolated phonon evidence
- PASS / WARN / BLOCK preservation check

## Remaining Gaps

- This PR does not add a finite-temperature or anharmonic theory discussion; those remain in the energy/free-energy boundary pages.
- This PR does not add EPC or superconductivity guidance; EPC remains a downstream consumer of the phonon evidence chain.
- External QE documentation links were not live-revalidated in this PR; the source spine remains the maintenance point for external link audits.

## Merge Recommendation

Ready for draft PR review after final command validation. The diff is scoped to three phonon / soft-mode pages plus this report.

## Final QA Gate

| Gate | Result | Evidence |
|---|---|---|
| Physics accuracy | PASS | The pages tie imaginary frequency to negative `omega^2`, distinguish direct `ph.x` q-point evidence from `matdyn.x` interpolation, require eigenvector / displacement-pattern review, and frame ASR as diagnostic rather than repair. |
| Style consistency | PASS | The theory-minimum, judgement and workflow pages keep distinct roles and add explanatory prose plus compact tables rather than checklist-only text. |
| Source boundary | PASS | No new DOI, BibTeX or tutorial-as-authority was added; QE program behavior remains tied to official INPUT references and PHonon guide, with DFPT method context in the existing canonical spine. |
| Output-review framing | PASS | Output evidence now includes perturbation convergence, q-grid / IFC file chain, direct q-point vs interpolation, ASR before/after, eigenvectors and warnings. |
| PASS/WARN/BLOCK preservation | PASS | All three target pages retain explicit `PASS / WARN / BLOCK` sections or decision framing. |
| Local links | PASS | Full local Markdown link check reported `OK local markdown links checked: 141 files`. |
| Private path / document residue | PASS | Target-file scan found no private machine paths or external document residues. |
| Project-scope drift | PASS | Target-file scan found no unrelated tooling drift; content stays within QE-Learning phonon / DFPT scope. |
| Fixed material case scan | PASS | No material-specific case examples were introduced. |
| Universal parameter scan | PASS | No fixed q-grid, cutoff, k-mesh, smearing, ASR or threshold recipe was introduced. |
| High-risk overclaim scan | PASS | Keyword hits for Gamma stability, ASR, imaginary frequency, EPC and superconductivity occur in boundary, misconception or downstream-limitation contexts; no positive overclaim was found. |
| Merge recommendation | READY | No BLOCK gate remains; suitable for draft PR review after fresh GitHub PR creation. |
