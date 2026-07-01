# Phase 3-C Advanced Boundary Judgement Report

## Scope

本轮只处理 `physics-judgement/` 中剩余高风险判断页，目标是补齐从 ground-state、electronic、phonon workflow 进入高级解释时最容易越界的结果可信度判断。修改集中在 vdW/低维边界、work function/electrostatics、Wannier validation、EPC 数据链、ground-state 与 excited-state 边界、energy 与 free energy 边界、不确定性陈述模板。

本轮没有扩写完整 Wannier、EPW、GW/BSE、有限温热力学或高级工具教程，也没有大规模修改 `workflows/` 正文。

## Files changed

- `physics-judgement/vdw-and-low-dimensional-boundary.md`
- `physics-judgement/work-function-and-electrostatic-boundary.md`
- `physics-judgement/wannier-validation-and-window-choice.md`
- `physics-judgement/epc-data-chain-and-convergence.md`
- `physics-judgement/ground-state-vs-excited-state.md`
- `physics-judgement/energy-vs-free-energy.md`
- `physics-judgement/uncertainty-statement-template.md`
- `PHASE3C_ADVANCED_BOUNDARY_JUDGEMENT_REPORT.md`

## Physics judgement pages upgraded

### `vdw-and-low-dimensional-boundary.md`

- 补充了层间距、吸附高度、work function、phonon soft branch 和 downstream model mismatch 等典型 output 现象。
- 区分了 numerical error、model error、pseudopotential error、boundary-condition error、post-processing error、workflow propagation error 和真实弱相互作用物理。
- 增加 `input_dft`、`vdw_corr`、PP、cell/vacuum、dipole/isolated boundary、force/stress、`prefix/outdir` 的必查项。
- 明确 PASS/WARN/BLOCK：同一模型与边界链完整才允许进入下游；vdW 与非 vdW 混用或低维 electrostatics 未审阅进入 BLOCK。
- 写清不能把某个 vdW correction 当作所有性质上的普适改进，也不能只凭单点 total energy 给弱相互作用定量结论。

### `work-function-and-electrostatic-boundary.md`

- 补充 vacuum plateau、Fermi energy、planar average、potential slope、`pp.x` 后处理等 output 现象。
- 区分 potential zero/reference、slab asymmetry、vacuum/dipole boundary、smearing/Fermi level 敏感性和 visualization artifact。
- 增加 SCF、slab direction、`plot_num`、average direction、Fermi energy、dipole/isolated boundary 与 warning 的审阅链。
- 明确无 vacuum plateau、Fermi level 未记录、energy reference 不明或从图像直接读数时进入 BLOCK。
- 强化 work function 需要同一 SCF -> `pp.x` -> average 数据链。

### `wannier-validation-and-window-choice.md`

- 补充 Wannier bands 与 QE bands 不匹配、spread 较小但目标 bands 错位、frozen/outer window 不清、SOC/spinor 链混乱等现象。
- 区分 projection/window/disentanglement 问题、target subspace 错位、SOC/non-SOC 数据混用和 downstream observable 要求。
- 增加 upstream SCF/NSCF、target band manifold、projection、frozen window、outer window、spread、interpolated-vs-ab-initio bands 和 interface files 的审阅项。
- 明确 small spread 不能替代 band interpolation validation；未验证 bands 时 EPC、transport、topology 进入 BLOCK。

### `epc-data-chain-and-convergence.md`

- 补充 `lambda`、`Tc`、alpha2F、linewidth、imaginary phonon、Wannier validation 和 EPW provenance 的 output 现象。
- 区分 electronic convergence、phonon convergence、k/q-grid convergence、smearing/broadening、Wannier interpolation、polar correction、Fermi surface、PP/SOC/U/functional consistency。
- 增加 SCF、NSCF、`ph.x`、`fildvscf`、`electron_phonon`、Wannier、EPW grids 与 EPC observable 的必查链。
- 明确 EPC 不能只看最终数值；phonon 不可信、Wannier 未验证或 grid/broadening 未收敛时进入 BLOCK。

### `ground-state-vs-excited-state.md`

- 补充 bands、DOS peak、hybrid/scissor、spectra 对 empty states/window/broadening 敏感、Yambo/GW/BSE/TDDFT 正常结束但仍需审阅等现象。
- 区分 DFT-level band structure、quasiparticle band structure、optical gap、exciton、absorption spectrum、DOS 与 spectral function。
- 增加 SCF、bands/DOS/PDOS、`nbnd`、k mesh/path、energy reference、advanced tool input 和 broadening/window 的审阅项。
- 明确 ground-state bands/DOS 不能直接支持 quasiparticle、optical、exciton 或 device-level excited-state 结论。

### `energy-vs-free-energy.md`

- 补充 total energy 差异接近数值误差、pressure/stress 未稳、phonon imaginary modes、NEB barrier、短 MD trajectory 等现象。
- 区分 0 K electronic total energy、static lattice enthalpy、harmonic vibrational free energy、QHA、finite-temperature Gibbs/Helmholtz free energy、anharmonic/MD/free-energy sampling。
- 增加 total energy、force/stress/pressure、phonon frequencies/DOS、imaginary modes、MD settings、NEB images 和 model consistency 的审阅项。
- 明确单点电子能量不能直接写成有限温自由能；未解释 imaginary modes 不能做 harmonic free energy。

### `uncertainty-statement-template.md`

- 补充 uncertainty 来源分类：numerical、model、pseudopotential、structure、kmesh/cutoff/smearing、q-grid、boundary condition、post-processing、workflow provenance。
- 增加可复用 statement 模板：energy difference、band/gap、DOS/PDOS、phonon stability、work function、DFT+U/SOC、vdW/low-dimensional、EPC、free energy。
- 每个模板都包含 calculation level、input model、convergence evidence、output evidence、limitation 和 downstream permission。
- 明确缺少 evidence 的陈述只能作为 WARN 或 BLOCK，不进入正式科研结论。

## Source boundary

本轮使用的来源类别：

- QE official docs：`INPUT_PW`、`INPUT_PH`、`INPUT_PP`、PostProc 文档。
- Tool official docs：Wannier90 documentation、EPW documentation、Yambo educational wiki。
- Canonical literature：vdW-DF / dispersion correction、MLWF 与 disentanglement、EPC review、HK/KS/Kohn、Onida-Reining-Rubio excited-state review、DFPT phonon review、DFT reproducibility literature。
- Internal workflow / standards：`workflows/` 主线页、`standards/output-review-checklist.md`、`standards/calculation-record-template.md`、`references/canonical-literature.md`。

参数定义和工具行为仍以官方文档为准；canonical literature 只用于方法背景和判断边界。

## What was intentionally not changed

- 没有写完整 Wannier tutorial。
- 没有写完整 EPW tutorial。
- 没有写完整 GW/BSE tutorial。
- 没有写完整 finite-temperature thermodynamics tutorial。
- 没有大规模修改 `workflows/` 正文。
- 没有新增材料绑定算例或固定参数建议。

## Follow-up

建议下一阶段进入 Phase 4：`references/source-index.md` 与 `references/canonical-literature.md` 的 source spine consolidation。重点是统一前面各阶段用到的官方文档、工具文档和 canonical literature，减少重复来源、修正断链，并把 stable、boundary、version-sensitive 的引用边界写得更清楚。
