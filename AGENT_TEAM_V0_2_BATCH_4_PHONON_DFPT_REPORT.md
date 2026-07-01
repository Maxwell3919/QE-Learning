# Agent Team v0.2 Batch 4 Phonon / DFPT Report

## Scope

Batch 4 completes the phonon / DFPT workflow handbook slice. It focuses on `ph.x`, `q2r.x`, `matdyn.x`, `dynmat.x`-level interpretation, Gamma modes, phonon dispersion, phonon DOS, Born effective charge, dielectric tensor, IR/Raman, and imaginary-frequency triage. It does not expand EPC, Wannier, GW/BSE, MD, NEB, or thermodynamics tutorials.

## Subagents used

| Subagent | Role | Target pages | Output |
|---|---|---|---|
| W4-1 Gamma / dispersion writer | Workflow writer | `gamma-phonon.md`, `phonon-dispersion-dfpt.md` | Added data-chain prose, `fildyn -> flfrc -> flfrq` boundary, and stronger Gamma/full-BZ gates |
| W4-2 Phonon DOS / IR-Raman writer | Workflow writer | `phonon-dos.md`, `ir-raman.md` | Added DOS q-space sampling boundary and IR/Raman response tensor review |
| W4-3 Born-charge / debugging writer | Workflow writer | `dielectric-born-effective-charge.md`, `phonon-debugging.md` | Added response-quantity boundary, tensor symmetry review, and imaginary-frequency triage sequence |
| D4-1 Phonon diagram subagent | Diagram designer | phonon workflow and dynamical-matrix diagrams | Added two original SVG diagrams and page references |
| Main agent | Integration and QA | report, validation, PR integration | Rechecked official QE parameter boundaries, source boundary, links, diagrams, and final gate |

## Research summary

The phonon workflow is a response-calculation data chain rather than an ordinary plotting step. A defensible phonon conclusion depends on the relaxed structure, final static SCF, pseudopotential consistency, electronic convergence, perturbation convergence, q-grid completeness, interpolation file-chain integrity, and post-processing choices such as ASR or non-analytic correction.

## Writing changes

- Expanded [workflows/phonon/gamma-phonon.md](workflows/phonon/gamma-phonon.md) with a clearer `ph.x` / `dynmat.x` ownership boundary, final static SCF prerequisite, `fildyn` provenance, and Gamma-only stability boundary.
- Expanded [workflows/phonon/phonon-dispersion-dfpt.md](workflows/phonon/phonon-dispersion-dfpt.md) with the `ph.x -> q2r.x -> matdyn.x` data chain, `fildyn/flfrc/flfrq` file separation, ASR as a post-processing constraint, and direct q-point / interpolation triage.
- Expanded [workflows/phonon/phonon-dos.md](workflows/phonon/phonon-dos.md) with the distinction between `ph.x` q-grid and `matdyn.x` DOS q mesh, DOS broadening / `deltaE` review, and low-frequency / negative-frequency checks.
- Expanded [workflows/phonon/ir-raman.md](workflows/phonon/ir-raman.md) with response tensor prerequisites, `epsil` / `lraman` execution evidence, mode symmetry / degeneracy review, and finite-temperature spectrum boundaries.
- Expanded [workflows/phonon/dielectric-born-effective-charge.md](workflows/phonon/dielectric-born-effective-charge.md) with Born charge as displacement-polarization response, tensor symmetry / charge-neutrality review, and non-analytic correction file-chain requirements.
- Expanded [workflows/phonon/phonon-debugging.md](workflows/phonon/phonon-debugging.md) with an operational triage order: structure / SCF, `ph.x` response, q-grid and file chain, ASR comparison, direct q-point vs interpolation, eigenvector pattern, and one-variable rerun.

## Diagram changes

- Added [assets/diagrams/workflows/qe-phonon-dfpt-chain.svg](assets/diagrams/workflows/qe-phonon-dfpt-chain.svg), referenced from [workflows/phonon/phonon-dispersion-dfpt.md](workflows/phonon/phonon-dispersion-dfpt.md).
- Added [assets/diagrams/theory-minimum/phonon-dynamical-matrix-picture.svg](assets/diagrams/theory-minimum/phonon-dynamical-matrix-picture.svg), referenced from [theory-minimum/dfpt-phonons.md](theory-minimum/dfpt-phonons.md).
- Both diagrams are original vector schematics and include SVG `<title>` / `<desc>` accessibility text.

