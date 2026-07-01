# NSCF workflow

## 本页解决什么问题

本页说明如何把 `pw.x nscf` 作为已审阅 SCF ground state 的后续数据生成步骤来使用。NSCF 在固定电荷密度上求解指定 k 点和 bands 的 eigenvalues/wavefunctions，用于 DOS、PDOS、Fermi surface、Wannier interface 等后处理；它不重新建立 ground state，也不能替代 SCF 收敛和参数收敛。

## 页面定位

- 对应学习路线：[learn/05-electronic-structure-loop.md](../../learn/05-electronic-structure-loop.md)
- 上游依赖：已通过 output review 的 SCF ground state
- 下游用途：DOS、PDOS、Fermi surface、Wannier90 interface、dense eigenvalue analysis
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

NSCF（non-self-consistent field）在固定 SCF 电荷密度上，对指定 k 点集合重新求解 Kohn-Sham eigenvalues。它不重新建立 ground-state density，主要用于生成更密 k mesh 或更多 bands，供 DOS、PDOS、Fermi surface 或 Wannier interface 使用。

NSCF 与 bands 的区别在于：NSCF 通常使用 dense uniform k mesh 做积分或投影；bands 使用 high-symmetry k-path 做色散图。

## 上游依赖

- 已完成 SCF，并确认 `prefix/outdir` 中的 ground-state 数据可读取。
- SCF 的赝势、cutoff、occupation/smearing 和结构已通过审阅。
- 已明确下游需要的是 dense k mesh、更多 `nbnd`，还是特定投影数据。
- NSCF input 中的结构、赝势、cutoff、spin/SOC、occupation family 应与上游 SCF 保持一致；除 `K_POINTS`、`nbnd` 和明确记录的后处理需求外，不应随意改变模型设置。
- `prefix/outdir` 必须指向同一个 SCF 数据边；不能把不同结构、不同赝势、不同泛函或不同磁性/SOC 设置的 scratch 混用。

## 计算图

```text
<structure> + <pseudo>
  -> pw.x scf on <scf_k_mesh>
  -> pw.x nscf on <dense_uniform_k_mesh>
  -> eigenvalues / wavefunctions for DOS, PDOS, Fermi surface, Wannier
```

## 需要的 QE 程序

- `pw.x`：先执行 `calculation='scf'`，再执行 `calculation='nscf'`。
- 下游可接 `dos.x`、`projwfc.x`、`fs.x`、`pw2wannier90.x`。

## 命令与文件边界

```bash
pw.x -in pw.nscf.<system>.in > pw.nscf.<system>.out
```

NSCF 输入文件仍需完整写出结构、赝势和数值设置，因为 `pw.x` 会核对这些对象并读取上游 `prefix/outdir`。人工文件名 `pw.nscf.<system>.out` 不会自动告诉下游程序读取哪个数据；下游程序依赖的是 `prefix/outdir` 中的 NSCF 波函数和 eigenvalue 数据。

NSCF 与 bands k-path 计算的边界必须写清：dense uniform mesh 用于积分型后处理，high-symmetry path 用于能带色散展示。将路径型 k-list 用于 DOS/PDOS/Fermi surface 是 `BLOCK` 级文件链错误。

## 通用输入模板

```fortran
&CONTROL
  calculation = 'nscf',
  prefix = '<system>',
  outdir = '<scratch_dir>',
  pseudo_dir = '<pseudo_dir>',
/
&SYSTEM
  ibrav = 0,
  nat = <number_of_atoms>,
  ntyp = <number_of_species>,
  ecutwfc = <wavefunction_cutoff>,
  ecutrho = <charge_density_cutoff>,
  nbnd = <number_of_bands>,
  occupations = '<occupation_scheme>',
  smearing = '<smearing_type>',
  degauss = <smearing_width>,
/
&ELECTRONS
  conv_thr = <nscf_threshold>,
/
ATOMIC_SPECIES
  <Element> <Mass> <Pseudo.UPF>

ATOMIC_POSITIONS <coordinate_type>
  <Element> <x> <y> <z>

K_POINTS automatic
  <dense_nk1> <dense_nk2> <dense_nk3> <sk1> <sk2> <sk3>

CELL_PARAMETERS <unit>
  <a1x> <a1y> <a1z>
  <a2x> <a2y> <a2z>
  <a3x> <a3y> <a3z>
```

## 关键 QE 输入对象

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `calculation='nscf'` | `pw.x` | 在固定 SCF 密度上求解指定 k 点本征值 | 把 NSCF 当成新的 ground state | output 显示 non self-consistent calculation |
| `prefix/outdir` | `pw.x` | 读取上游 SCF scratch | 与 SCF 不一致导致读旧数据或找不到密度 | output 中的读取目录和 prefix |
| `K_POINTS automatic` | `pw.x` | dense uniform k mesh | 用 bands path 生成 DOS/PDOS | output 的 k 点数量和 irreducible k-points |
| `nbnd` | `pw.x` | 控制输出 bands 数量 | 导带不足，DOS/PDOS/Wannier 截断 | output 中 number of Kohn-Sham states |
| `occupations/smearing/degauss` | `pw.x` | 影响金属或小带隙体系占据 | 与 SCF 或 DOS 目标不一致 | output 中 occupation scheme 与 Fermi energy |

## Output review

