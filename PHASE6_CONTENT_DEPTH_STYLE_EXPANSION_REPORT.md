# Phase 6 Content Depth and Style Expansion Report

## Merge / revert state

- PR #1 `Build QE-Learning reference-handbook foundation` is present on `origin/main`.
- `origin/main` HEAD at Phase 6 start: `5268a50679d8ca4aac381f912028d80c5a5b598a`.
- A revert branch exists at `origin/revert-1-docs/phase5-standards-qa-final-integration`, but its revert commit `1dc34f6` is not an ancestor of `origin/main`.
- Remote revert detected on `main`: no.
- Revert-the-revert executed: no.
- Phase 6 branch: `docs/phase6-content-depth-style-expansion`, created from `origin/main` after resetting local `main` to the remote merge commit.
- Uncommitted changes before Phase 6 writing: none after branch creation.

## Scope

This phase addresses content depth and handbook style after the Phase 0-5 reference-handbook foundation. The work focused on:

- expanding pages that were structurally correct but too checklist-like;
- adding physical pictures to core theory and judgement pages;
- strengthening DFT, reciprocal-space, symmetry, bands/DOS, phonon and response background;
- reducing defensive opening formulations and moving boundary language toward later judgement sections;
- keeping `PASS / WARN / BLOCK`, output review and source boundaries intact.

This phase did not restructure directories, remove Phase 0-5 reports, rewrite workflows, add concrete material cases, or introduce fixed parameter recommendations.

## Files changed

- `PHASE6_CONTENT_DEPTH_STYLE_EXPANSION_REPORT.md`
- `physics-judgement/01-dft-approximation-map.md`
- `physics-judgement/04-band-gap-problem-and-derivative-discontinuity.md`
- `physics-judgement/09-phonons-soft-modes-and-dynamical-stability.md`
- `physics-judgement/10-dfpt-response-and-polar-materials.md`
- `physics-judgement/13-wannierization-and-effective-model-thinking.md`
- `physics-judgement/14-electron-phonon-coupling-physical-judgement.md`
- `theory-minimum/README.md`
- `theory-minimum/dft-ks-scf.md`
- `theory-minimum/plane-wave-cutoff.md`
- `theory-minimum/pseudopotentials.md`
- `theory-minimum/kpoints-symmetry-kpath.md`
- `theory-minimum/smearing-metals.md`
- `theory-minimum/force-stress-relaxation.md`
- `theory-minimum/magnetism-soc-dftu.md`
- `theory-minimum/dfpt-phonons.md`
- `theory-minimum/dielectric-born-charge.md`
- `theory-minimum/reciprocal-space-and-brillouin-zone.md`
- `theory-minimum/band-structure-and-dos.md`
- `theory-minimum/crystal-symmetry-space-group-point-group.md`

## Pages expanded

### `physics-judgement/01-dft-approximation-map.md`

- Expanded the DFT error-source physical picture from a short list into a layered review map.
- Added the review diagram:

```text
DFT result
  = physical model
  + numerical representation
  + pseudopotential model
  + boundary condition
  + workflow data chain
  + post-processing interpretation
```

- Clarified that this is a review classification rather than a strict mathematical decomposition.
- Added more concrete routing examples for SCF oscillation, band gap anomalies, phonon imaginary frequencies and work-function drift.
- Added links to the new reciprocal-space, bands/DOS and symmetry background pages.

### `theory-minimum/dft-ks-scf.md`

- Added a physical picture for the many-electron problem, Born-Oppenheimer context, Hohenberg-Kohn density view, Kohn-Sham auxiliary system and SCF loop.
- Added a minimal mathematical structure for `H_KS[n]`, charge density iteration and the different status of total energy, density and eigenvalues.
- Rewrote the opening to use positive handbook framing instead of defensive limitation language.
- Added a回链 to `band-structure-and-dos.md`.

### `theory-minimum/kpoints-symmetry-kpath.md`

- Added reciprocal-space physical background: Bloch theorem, Brillouin zone, uniform mesh, high-symmetry path and symmetry reduction.
- Added a minimal mathematical structure for reciprocal vectors, BZ integration and k-path sampling.
- Added minimum context for point group, space group, reciprocal symmetry, little group, degeneracy and why SOC/magnetism can change symmetry interpretation.
- Added回链 to the three new theory pages.

### `theory-minimum/plane-wave-cutoff.md`

- Added a physical picture of plane waves as Fourier resolution and cutoff as the shortest represented wavelength scale.
- Added minimal equations for plane-wave expansion and the relation between `ecutwfc`, `ecutrho`, FFT grid and target-observable convergence.
- Clarified why force, stress, phonon, Born charge and work function may remain sensitive after total energy appears stable.

### `theory-minimum/pseudopotentials.md`

- Added a physical picture of core/valence separation, all-electron versus pseudopotential representations and the efficiency/reliability tradeoff.
- Added minimum context for norm-conserving, ultrasoft and PAW pseudopotentials without ranking them as universal quality levels.
- Added a transferability-oriented model map linking UPF metadata, valence configuration, relativistic treatment, functional consistency and downstream observables.

### `theory-minimum/dfpt-phonons.md`

- Added a physical picture of lattice vibrations as normal modes on the Born-Oppenheimer energy surface.
- Added harmonic approximation, force constants, dynamical matrix and `omega^2` curvature interpretation.
- Strengthened the explanation of q-grid, IFC, `q2r.x`, `matdyn.x`, imaginary frequencies and ASR boundaries.

