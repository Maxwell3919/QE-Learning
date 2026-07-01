# NSCF workflow

## 页面定位

- 对应学习路线：[learn/05-electronic-structure-loop.md](../../learn/05-electronic-structure-loop.md)
- 上游依赖：已通过 output review 的 SCF ground state
- 下游用途：DOS、PDOS、Fermi surface、Wannier90 interface、dense eigenvalue analysis
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

NSCF（non-self-consistent field）在固定 SCF 电荷密度上，对指定 k 点集合重新求解 Kohn-Sham eigenvalues。它不重新建立 ground-state density，主要用于生成更密 k mesh 或更多 bands，供 DOS、PDOS、Fermi surface 或 Wannier interface 使用。

NSCF 与 bands 的区别在于：NSCF 通常使用 dense uniform k mesh 做积分或投影；bands 使用 high-symmetry k-path 做色散图。

## 输入前提

- 已完成 SCF，并确认 `prefix/outdir` 中的 ground-state 数据可读取。
- SCF 的赝势、cutoff、occupation/smearing 和结构已通过审阅。
- 已明确下游需要的是 dense k mesh、更多 `nbnd`，还是特定投影数据。

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

## 输入字段说明

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `calculation='nscf'` | `pw.x` | 在固定 SCF 密度上求解指定 k 点本征值 | 把 NSCF 当成新的 ground state | output 显示 non self-consistent calculation |
| `prefix/outdir` | `pw.x` | 读取上游 SCF scratch | 与 SCF 不一致导致读旧数据或找不到密度 | output 中的读取目录和 prefix |
| `K_POINTS automatic` | `pw.x` | dense uniform k mesh | 用 bands path 生成 DOS/PDOS | output 的 k 点数量和 irreducible k-points |
| `nbnd` | `pw.x` | 控制输出 bands 数量 | 导带不足，DOS/PDOS/Wannier 截断 | output 中 number of Kohn-Sham states |
| `occupations/smearing/degauss` | `pw.x` | 影响金属或小带隙体系占据 | 与 SCF 或 DOS 目标不一致 | output 中 occupation scheme 与 Fermi energy |

## 通用输出审阅模板

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

## 与其他 workflow 的关系

- DOS：`dos.x` 通常读取 NSCF 的 dense k mesh 数据。
- PDOS：`projwfc.x` 依赖 NSCF wavefunctions 和 projection 信息。
- Fermi surface：需要金属体系的 dense k mesh eigenvalues。
- Wannier90：常需要合适的 `nbnd`、k mesh 和投影窗口。

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE INPUT_DOS reference: <https://www.quantum-espresso.org/Doc/INPUT_DOS.html>
- QE INPUT_PROJWFC reference: <https://www.quantum-espresso.org/Doc/INPUT_PROJWFC.html>