```markdown
## output review

- QE 程序:
- 计算类型:
- QE version:
- Input dependency:
- Structure summary:
- Pseudopotentials loaded:
- Cutoff reported:
- K-points reported:
- Convergence status:
- 本 workflow 关键输出:
- Warnings:
- Scratch / restart status:
- PASS / WARN / BLOCK:
- Reason:
- Allowed downstream workflows:
```

### 必查 output 证据

| 检查项 | 从哪里看 | 能证明什么 | 不能证明什么 | WARN/BLOCK 触发 |
|---|---|---|---|---|
| 上游读取 | output 开头的 `prefix/outdir`、data-file 读取信息 | NSCF 找到了某个上游数据边 | 不证明读到的是正确 SCF | prefix/outdir 错配、读取失败或旧 scratch 不明为 `BLOCK` |
| 结构/赝势一致性 | cell、atoms、pseudo loading | NSCF input 与上游记录可对照 | 不证明模型设置完全一致 | 结构、PP、spin/SOC 与 SCF 不一致为 `BLOCK` |
| k-points | number of k points、irreducible k-points | k 点集合被 QE 采用 | 不证明 mesh 足够密 | mesh 与目标后处理不匹配为 `WARN/BLOCK` |
| bands 数量 | number of Kohn-Sham states / bands | `nbnd` 生效 | 不证明能量窗口足够 | 目标未占据态或投影窗口缺失为 `WARN/BLOCK` |
| Fermi energy / occupation | Fermi energy、occupation scheme | 占据语境可追踪 | 不证明 DOS/Fermi surface 已收敛 | 与 SCF 差异无法解释为 `WARN` |
| warnings / I/O | warning、read/write errors | 文件链风险 | 不证明数据完整 | 波函数写入失败、并行 I/O 异常为 `BLOCK` |

## 输出判断标准

- 确认 NSCF 读取的是目标 SCF 的 `prefix/outdir`。
- k 点数量应符合下游 DOS/PDOS/Fermi surface 的分辨率需求。
- `nbnd` 应覆盖目标能量窗口和未占据态范围。
- Fermi energy、occupation 和 smearing 应与 SCF 记录一致或有明确变更理由。
- warning、对称性改写和旧 scratch 读取都应进入 `record.md`。

## 收敛性要求

- DOS/PDOS 通常要求比 SCF 更密的 uniform k mesh。
- Fermi surface 和 Wannier 前处理对 k mesh 和 `nbnd` 更敏感。
- 若 NSCF 改变 `nbnd`、k mesh 或 occupation 策略，应记录对下游量的影响。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| `PASS` | 上游 SCF 为 `PASS`；`prefix/outdir` 一致；k mesh 类型符合目标；`nbnd` 覆盖目标能量窗口；occupation/Fermi energy 可解释；无未解释 warning | 允许进入 DOS、PDOS、Fermi surface、Wannier 前处理或其他声明的 NSCF 下游 |
| `WARN` | NSCF 完成但 mesh、`nbnd` 或 energy window 只适合探索；Fermi level 与上游差异需标注；发生可追踪 restart/I/O warning | 只允许绘图预检、参数调整或探索性后处理 |
| `BLOCK` | 上游 SCF 为 `BLOCK`；prefix/outdir 错配；路径型 k-list 被用于积分型后处理；`nbnd` 不足；波函数/eigenvalue 文件缺失 | 不允许进入 DOS/PDOS/Fermi surface/Wannier 等下游 |

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| `dos.x` 或 `projwfc.x` 找不到数据 | NSCF 的 `prefix/outdir` 与下游不一致 | 检查所有输入中的 `prefix/outdir` |
| DOS 很锯齿或 Fermi 附近不稳定 | dense k mesh 不足或 smearing 不合适 | 增加 k mesh，复查 smearing |
| 导带范围不够 | `nbnd` 太小 | 增加 `nbnd` 并重跑 NSCF |
| 与 bands 图判断矛盾 | k mesh、能量零点或 smearing 处理不一致 | 对照 bands、DOS 的 Fermi reference |

## 通用学习模板

用 `<system>` 记录个人学习任务时，应保留 SCF 与 NSCF 两套 input/output，并在 record 中明确 NSCF 读取了哪个 SCF 状态。

## 记录模板

```text
pw.scf.<system>.in
pw.scf.<system>.out
pw.nscf.<system>.in
pw.nscf.<system>.out
record.md
```

## 下游影响

- DOS：`dos.x` 通常读取 NSCF 的 dense k mesh 数据。
- PDOS：`projwfc.x` 依赖 NSCF wavefunctions 和 projection 信息。
- Fermi surface：需要金属体系的 dense k mesh eigenvalues。
- Wannier90：常需要合适的 `nbnd`、k mesh 和投影窗口。

## 来源与边界

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE INPUT_DOS reference: <https://www.quantum-espresso.org/Doc/INPUT_DOS.html>
- QE INPUT_PROJWFC reference: <https://www.quantum-espresso.org/Doc/INPUT_PROJWFC.html>
- 本仓库 output review 标准：[standards/output-review-checklist.md](../../standards/output-review-checklist.md)

`calculation='nscf'`、`nbnd`、`K_POINTS` 等字段定义以 QE `INPUT_PW` 为准，属于 version-sensitive 内容；NSCF 不重建 ground state、必须依赖可信 SCF 的判断是 stable；mesh 密度和 `nbnd` 只作为后处理需求边界，需由目标 observable 决定。