## Physics QA findings

- Gamma phonon is explicitly limited to zone-center evidence and no longer reads as full Brillouin-zone stability evidence.
- `ph.x` is consistently described as a DFPT response calculation that reads final static SCF data; normal program termination is not treated as physical reliability.
- `q2r.x` and `matdyn.x` are separated by file-chain boundaries: `fildyn` for dynamical matrices, `flfrc` for IFCs, and `flfrq/fldos` for post-processing outputs.
- ASR is consistently described as a post-processing constraint or diagnostic, not a repair for unrelaxed structures, weak SCF evidence, incomplete q-grids, or mixed files.
- Imaginary-frequency language requires triage with q-position, branch, ASR comparison, direct q-point / interpolation distinction, and eigenvector / displacement-pattern evidence.
- Born effective charge is described as displacement-polarization response, not static valence, oxidation state, or charge partition.
- IR/Raman pages keep harmonic DFPT and response tensor outputs separate from finite-temperature experimental spectra.

## Source QA findings

No new DOI, BibTeX metadata, or external tutorial dependency was added. Parameter definitions remain tied to QE official `INPUT_PH`, `INPUT_Q2R`, `INPUT_MATDYN`, `INPUT_DYNMAT`, and PHonon documentation; canonical DFPT literature supports method boundaries. The main agent spot-checked the newly emphasized `ph.x` and `matdyn.x` field boundaries against the official QE input pages before final validation.

## Validation results

- `git status --short --branch`: branch `docs/v0.2-phonon-dfpt-workflows`; only Batch 4 target files are changed.
- `git diff --check`: passed.
- Markdown local links: passed after final validation.
- Image link/reference scan: passed after final validation.
- SVG XML validation: passed for the two new diagrams.
- Private path / document residue scan: passed on touched Batch 4 files.
- Scope drift, fixed material, universal parameter and high-risk overclaim scans: passed on touched Batch 4 content files.
- Phonon workflow structure scan: passed, all 6 phonon workflow pages contain `Output review` and `PASS / WARN / BLOCK`.

## Remaining gaps

- Batch 5 will revisit `theory-minimum/dfpt-phonons.md` and `theory-minimum/dielectric-born-charge.md` as part of the theory-minimum final pass.
- Batch 6 will revisit `physics-judgement/09-phonons-soft-modes-and-dynamical-stability.md` and `physics-judgement/10-dfpt-response-and-polar-materials.md` as part of the physics-judgement final pass.
- Advanced EPC and finite-temperature thermodynamics remain intentionally outside this batch.

## Final QA Gate

| Gate | Result | Evidence |
|---|---|---|
| Physics accuracy | PASS | Gamma/full-BZ, ASR, imaginary frequency, Born charge, polar response, and harmonic spectrum boundaries are explicit. |
| QE workflow correctness | PASS | `ph.x`, `q2r.x`, `matdyn.x`, `dynmat.x`, `fildyn`, `flfrc`, and `flfrq/fldos` roles are separated. |
| Output-review framing | PASS | Each touched workflow page now orders evidence before interpretation and keeps image/prose aids subordinate to output review. |
| PASS/WARN/BLOCK preservation | PASS | All 6 phonon workflow pages retain `PASS / WARN / BLOCK`; gates were strengthened, not removed. |
| Style consistency | PASS | Pages use reference-handbook prose plus tables; new warning language is concentrated in review, misconceptions, and gate sections. |
| Source boundary | PASS | Field semantics point back to QE official input pages; workflow gates are labelled as internal inference or standards. |
| Diagram validity, if applicable | PASS | Two original SVGs include title/desc, are locally referenced, and contain no material cases or fixed parameters. |
| Local links | PASS | Markdown local-link check passed. |
| Private path / document residue | PASS | No private path or research-document residue found in touched Batch 4 files. |
| Project-scope drift | PASS | No off-scope platform or automation-system content added to handbook正文. |
| Fixed material case scan | PASS | No concrete material examples added. |
| Universal parameter scan | PASS | No fixed q-grid, cutoff, k mesh, smearing, or threshold recommendation added. |
| High-risk overclaim scan | PASS | No Gamma-as-full-stability, ASR-as-repair, Born-charge-as-valence, or harmonic-spectrum-as-experiment claim added. |
| Merge recommendation | READY | Ready after final validation commands pass. |
