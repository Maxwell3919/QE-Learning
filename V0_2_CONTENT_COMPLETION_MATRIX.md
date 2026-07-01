# QE-Learning v0.2 Content Completion Matrix

本矩阵用于把 v0.1 reference-handbook foundation 推进到 v0.2 内容完整版本。它不是新目录路线图，而是逐页回填和 QA 的执行清单。

## 状态说明

- `complete`: v0.1 后已经可作为参考手册页面使用，只需要后续链接或风格维护。
- `needs depth`: 需要补充正文式物理图像、判断图像或理论背景。
- `needs QE mapping`: 需要更清楚映射到 QE 程序、input 字段、output 证据或文件链。
- `needs output review`: 需要补强 output evidence 说明。
- `needs P/W/B`: 需要补强 `PASS / WARN / BLOCK` 或下游准入边界。
- `needs source boundary`: 需要说明来源支持什么判断、不支持什么结论。
- `overlap risk`: 与另一页功能接近，需要明确 canonical topic 与 quick review card 的分工。

## Matrix

| File | Role | Status | Missing elements | Batch | Writer subagent | QA focus |
|---|---|---|---|---|---|---|
| `README.md` | Project landing | complete | release navigation only | 9 | W9-1 | scope and handbook entry |
| `HANDBOOK_INDEX.md` | Handbook navigation | complete | update after diagrams/content batches | 9 | W9-1 | route consistency |
| `learn/README.md` | Learn path | complete | final navigation refresh | 9 | W9-1 | capability path |
| `learn/00-start-here.md` | Learn path | complete | optional style polish | 9 | W9-2 | no checklist-only completion |
| `learn/01-qe-workflow-map.md` | Learn path | complete | optional diagram link | 8/9 | W9-2 | workflow-map consistency |
| `learn/02-first-scf-loop.md` | Learn path | complete | optional ground-state diagram link | 8/9 | W9-3 | output evidence route |
| `learn/03-convergence-loop.md` | Learn path | complete | optional convergence diagram link | 8/9 | W9-3 | observable convergence |
| `learn/04-relaxation-loop.md` | Learn path | complete | optional ground-state link refresh | 9 | W9-4 | final SCF boundary |
| `learn/05-electronic-structure-loop.md` | Learn path | complete | optional electronic diagram link | 8/9 | W9-4 | bands/DOS boundary |
| `learn/06-phonon-dfpt-loop.md` | Learn path | complete | optional phonon diagram link | 8/9 | W9-5 | DFPT chain |
| `learn/07-postprocessing-loop.md` | Learn path | complete | optional post-processing diagram link | 8/9 | W9-5 | visualization boundary |
| `learn/08-hpc-and-reproducibility.md` | Learn path | complete | optional record link refresh | 9 | W9-6 | reproducibility |
| `learn/09-feature-expansion-map.md` | Learn path | complete | source/tool boundary refresh | 7/9 | W7-7 | advanced tools not basic path |
| `workflows/README.md` | Workflow index | complete | optional diagram index link | 8/9 | W9-1 | directory responsibility |
| `workflows/ground-state/scf.md` | Workflow reference | complete | optional diagram/caption link | 2/8 | W2-1 | SCF evidence, scratch chain |
| `workflows/ground-state/nscf.md` | Workflow reference | complete | optional NSCF vs bands/DOS cross-link | 2/3 | W2-2 | upstream SCF dependency |
| `workflows/ground-state/relax.md` | Workflow reference | complete | optional final static SCF emphasis | 2 | W2-3 | force convergence |
| `workflows/ground-state/vc-relax.md` | Workflow reference | complete | optional stress/cell evidence refinement | 2 | W2-3 | stress and downstream |
| `workflows/ground-state/cutoff-convergence.md` | Workflow reference | complete | optional observable hierarchy | 2 | W2-4 | numerical convergence |
| `workflows/ground-state/kpoint-convergence.md` | Workflow reference | complete | optional kmesh diagram link | 2/8 | W2-5 | BZ sampling |
| `workflows/ground-state/smearing-and-occupations.md` | Workflow reference | complete | optional smearing/electronic cross-link | 2/3 | W2-5 | metal/small-gap boundary |
| `workflows/electronic/bands.md` | Workflow reference | complete | optional evidence-boundary diagram | 3/8 | W3-1 | k-path vs mesh |
| `workflows/electronic/dos.md` | Workflow reference | complete | optional evidence-boundary diagram | 3/8 | W3-2 | uniform mesh and energy reference |
| `workflows/electronic/pdos.md` | Workflow reference | complete | optional projector boundary emphasis | 3 | W3-3 | PDOS overclaim |
| `workflows/electronic/charge-density.md` | Workflow reference | complete | optional pp.x file-chain diagram link | 3 | W3-4 | visualization vs evidence |
| `workflows/electronic/electrostatic-potential.md` | Workflow reference | complete | optional work-function cross-link | 3 | W3-4 | potential reference |
| `workflows/electronic/elf.md` | Workflow reference | complete | optional source boundary note | 3 | W3-4 | ELF interpretation |
| `workflows/electronic/work-function.md` | Workflow reference | complete | optional plateau diagram link | 3/8 | W3-5 | vacuum plateau and EF |
| `workflows/electronic/fermi-surface.md` | Workflow reference | complete | optional dense mesh emphasis | 3 | W3-6 | smearing/Fermi level |
| `workflows/phonon/gamma-phonon.md` | Workflow reference | complete | optional phonon diagram link | 4/8 | W4-1 | Gamma not full stability |
| `workflows/phonon/phonon-dispersion-dfpt.md` | Workflow reference | complete | optional q2r/matdyn diagram link | 4/8 | W4-2 | q-grid and interpolation |
| `workflows/phonon/phonon-dos.md` | Workflow reference | complete | optional DOS vs dispersion cross-link | 4 | W4-3 | DOS q-sampling |
| `workflows/phonon/dielectric-born-effective-charge.md` | Workflow reference | complete | optional polar-response diagram | 4/8 | W4-4 | tensor evidence |
| `workflows/phonon/ir-raman.md` | Workflow reference | complete | optional response source boundary | 4 | W4-5 | harmonic spectrum boundary |
| `workflows/phonon/phonon-debugging.md` | Workflow reference | complete | optional triage flow diagram | 4/8 | W4-6 | imaginary-frequency triage |
| `workflows/advanced/dft-plus-u.md` | Advanced boundary | complete | optional source-policy refresh | 6/7 | W6-6 | U provenance |
| `workflows/advanced/noncollinear-soc.md` | Advanced boundary | complete | optional SOC chain refinement | 6/7 | W6-7 | SOC and symmetry |
| `workflows/advanced/vdW-corrections.md` | Advanced boundary | complete | optional low-dimensional boundary link | 6/7 | W6-9 | vdW overclaim |
| `workflows/advanced/hybrid-functional-overview.md` | Advanced boundary | complete | optional excited-state boundary link | 6/7 | W6-15 | hybrid not universal |
| `workflows/advanced/wannier90-overview.md` | Advanced boundary | complete | optional Wannier validation cross-link | 6/7 | W6-13 | interpolation evidence |
| `workflows/advanced/electron-phonon-overview.md` | Advanced boundary | complete | optional EPC data-chain cross-link | 6/7 | W6-14 | EPC convergence |
| `workflows/advanced/spin-polarization.md` | Advanced boundary | complete | optional magnetism cross-link | 6/7 | W6-7 | magnetic initialization |
| `workflows/advanced/pseudopotential-generation-overview.md` | Advanced boundary | complete | optional source boundary | 7 | W7-5 | official docs only |
| `workflows/advanced/neb-reaction-path.md` | Advanced boundary | complete | optional free-energy boundary link | 6/7 | W6-16 | MEP vs free energy |
| `workflows/advanced/molecular-dynamics.md` | Advanced boundary | complete | optional finite-T boundary link | 6/7 | W6-16 | MD sampling |
| `theory-minimum/README.md` | Theory index | complete | final navigation refresh | 9 | W9-1 | directory boundary |
| `theory-minimum/dft-ks-scf.md` | Theory background | complete | optional style/source refresh | 5 | W5-1 | ground-state theory |
| `theory-minimum/plane-wave-cutoff.md` | Theory background | complete | optional diagram link | 5/8 | W5-2 | basis convergence |
| `theory-minimum/pseudopotentials.md` | Theory background | complete | optional transferability cross-link | 5 | W5-3 | PP provenance |
| `theory-minimum/functional-choice.md` | Theory background | needs depth | explicit `物理图像`, P/W/B relation, source boundary | 5 | W5-4 | older-format cleanup |
| `theory-minimum/reciprocal-space-and-brillouin-zone.md` | Theory background | complete | optional diagram link | 5/8 | W5-5 | reciprocal-space picture |
| `theory-minimum/crystal-symmetry-space-group-point-group.md` | Theory background | complete | optional terminology polish | 5 | W5-6 | little group / degeneracy |
| `theory-minimum/kpoints-symmetry-kpath.md` | Theory background | complete | optional reduce duplication | 5 | W5-7 | kmesh vs k-path |
| `theory-minimum/band-structure-and-dos.md` | Theory background | complete | optional evidence diagram link | 5/8 | W5-8 | bands/DOS evidence |
| `theory-minimum/smearing-metals.md` | Theory background | complete | optional EPC/Fermi cross-link | 5 | W5-9 | occupation boundary |
| `theory-minimum/force-stress-relaxation.md` | Theory background | complete | optional final SCF link | 5 | W5-10 | force/stress evidence |
| `theory-minimum/dfpt-phonons.md` | Theory background | complete | optional phonon diagram link | 5/8 | W5-11 | dynamical matrix |
| `theory-minimum/dielectric-born-charge.md` | Theory background | complete | optional polar-response diagram link | 5/8 | W5-12 | Born charge response |
| `theory-minimum/magnetism-soc-dftu.md` | Theory background | complete | optional SOC/U cross-link refresh | 5/6 | W5-13 | model choices |
| `physics-judgement/README.md` | Judgement index | overlap risk | quick-card vs numbered-page routing | 6/9 | W6-1 | canonical routing |
| `physics-judgement/CONCLUSION_REFERENCE_BOOK.md` | Judgement reference | overlap risk | keep as conclusion kernel, avoid duplicate正文 | 6 | W6-1 | source boundary |
| `physics-judgement/01-dft-approximation-map.md` | Judgement page | complete | optional diagram link | 6/8 | W6-2 | error taxonomy |
| `physics-judgement/02-exchange-correlation-ladder.md` | Judgement page | complete | optional functional quick-card link | 6 | W6-3 | functional ladder |
| `physics-judgement/03-self-interaction-and-delocalization-error.md` | Judgement page | complete | optional band-gap card link | 6 | W6-4 | delocalization |
| `physics-judgement/04-band-gap-problem-and-derivative-discontinuity.md` | Judgement page | complete | optional duplicate-route cleanup | 6 | W6-5 | gap overclaims |
| `physics-judgement/05-strong-correlation-and-dft-plus-u.md` | Judgement page | complete | optional U quick-card route | 6 | W6-6 | DFT+U boundary |
| `physics-judgement/06-magnetism-spin-and-soc.md` | Judgement page | complete | optional SOC quick-card route | 6 | W6-7 | SOC/magnetism |
| `physics-judgement/07-van-der-waals-and-nonlocal-interactions.md` | Judgement page | complete | optional vdW quick-card route | 6 | W6-8 | vdW model |
| `physics-judgement/08-metals-fermi-surface-and-smearing.md` | Judgement page | complete | optional kmesh quick-card route | 6 | W6-9 | metals/smearing |
| `physics-judgement/09-phonons-soft-modes-and-dynamical-stability.md` | Judgement page | complete | optional triage diagram link | 6/8 | W6-10 | soft modes |
| `physics-judgement/10-dfpt-response-and-polar-materials.md` | Judgement page | complete | optional polar diagram link | 6/8 | W6-11 | polar response |
| `physics-judgement/11-surfaces-interfaces-and-electrostatics.md` | Judgement page | complete | optional work-function quick-card route | 6 | W6-12 | electrostatics |
| `physics-judgement/12-finite-size-boundary-and-dimensionality.md` | Judgement page | complete | optional vdW/low-D link | 6 | W6-13 | finite-size |
| `physics-judgement/13-wannierization-and-effective-model-thinking.md` | Judgement page | complete | optional Wannier quick-card route | 6 | W6-14 | Wannier model |
| `physics-judgement/14-electron-phonon-coupling-physical-judgement.md` | Judgement page | complete | optional EPC quick-card route | 6 | W6-15 | EPC chain |
| `physics-judgement/15-spectroscopy-gw-bse-and-excited-state-boundaries.md` | Judgement page | complete | optional excited-state quick-card route | 6 | W6-16 | GW/BSE boundary |
| `physics-judgement/16-thermodynamics-free-energy-and-md-boundaries.md` | Judgement page | complete | optional free-energy quick-card route | 6 | W6-17 | thermodynamics |
| `physics-judgement/17-uncertainty-error-bars-and-reproducibility.md` | Judgement page | complete | optional uncertainty template route | 6 | W6-18 | uncertainty |
| `physics-judgement/18-physics-judgement-checklist.md` | Judgement checklist | complete | optional diagram link | 6/8 | W6-19 | checklist consistency |
| `physics-judgement/*-boundary.md`, `*-convergence.md`, `*-template.md` | Quick judgement cards | overlap risk | role clarity vs numbered canonical pages | 6 | W6-card team | no duplicate canonical drift |
| `standards/README.md` | Standard index | complete | final navigation refresh | 7/9 | W7-1 | standards scope |
| `standards/pass-warn-block.md` | Standard | needs source boundary | add internal links to output review, record, source policy | 7 | W7-1 | P/W/B canonical semantics |
| `standards/output-review-checklist.md` | Standard | complete | optional cross-link refresh | 7 | W7-2 | evidence tables |
| `standards/calculation-record-template.md` | Standard | complete | optional output-review wording | 7 | W7-3 | reproducibility record |
| `standards/input-file-naming.md` | Standard | needs P/W/B | explicit consequences for prefix/outdir/restart/scratch mix | 7 | W7-4 | naming as evidence |
| `standards/project-layout.md` | Standard | needs P/W/B | exact terminology consistency | 7 | W7-5 | personal project boundary |
| `standards/citation-and-source-policy.md` | Standard | complete | optional source spine cross-link | 7 | W7-6 | source policy |
| `references/README.md` | Source index | complete | optional source map refresh | 7/9 | W7-7 | source roles |
| `references/source-index.md` | Source index | needs QE mapping | map source type to output review/PWB decisions | 7 | W7-8 | source-use boundary |
| `references/canonical-literature.md` | Source reference | complete | optional metadata audit | 7 | W7-9 | A0/A1/A2 levels |
| `references/canonical.bib` | BibTeX source | complete | duplicate-key check | 7/9 | W7-9 | no invented metadata |
| `references/reading-maintenance-policy.md` | Source standard | complete | optional v0.2 update | 7 | W7-10 | maintenance policy |
| `references/tools/*.md` | Tool cards | needs QE mapping | QE file-chain/downstream boundary in short cards | 7 | W7-tool team | tool not basic path |
| `references/tutorial-sites/*.md` | Tutorial cards | needs source boundary | what supports / what cannot prove | 7 | W7-tutorial team | tutorial not official INPUT |
| `assets/diagrams/README.md` | Diagram policy | missing | create diagram source-boundary policy | 8 | D8-0 | original diagrams only |
| `assets/diagrams/workflows/*.svg` | Diagrams | missing | workflow learning diagrams | 8 | D8-1 | command/file-chain accuracy |
| `assets/diagrams/theory-minimum/*.svg` | Diagrams | missing | theory learning diagrams | 8 | D8-2 | no misleading physics |
| `assets/diagrams/physics-judgement/*.svg` | Diagrams | missing | judgement diagrams | 8 | D8-3 | no overclaim arrows |
| `V0_2_AGENT_TEAM_COMPLETION_REPORT.md` | v0.2 final report | missing | cumulative batch summary | 1/9 | A3/W9-final | merge readiness |
| `RELEASE_NOTES_v0.2.md` | v0.2 release note | missing | release delta and validation | 9 | W9-final | release boundary |

## Batch allocation summary

| Batch | Scope | Main risk to control |
|---|---|---|
| 2 | Ground-state workflow completion | `JOB DONE` and SCF convergence overclaim; parameter recipes |
| 3 | Electronic workflow completion | k-path vs uniform mesh; DFT gap overclaim; PDOS overinterpretation |
| 4 | Phonon / DFPT workflow completion | Gamma vs full dispersion; ASR misuse; imaginary-frequency triage |
| 5 | Theory-minimum final deepening | textbook drift; missing QE input/output mapping |
| 6 | Physics-judgement final deepening | numbered vs quick-card overlap; model vs numerical error |
| 7 | Standards / records / source-policy completion | bureaucratic wording; source without supported-use boundary |
| 8 | Diagram integration | decorative or misleading diagrams; unreferenced assets |
| 9 | Navigation / release finalization | stale status pages; broken cross-links; release overclaim |
