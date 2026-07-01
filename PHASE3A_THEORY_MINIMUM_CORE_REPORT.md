# Phase 3-A Theory Minimum Core Report

## Scope

本轮只处理 `theory-minimum/` 中与 ground-state、electronic 和 phonon 三大 workflow 直接相关的核心理论页，并最小更新 `theory-minimum/README.md`。目标是把理论页补成 QE input/output 可回查层，而不是扩写 DFT 教材、重写 workflow 正文或展开 physics-judgement 高级判断。

## Files changed

- `theory-minimum/README.md`
- `theory-minimum/dft-ks-scf.md`
- `theory-minimum/plane-wave-cutoff.md`
- `theory-minimum/pseudopotentials.md`
- `theory-minimum/kpoints-symmetry-kpath.md`
- `theory-minimum/smearing-metals.md`
- `theory-minimum/force-stress-relaxation.md`
- `theory-minimum/dfpt-phonons.md`
- `theory-minimum/dielectric-born-charge.md`
- `PHASE3A_THEORY_MINIMUM_CORE_REPORT.md`

## Theory pages upgraded

### `theory-minimum/dft-ks-scf.md`

- 补强 DFT ground-state、Kohn-Sham auxiliary system、SCF density loop 和 exchange-correlation approximation 的最低边界。
- 映射 `pw.x`、`calculation='scf'`、`input_dft`、`conv_thr`、mixing、occupation、`prefix/outdir` 到 output review。
- 增加 Kohn-Sham eigenvalue、`JOB DONE`、total energy 和 SCF convergence 的常见误区。
- 回链到 SCF、NSCF、bands、phonon、output review checklist 和 physics-judgement ground-state/eigenvalue 边界。

### `theory-minimum/plane-wave-cutoff.md`

- 将 `ecutwfc` 和 `ecutrho` 定位为系统性 numerical convergence 参数。
- 明确 PP 类型、density cutoff、FFT grid、force、stress、phonon 和 work function 的敏感性。
- 强调 cutoff 不存在材料无关固定值，不能只看 single-point total energy。
- 回链到 cutoff convergence、SCF、phonon dispersion 和 output review checklist。

### `theory-minimum/pseudopotentials.md`

- 将 pseudopotential 定位为 model choice 与 numerical boundary 的共同来源。
- 补充 `ATOMIC_SPECIES`、`.UPF`、`pseudo_dir`、`input_dft`、`ecutwfc/ecutrho`、SOC、DFT+U 和 projector 相关边界。
- 强调 PP provenance、functional consistency、transferability 和 output loading review。
- 回链到 calculation record、SCF、cutoff convergence、phonon 和 physics-judgement PP transferability 页面。

### `theory-minimum/kpoints-symmetry-kpath.md`

- 区分 uniform k mesh、irreducible k-points 和 high-symmetry k-path。
- 明确 band path 不是 BZ integration，不能替代 DOS 或 k-point convergence。
- 补充 symmetry、`nosym/noinv`、SOC/magnetism、cell convention 和 k-path 来源边界。
- 回链到 k-point convergence、bands、DOS、Fermi surface 和 phonon dispersion。

### `theory-minimum/smearing-metals.md`

- 将 smearing 定位为 occupation 和 BZ integration 的数值策略，而不是材料真实热状态。
- 映射 `occupations`、`smearing`、`degauss`、`K_POINTS`、`dos.x` broadening 和 Fermi energy。
- 强化金属、小带隙、DOS、Fermi surface、phonon/EPC 对 smearing/k mesh 的敏感性。
- 回链到 smearing workflow、DOS、Fermi surface、phonon dispersion 和 kmesh-smearing physics-judgement。

### `theory-minimum/force-stress-relaxation.md`

- 补强 force、stress、relax、vc-relax 和 final static SCF 的最低理论边界。
- 映射 `forc_conv_thr`、`press_conv_thr`、`ion_dynamics`、`cell_dynamics`、`cell_dofree`、final coordinates/cell 到 output review。
- 强调 relax/vc-relax output 不等于最终 electronic structure。
- 回链到 relax、vc-relax、SCF、Gamma phonon 和 relax boundary physics-judgement。

### `theory-minimum/dfpt-phonons.md`

- 将 DFPT phonon 定位为 response calculation，不是普通后处理。
- 映射 `ph.x`、`tr2_ph`、`fildyn`、`ldisp`、`nq1/nq2/nq3`、`q2r.x`、`matdyn.x`、`dynmat.x`。
- 强调 phonon 可信度依赖结构、SCF、cutoff、k mesh、smearing、q-grid 和 response convergence 全链条。
- 回链到 Gamma phonon、phonon dispersion、phonon debugging 和 soft-mode physics-judgement。

### `theory-minimum/dielectric-born-charge.md`

- 补强 dielectric tensor 和 Born effective charge 作为 response quantities 的最低概念。
- 映射 `epsil=.true.`、`trans`、`tr2_ph`、`fildyn`、`asr/zasr` 和 non-analytic correction。
- 强调 Born charge 不是静态离子价，tensor symmetry、response convergence 和文件链必须审阅。
- 回链到 dielectric/Born workflow、IR/Raman、Gamma phonon 和 polar-response physics-judgement。

## Source boundary

本轮使用的来源类别：

- QE official docs 和 `INPUT_*` references：用于 `pw.x`、`ph.x`、`q2r.x`、`matdyn.x`、`dynmat.x` 参数定义。
- canonical literature：用于 DFT、Kohn-Sham、Monkhorst-Pack、smearing、pseudopotential、DFPT、Born charge 和 reproducibility 的方法边界。
- internal workflow / standards：用于 PASS / WARN / BLOCK、output review、calculation record 和 workflow 回链。

参数定义以 QE 官方 `INPUT_*` 为准；canonical literature 用于理论边界和解释框架，不替代版本敏感字段。

## What was intentionally not changed

- 未深入处理 `theory-minimum/magnetism-soc-dftu.md`。
- 未大规模修改 `physics-judgement/` 正文。
- 未大规模修改 `workflows/` 正文。
- 未处理 advanced workflows、EPC、Wannier、GW/BSE、MD、NEB 或 thermodynamics。
- 未写材料无关的 cutoff、k mesh、degauss 或 q-grid 数值建议。
- 未把 theory-minimum 写成完整 DFT 教材。

## Follow-up

Phase 3-B 建议目标：

- `theory-minimum/magnetism-soc-dftu.md`
- `physics-judgement/functional-choice-and-sensitivity.md`
- `physics-judgement/band-gap-problem-and-delocalization.md`
- `physics-judgement/kohn-sham-eigenvalue-boundary.md`
- `physics-judgement/u-value-provenance-and-boundary.md`
- `physics-judgement/soc-and-symmetry-boundary.md`
- `physics-judgement/pseudopotential-transferability-and-functional-consistency.md`