### `theory-minimum/dielectric-born-charge.md`

- Added a physical picture for polarization response, dielectric tensor, Born effective charge, LO-TO splitting and non-analytic correction.
- Added minimal response derivatives for `Z*` and `epsilon`.
- Clarified why Born effective charge is dynamic response evidence rather than static ionic valence.

### P1 theory pages

- `smearing-metals.md`: added the physical picture of Fermi-level discontinuity, finite k mesh and smoothing of occupations.
- `force-stress-relaxation.md`: added the energy-surface picture behind force, stress, `relax` and `vc-relax`.
- `magnetism-soc-dftu.md`: added physical context for spin density, SOC as relativistic Hamiltonian change and DFT+U as localized-manifold model correction.

### P1 physics-judgement pages

- `04-band-gap-problem-and-derivative-discontinuity.md`: expanded the distinction among Kohn-Sham gap, fundamental gap, optical gap, derivative discontinuity and delocalization/self-interaction errors.
- `09-phonons-soft-modes-and-dynamical-stability.md`: expanded `omega^2`, curvature, acoustic artifacts, soft modes, eigenvectors and ASR boundaries.
- `10-dfpt-response-and-polar-materials.md`: expanded electric-field response, Born charge, dielectric tensor and LO-TO physical background.
- `13-wannierization-and-effective-model-thinking.md`: expanded Bloch/Wannier representation, gauge/subspace choice, isolated versus entangled band windows and validation criteria.
- `14-electron-phonon-coupling-physical-judgement.md`: expanded EPC matrix-element, scattering, dense k/q mesh, Wannier/EPW and observable-chain explanation.

## New pages

### `theory-minimum/reciprocal-space-and-brillouin-zone.md`

Added to give a continuous explanation of reciprocal lattice, Bloch theorem, Brillouin zone, k/q points, BZ integration, high-symmetry paths and QE output evidence.

### `theory-minimum/band-structure-and-dos.md`

Added to explain `E_n(k)`, band-path visualisation, DOS as BZ statistics, PDOS projection boundaries, Fermi level, VBM/CBM, energy zero and bands/DOS cross-review.

### `theory-minimum/crystal-symmetry-space-group-point-group.md`

Added to explain space group, point group, reciprocal symmetry, little group, irreducible k-points, degeneracy, and why relax/SOC/magnetism can require symmetry and k-path re-review.

All three pages were registered in `theory-minimum/README.md` and linked from the relevant existing pages.

## Style cleanup

- Page openings were adjusted toward positive definitions and handbook framing.
- `## 物理图像` now appears in all P0 and selected P1 pages touched in this phase.
- Several numbered judgement pages now use an explicit `## 物理图像` heading to make physical explanation visible during reading.
- Boundary warnings such as “不要把...” remain mainly in `常见误区`, `PASS / WARN / BLOCK` and judgement-boundary sections.
- The expanded pages keep review tables, but no longer rely on checklist structure as the primary explanation.

## Remaining gaps

- Many non-target `physics-judgement/` pages still use compact judgement-page style and could benefit from a later readability pass.
- Some advanced pages still depend on source-level descriptions rather than full expert commentary; they remain appropriate for boundary references, not tutorials.
- External source metadata was not reverified in this phase; Phase 4 source spine remains the current source boundary.
- Further expert review would be useful for topology/SOC-specific wording, advanced EPC interpretation and finite-temperature free-energy language.

## Validation

Validation was run after writing and before commit:

- `git status --short --branch`
- `git diff --check`
- Markdown local link check
- Private path and Word-reference residue scan
- automation/tool drift wording scan
- fixed material case scan
- universal-parameter wording scan
- defensive wording density scan for `不是` / `而是` / `不要把`
- P0/P1 `## 物理图像` presence check
- P0 page length check
- forbidden directory check

Results:

- `git diff --check`: passed.
- Markdown local link check: passed for 138 Markdown files.
- Modified/new-file scan for private paths, Word reference filenames and automation/tool drift wording: no hits.
- Full-repository Word-reference scan still reports Phase 0 planning/intake documents that intentionally preserve research-input filenames; these are not handbook正文 references and were not introduced in Phase 6.
- Fixed-material-case / universal-parameter scan: no new Phase 6 hits; full-repository hits are existing boundary-policy sentences or forbidden-claim examples.
- High-risk overclaim scan: hits are explicit warnings such as `JOB DONE` not being `PASS`, PDOS not being a unique chemical decomposition, and U not being an element constant.
- Defensive wording density in `theory-minimum/` and `physics-judgement/`: `不是` 93, `而是` 9, `不要把` 0. This phase reduced opening-template usage in touched pages but did not attempt a repo-wide rewrite of established boundary sections.
- P0/P1 target pages touched in this phase all contain an exact `## 物理图像` heading.
- P0 pages and new theory pages exceed the requested handbook-depth floor by character count.
- Forbidden directories check: no `cases/`, `archive/`, `structure-learning/`, `examples/`, or `tutorial-cases/` directory found at the checked depth.

## Merge recommendation

Recommended for PR review after validation passes.

Suggested PR:

- Base: `main`
- Compare: `docs/phase6-content-depth-style-expansion`
- Title: `Expand QE-Learning theory depth and handbook style`
