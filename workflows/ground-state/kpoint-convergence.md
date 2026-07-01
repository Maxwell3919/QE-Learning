# K-point convergence workflow

## 本页解决什么问题

本页说明如何为 ground-state workflow 建立 k-point sampling 策略。k-point convergence 的对象不是网格数字本身，而是 Brillouin-zone 积分后目标 observable 的稳定性。SCF uniform mesh、NSCF dense mesh 和 bands high-symmetry path 是不同对象，不能互相替代。

## 页面定位

- 对应学习路线：[learn/03-convergence-loop.md](../../learn/03-convergence-loop.md)
- 上游依赖：checked structure, pseudo, cutoff policy
- 下游用途：k-point policy for SCF, DOS, phonon, Fermi surface
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

评估 uniform k mesh、shift 和 symmetry 对目标量的影响，避免把 bands path 误当作 k 点收敛。

## 上游依赖

- `<structure>`、`<pseudo>` 和 `<system>` 已记录。
- SCF 基础参数与本 workflow 目标一致。
- 已明确输出是否允许进入下游 workflow。
- cutoff、赝势、occupation/smearing、spin/SOC 和结构在 k-point scan 中保持固定。
- k-point scan 需要记录 mesh、shift、symmetry 设置和 output 中的 irreducible k-points；如果 cell 后续变化，应重新评估 k-point density。

## 计算图

```text
<structure> + <pseudo> + cutoff policy
  -> pw.x scf with k-mesh scan
  -> convergence table
  -> k-point policy
```

## 需要的 QE 程序

`pw.x`

## 命令与文件边界

```bash
pw.x -in pw.scf.<system>.k<mesh_label>.in > pw.scf.<system>.k<mesh_label>.out
```

每个 mesh 应有可追踪的 input/output label，并在 convergence table 中记录 full mesh、shift flags、irreducible k-points、symmetry 设置、SCF 状态和目标 observable。不同 k mesh 的 scratch 不应无记录覆盖；如果复用 wavefunction 加速，必须确认结构、赝势、cutoff 和 occupation 不变。

## 通用输入模板

```fortran
&CONTROL
  calculation = 'scf',
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
  occupations = '<occupation_scheme>',
/
&ELECTRONS
  conv_thr = <scf_threshold>,
/
ATOMIC_SPECIES
  <Element> <Mass> <Pseudo.UPF>
ATOMIC_POSITIONS <coordinate_type>
  <Element> <x> <y> <z>
K_POINTS automatic
  <nk1> <nk2> <nk3> <sk1> <sk2> <sk3>
CELL_PARAMETERS <unit>
  <a1x> <a1y> <a1z>
  <a2x> <a2y> <a2z>
  <a3x> <a3y> <a3z>
```

## 关键 QE 输入对象

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `K_POINTS automatic` | pw.x | SCF/NSCF 的 uniform k mesh | 用 bands path 做收敛 | output 显示 k points 和 irreducible k-points |
| `shift flags` | pw.x | 记录 shifted/unshifted mesh | 不记录 shift 导致不可复查 | output k 点列表或 input record |
| `nosym/noinv` | pw.x | 影响对称性约化 | 随意开关改变不可约 k 点 | output 显示 symmetry operations |
| `occupations/smearing` | pw.x | 金属 k 点收敛相关 | 改变 k mesh 同时改 smearing | output occupation scheme |

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
| full k mesh | input 和 output k-point summary | mesh 设置可复查 | 不证明已收敛 | mesh label 与 output 不一致为 `BLOCK` |
| irreducible k-points | output 中不可约 k 点数和 symmetry 信息 | symmetry 约化被采用 | 不证明 symmetry 设置合理 | 不可约点异常变化需 `WARN` |
| shift flags | input record、k-point 列表 | shifted/unshifted mesh 可复现 | 不证明与体系最优匹配 | shift 未记录导致不可复查为 `WARN` |
| 目标 observable | total energy、force、stress、Fermi energy、DOS 或后续量 | kmesh 对目标的影响 | 不证明其他量同样稳定 | 只看总能但下游关心 DOS/phonon 为 `WARN` |
| occupation coupling | occupation scheme、Fermi energy、smearing | 金属积分语境可解释 | 不证明 Fermi surface 已解析 | 金属 mesh/smearing 未联动检查为 `WARN/BLOCK` |

## 输出判断标准

- 检查 irreducible k-points 是否随 symmetry 预期变化。
- 记录 total energy、force、stress、Fermi energy、DOS 近 Fermi 区域或 phonon frequency 的变化。
- bands k-path 只用于能带图，不用于 k mesh convergence。

## 收敛与可靠性

- 固定 cutoff、structure、pseudo 和 smearing，只扫描 k mesh。
- 金属、小带隙和低维体系通常对 k 点更敏感。
- DOS 和 Fermi surface 需要 dense NSCF k mesh，不能只依赖 SCF mesh。
- bands k-path 只检查沿高对称路径的色散，不是 Brillouin-zone 积分收敛测试。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| `PASS` | mesh、shift、symmetry 和 irreducible k-points 已记录；所有 scan 点 SCF 收敛；目标 observable 随 mesh 稳定；金属体系已联动审阅 smearing | 允许将该 k-point policy 用于声明的 SCF、relax 或下游前置计算 |
| `WARN` | 只检查 total energy；metal/small-gap/low-dimensional 体系的 mesh 余量不足；shift/symmetry 记录不完整但可追踪 | 只允许进入探索或继续收敛测试 |
| `BLOCK` | 使用 bands path 作为 kmesh 收敛；scan 点未收敛；mesh/output 不一致；与下游 DOS/phonon/Fermi surface 目标不匹配 | 不允许把该 k-point policy 传递到下游 |

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| DOS 锯齿或 Fermi 附近不稳 | k mesh 太稀 | 提高 NSCF dense mesh |
| bands 与 DOS 金属性判断冲突 | path 与 mesh 语义混用 | 分开检查 bands path 和 DOS mesh |
| 同一 mesh 点数结果不同 | shift/symmetry 未记录 | 记录 shift 和 symmetry 设置 |

## 通用学习模板

使用 `<system>`、`<structure>`、`<pseudo>`、`<k_mesh>`、`<ecutwfc>`、`<ecutrho>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
pw.scf.<system>.k<nk>.in
pw.scf.<system>.k<nk>.out
kpoint-convergence.<system>.md
record.md
```

## 下游影响

- SCF 使用基本 k mesh。
- DOS/PDOS 通常需要更密 NSCF mesh。
- bands 使用 k-path，应单独记录 k-path 来源。

## 来源与边界

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- SeeK-path documentation: <https://seekpath.readthedocs.io/>
- 本仓库 output review 标准：[standards/output-review-checklist.md](../../standards/output-review-checklist.md)
- Canonical literature: [references/canonical-literature.md](../../references/canonical-literature.md)

`K_POINTS` 语法和 symmetry 相关字段以 QE `INPUT_PW` 为准；Monkhorst-Pack sampling 和 Brillouin-zone integration 的方法边界是 stable；具体 mesh density 取决于体系维度、金属性和目标 observable。
