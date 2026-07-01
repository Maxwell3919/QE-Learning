# Phase 3-B Model Boundary Judgement Report

## Scope

本轮只处理 `theory-minimum/magnetism-soc-dftu.md` 与一组高风险 `physics-judgement/` 页面。目标是补强 functional、Kohn-Sham eigenvalue、band gap、DFT+U、SOC 和 pseudopotential 相关的模型边界判断，帮助读者区分 numerical error、model error、pseudopotential error、post-processing error 和 workflow propagation error。

本轮没有扩写 ground-state、electronic 或 phonon workflow 正文，也没有新增目录或材料绑定算例。

## Files changed

- `theory-minimum/magnetism-soc-dftu.md`
- `physics-judgement/functional-choice-and-sensitivity.md`
- `physics-judgement/kohn-sham-eigenvalue-boundary.md`
- `physics-judgement/band-gap-problem-and-delocalization.md`
- `physics-judgement/u-value-provenance-and-boundary.md`
- `physics-judgement/soc-and-symmetry-boundary.md`
- `physics-judgement/pseudopotential-transferability-and-functional-consistency.md`
- `PHASE3B_MODEL_BOUNDARY_JUDGEMENT_REPORT.md`

## Theory page upgraded

### `theory-minimum/magnetism-soc-dftu.md`

- 补强 QE input 映射：`nspin`、`starting_magnetization`、`tot_magnetization`、`noncolin`、`lspinorb`、fully relativistic pseudopotential、`HUBBARD` card、`input_dft`、`occupations/smearing/degauss`。
- 补强 output review 映射：spin / noncollinear / SOC 回显、total / absolute magnetization、atomic magnetization、symmetry、loaded pseudopotential、Hubbard setup、occupation matrix 和 warnings。
- 补强 workflow 回链：SCF、NSCF、bands、DOS、PDOS、Fermi surface、phonon 和 advanced workflows 都必须保持同一 spin/SOC/DFT+U 数据链。
- 补强常见误区：初始磁矩当最终磁矩、SOC 后沿用旧 k-path 和简并解释、U 无来源、PP 不支持 SOC、磁性未收敛仍进入 bands/DOS。
- 补强 PASS/WARN/BLOCK：明确何时可进入下游、何时只能探索、何时必须停止。
- 补强 physics-judgement 边界：回链到 functional、PP transferability、U provenance、SOC symmetry 和 band gap 判断页。

## Physics judgement pages upgraded

### `physics-judgement/functional-choice-and-sensitivity.md`

- 增加 functional 切换时的典型 output 现象：XC 回显不一致、结构/磁性/gap/phonon 敏感、下游未重算。
- 区分 numerical error、model error、PP error、boundary error、post-processing error 和 workflow propagation error。
- 补充 `input_dft`、UPF metadata、`vdw_corr`、hybrid 字段、forces/stress/phonon output 的必查项。
- 增加 PASS/WARN/BLOCK 和不能过度解释的结论：functional 不是普适提升，模型差异不能写成普通收敛问题。

### `physics-judgement/kohn-sham-eigenvalue-boundary.md`

- 补强 Kohn-Sham auxiliary eigenvalue 的使用边界。
- 增加 bands/DOS/PDOS 冲突、Fermi level、energy zero、SOC/DFT+U 后简并变化等 output 现象。
- 明确 band path、uniform mesh、PDOS projection 和 energy reference 的证据分工。
- 增加 PASS/WARN/BLOCK：可写 DFT-level 趋势，但不能把普通 KS 图像写成准粒子或光谱结论。

### `physics-judgement/band-gap-problem-and-delocalization.md`

- 补强 fundamental gap、Kohn-Sham gap、optical gap 的边界。
- 增加 delocalization、self-interaction、derivative discontinuity 与 numerical convergence 的区分。
- 补充 bands、DOS、Fermi level、smearing、SOC、U、PP 和 convergence 的必查项。
- 增加 PASS/WARN/BLOCK：DFT-level gap 可带边界使用；定量激发态、光学或器件级结论进入 BLOCK。

### `physics-judgement/u-value-provenance-and-boundary.md`

- 补强 U provenance：literature U、empirical U、linear-response U、DFPT U、self-consistent U 和 U+V 的边界。
- 增加 `HUBBARD` card、legacy DFT+U input、`hp.x`、Hubbard occupations、PP、functional 和 spin state 的审阅项。
- 明确 U 不是元素常数，不能只为匹配 gap 调 U。
- 增加 PASS/WARN/BLOCK：U 来源、projector/manifold、PP、functional、结构状态和下游复算链必须完整。

### `physics-judgement/soc-and-symmetry-boundary.md`

- 补强 SOC 是 relativistic model choice，不是绘图开关。
- 增加 fully relativistic PP、`noncolin`、`lspinorb`、symmetry、irreducible k-points、magnetization、SOC/non-SOC 文件链的审阅项。
- 明确 SOC 会改变 degeneracy、k-path、PDOS 分量、Fermi surface 和高级下游解释。
- 增加 PASS/WARN/BLOCK：PP 不支持 SOC、SOC/non-SOC 文件混用或未重审 symmetry/k-path 时进入 BLOCK。

### `physics-judgement/pseudopotential-transferability-and-functional-consistency.md`

- 补强 PP 是 electron-ion interaction model，而不是普通输入附件。
- 增加 PP/XC mismatch、recommended cutoff、relativistic capability、valence/projector、DFT+U/PDOS/Wannier 相关边界。
- 明确 transferability 不能由 `JOB DONE`、单次 SCF convergence 或单一 total energy 证明。
- 增加 PASS/WARN/BLOCK：PP source、functional consistency、cutoff convergence、target observable 边界不完整时不能进入下游结论。

## Source boundary

本轮使用的来源类别：

- QE official docs 和 `INPUT_PW` reference：用于 `nspin`、`starting_magnetization`、`noncolin`、`lspinorb`、`input_dft`、`HUBBARD` card、`occupations` 等字段边界。
- QE pseudopotential and Hubbard documentation：用于 UPF、SOC-capable PP、Hubbard input 和版本敏感边界。
- Canonical literature：用于 Kohn-Sham eigenvalue、functional error、band-gap problem、delocalization、DFT+U provenance、pseudopotential transferability 和 reproducibility。
- Internal workflow / standards：用于 output review、PASS/WARN/BLOCK、calculation record 和下游准入。

参数定义仍以 QE 官方 `INPUT_*` 为准；文献用于模型边界和结果解释，不替代官方字段定义。

## What was intentionally not changed

- 未处理 vdW / low-dimensional boundary。
- 未处理 work function electrostatic boundary。
- 未处理 Wannier validation。
- 未处理 EPC。
- 未处理 GW/BSE。
- 未处理 thermodynamics。
- 未大规模修改 workflow 正文。
- 未新增材料绑定算例或固定参数建议。

## Follow-up

Phase 3-C 建议目标：

- `physics-judgement/vdw-and-low-dimensional-boundary.md`
- `physics-judgement/work-function-and-electrostatic-boundary.md`
- `physics-judgement/wannier-validation-and-window-choice.md`
- `physics-judgement/epc-data-chain-and-convergence.md`
- `physics-judgement/ground-state-vs-excited-state.md`
- `physics-judgement/energy-vs-free-energy.md`
- `physics-judgement/uncertainty-statement-template.md`
