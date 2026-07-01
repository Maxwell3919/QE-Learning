# Phase 2-A Ground-State Workflow Report

## Scope

本轮只处理 `workflows/ground-state/` 中的主线 ground-state workflow 页面，并最小范围补强 `standards/output-review-checklist.md` 中与 ground-state 相关的 output review 条目。

本轮目标是把 SCF、NSCF、cutoff convergence、k-point convergence、smearing/occupations、relax 和 vc-relax 写成可审查、可复查、可作为参考手册阅读的 QE 原生命令行 workflow 页面。它不进入 electronic、phonon、DFPT、Wannier、EPC 或 GW/BSE 相关页面。

## Files changed

- `workflows/ground-state/scf.md`
- `workflows/ground-state/nscf.md`
- `workflows/ground-state/cutoff-convergence.md`
- `workflows/ground-state/kpoint-convergence.md`
- `workflows/ground-state/smearing-and-occupations.md`
- `workflows/ground-state/relax.md`
- `workflows/ground-state/vc-relax.md`
- `standards/output-review-checklist.md`

## Workflow pages upgraded

### `workflows/ground-state/scf.md`

- 补强 SCF 的页面定位：SCF 建立 ground-state charge density，不等于完成所有物理分析。
- 补强 `prefix/outdir`、`pseudo_dir`、cutoff、k-points、occupations、smearing、`conv_thr`、`electron_maxstep` 和 mixing 字段的审阅边界。
- 增加 SCF output review 证据表，覆盖程序结束、结构读取、赝势加载、cutoff、k-points/symmetry、occupations、SCF iteration、total energy、warning/restart。
- 明确 `JOB DONE` 只表示程序完成，不表示结果可直接进入科研结论。
- 增加 SCF 专属 `PASS / WARN / BLOCK` 和下游准入说明。

### `workflows/ground-state/nscf.md`

- 补强 NSCF 与 SCF、bands k-path 计算的区别。
- 明确 NSCF 依赖已审阅 SCF 的 `prefix/outdir` 数据链。
- 补强 dense uniform k mesh、`nbnd`、occupations、Fermi energy 和后处理目标的对应关系。
- 增加 output review 证据表，覆盖上游读取、k-points、band 数量、Fermi energy、warning/I-O。
- 增加 NSCF 的 `PASS / WARN / BLOCK`，说明 DOS、PDOS、Fermi surface 和 Wannier 类下游准入。

### `workflows/ground-state/cutoff-convergence.md`

- 强化收敛测试的目标：审阅目标 observable，而不是寻找通用 cutoff。
- 补强 `ecutwfc`、`ecutrho`、赝势类型、固定变量扫描和 scratch 隔离要求。
- 增加 output review 证据表，覆盖实际 cutoff、赝势类型、SCF 状态、目标 observable 和文件链。
- 增加 `PASS / WARN / BLOCK`，区分 total energy、force、stress、band gap 和 phonon frequency 等目标量的不同敏感性。

### `workflows/ground-state/kpoint-convergence.md`

- 强化 SCF uniform mesh、DOS dense mesh 和 bands k-path 的边界。
- 补强 shifted/unshifted mesh、irreducible k-points、symmetry 和 metallic/small-gap 敏感性说明。
- 增加 output review 证据表，覆盖 full mesh、irreducible k-points、shift flags、目标 observable 和 occupations coupling。
- 增加 `PASS / WARN / BLOCK`，避免用路径型 k-list 替代 k-point convergence。

### `workflows/ground-state/smearing-and-occupations.md`

- 强化 `occupations='fixed'`、`occupations='smearing'`、`smearing` 和 `degauss` 的审阅边界。
- 补强 metal、insulator、small-gap 体系中 occupations 对 total energy、force、DOS、phonon 和 EPC 的影响。
- 增加 output review 证据表，覆盖 occupation scheme、smearing/degauss、Fermi energy、SCF 行为和下游一致性。
- 增加 `PASS / WARN / BLOCK`，避免把 smearing artifact 当作物理峰或带隙判断。

### `workflows/ground-state/relax.md`

- 强化 relax 的目标：优化 internal coordinates，不直接产生最终 electronic structure 结论。
- 补强 `calculation='relax'`、`&IONS`、`ion_dynamics`、`forc_conv_thr`、每步电子 SCF、final forces 和 final coordinates 的关系。
- 增加 output review 证据表，覆盖 ionic step、每步电子 SCF、final forces、final coordinates、stress/pressure 和 final static SCF 计划。
- 增加 `PASS / WARN / BLOCK`，说明是否允许进入 final static SCF、bands/DOS 或 phonon。

### `workflows/ground-state/vc-relax.md`

- 强化 vc-relax 的目标：同时优化 cell 和 internal coordinates。
- 补强 `&IONS`、`&CELL`、`cell_dynamics`、`press`、`press_conv_thr`、cell constraints 和低维体系自由度边界。
- 增加 output review 证据表，覆盖 force/stress、pressure、final cell parameters、cell constraints、k-point density re-review 和 final static SCF 计划。
- 增加 `PASS / WARN / BLOCK`，避免把未审阅 vc-relax output 直接作为最终性质结果。

### `standards/output-review-checklist.md`

- 为 SCF、NSCF、relax 和 vc-relax 增加 `检查项 | 从哪里看 | 能证明什么 | 不能证明什么 | WARN/BLOCK 触发` 证据表。
- 新增 ground-state convergence scan 记录要求。
- 新增 final static SCF after relaxation 审阅条目。
- 新增 ground-state common BLOCK triggers，强调 `JOB DONE`、旧 scratch、未收敛 SCF、错误 k-list 和未审阅 final static SCF 的风险。

## Source boundary

本轮使用的来源类别：

- QE official documentation；
- QE `INPUT_PW` reference；
- QE user guide / official workflow documentation；
- `references/source-index.md` 和 `references/canonical-literature.md` 中已有 canonical literature spine；
- 本仓库 `standards/pass-warn-block.md`、`standards/calculation-record-template.md` 和 `standards/output-review-checklist.md` 的内部规范。

参数定义以 QE 官方 `INPUT_PW` 为准；canonical literature 用于支撑 DFT、SCF、数值收敛和可复查性边界，不替代官方 input reference。

## What was intentionally not changed

本轮没有处理：

- `workflows/electronic/` 中的 bands、DOS、PDOS、charge density、potential、work function；
- `workflows/phonon/` 中的 phonon、DFPT、q2r、matdyn、Born charge、IR/Raman；
- `workflows/advanced/` 中的 Wannier、EPC、GW/BSE、SOC、DFT+U、vdW 等高级页；
- `theory-minimum/` 正文大扩写；
- `physics-judgement/` 正文大扩写；
- 任何具体材料案例或固定参数建议。

## Follow-up

Phase 2-B 建议进入 `workflows/electronic/`：

- `workflows/electronic/bands.md`
- `workflows/electronic/dos.md`
- `workflows/electronic/pdos.md`
- `workflows/electronic/charge-density.md`
- `workflows/electronic/electrostatic-potential.md`
- `workflows/electronic/elf.md`
- `workflows/electronic/work-function.md`
- `workflows/electronic/fermi-surface.md`

Phase 2-B 的重点应是电子结构和后处理 workflow 的 output review、energy zero/Fermi level、uniform mesh 与 k-path 区分、DOS/PDOS 交叉检查、potential/work function 的边界判断。
