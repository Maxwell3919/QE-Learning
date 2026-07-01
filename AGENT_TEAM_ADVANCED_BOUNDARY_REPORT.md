# Agent Team Advanced Boundary Report

## Scope

This PR is the Batch F topic-pack for Issue #4. It refines advanced-chain judgement pages:

- `physics-judgement/wannier-validation-and-window-choice.md`
- `physics-judgement/epc-data-chain-and-convergence.md`
- `physics-judgement/ground-state-vs-excited-state.md`
- `physics-judgement/energy-vs-free-energy.md`
- `physics-judgement/uncertainty-statement-template.md`

No full Wannier, EPW, GW/BSE, thermodynamics or uncertainty-method tutorial was added.

## Subagents Used

| Subagent | Role | Target pages | Output |
|---|---|---|---|
| R-F1 Wannier Research | Research | `wannier-validation-and-window-choice.md` | Brief on projections, windows, spread, disentanglement and ab initio band comparison |
| R-F2 EPC Research | Research | `epc-data-chain-and-convergence.md` | Brief on SCF -> DFPT -> Wannier -> dense k/q chain and EPC observables |
| R-F3 Excited-state Research | Research | `ground-state-vs-excited-state.md` | Brief on DFT ground state, quasiparticles, optical excitations, GW/BSE/TDDFT and scissor boundary |
| R-F4 Free-energy Research | Research | `energy-vs-free-energy.md` | Brief on total energy, enthalpy, harmonic free energy, QHA, anharmonicity and finite-temperature stability |
| R-F5 Uncertainty Research | Research | `uncertainty-statement-template.md` | Brief on claim strength, convergence evidence, model boundary and publication wording |
| W-F1 Wannier Writer | Writer | `wannier-validation-and-window-choice.md` | Added Fermi-window / band-crossing validation language |
| W-F2 EPC Writer | Writer | `epc-data-chain-and-convergence.md` | Added mode-/q-resolved EPC evidence and `Tc` boundary language |
| W-F3 Excited-state Writer | Writer | `ground-state-vs-excited-state.md` | Added quasiparticle/optical/electron-hole interaction hierarchy |
| W-F4 Free-energy Writer | Writer | `energy-vs-free-energy.md` | Added energy-difference uncertainty and vibrational contribution boundaries |
| W-F5 Uncertainty Writer | Writer | `uncertainty-statement-template.md` | Added claim-strength wording table |
| Q-F1 Advanced Physics QA | QA | all five pages | Checked high-risk advanced-chain overclaims |
| Q-F2 Style / Source QA | QA | all five pages | Checked scope, style and source boundary |

## Research Summary

The team converged on a shared advanced-chain rule: downstream scalar outputs are not sufficient evidence. Wannier spreads, EPC `lambda`, KS bands, total energy and generic uncertainty disclaimers all require a data-chain and claim-strength context before they can support scientific conclusions.

## Writing Changes

- Added Fermi-window, band-crossing and near-degeneracy checks to Wannier validation.
- Added mode-/q-resolved EPC evidence and clarified that `Tc` depends on the chosen theory and additional parameters, not only `lambda`.
- Clarified that ground-state DFT, quasiparticle methods and optical methods answer different questions; scissor corrections do not repair wavefunctions, screening or excitonic effects.
- Added energy-difference versus numerical/model uncertainty language and zero-point / vibrational entropy boundary language.
- Added a PASS/WARN/BLOCK-to-wording table so uncertainty statements are actionable rather than boilerplate.

## Physics QA Findings

No BLOCK-level physics issue remains after integration.

Accepted boundaries:

- Small Wannier spread is not treated as sufficient validation.
- Coarse-grid or integrated EPC output is not treated as quantitative transport / superconductivity evidence.
- Ground-state DFT is not written as an excited-state method.
- KS eigenvalues are not written as quasiparticle excitations.
- Static total energy is not written as finite-temperature free energy.
- Uncertainty language is tied to evidence and claim strength.

## Style QA Findings

The edits are targeted and preserve existing judgement-page structure. PASS/WARN/BLOCK remains a gate after explanatory context.

## Source QA Findings

No new DOI, BibTeX, tutorial-as-authority or material-specific source was introduced. Tool behavior remains tied to official tool documentation; physical-method boundaries remain tied to the existing canonical literature spine.

## Validation Results

Validation commands were run after integration:

- `git diff --check`
- full Markdown local-link check
- private-path and document-residue scan
- project-scope drift scan
- fixed material case and universal-parameter scan
- high-risk overclaim scan for Wannier spread, EPC scalar outputs, excited-state claims, free-energy claims and uncertainty boilerplate
- PASS / WARN / BLOCK preservation check

## Remaining Gaps

- This PR does not add full advanced workflow tutorials.
- External tool documentation links were not live-revalidated in this PR.
- Quantitative uncertainty propagation remains outside the current handbook scope.

## Merge Recommendation

Ready for draft PR review after final command validation. The diff is scoped to five advanced-chain judgement pages plus this report.

## Final QA Gate

| Gate | Result | Evidence |
|---|---|---|
| Physics accuracy | PASS | Wannier spread, EPC scalar outputs, KS bands, total energy and uncertainty statements are all tied to data-chain and claim-strength evidence rather than treated as standalone proof. |
| Style consistency | PASS | Edits are local, explanatory and preserve existing judgement-page structure without expanding into advanced tutorials. |
| Source boundary | PASS | No new DOI, BibTeX or tutorial-as-authority was added; tool behavior remains tied to official docs and physical boundaries to the canonical source spine. |
| Output-review framing | PASS | Added checks for Fermi-window band validation, q/mode-resolved EPC, independent-particle optical boundary, energy-difference uncertainty and claim-strength wording. |
| PASS/WARN/BLOCK preservation | PASS | All five target pages retain explicit PASS/WARN/BLOCK sections. |
| Local links | PASS | Full local Markdown link check reported `OK local markdown links checked: 144 files`. |
| Private path / document residue | PASS | Target-file scan found no private machine paths or external document residues. |
| Project-scope drift | PASS | Target-file scan found no unrelated tooling drift; content stays within QE-Learning advanced-boundary scope. |
| Fixed material case scan | PASS | No material-specific case examples were introduced. |
| Universal parameter scan | PASS | No fixed Wannier window, EPC grid, broadening, `Tc`, free-energy or uncertainty threshold recipe was introduced. |
| High-risk overclaim scan | PASS | Regex hits for spread sufficiency, KS quasiparticle claims, total-energy/free-energy and generic proof wording occur in boundary, misconception or report contexts; no positive overclaim was found. |
| Merge recommendation | READY | No BLOCK gate remains; suitable for draft PR creation and ready/merge after GitHub mergeability check. |
