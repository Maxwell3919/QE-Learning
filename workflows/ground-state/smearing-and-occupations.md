# Smearing and occupations workflow

## 页面定位

- 对应学习路线：[learn/03-convergence-loop.md](../../learn/03-convergence-loop.md)
- 上游依赖：physical metal/insulator judgment and SCF input
- 下游用途：occupation policy for SCF, relax, DOS, phonon
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

建立 `occupations`、`smearing`、`degauss` 的可解释策略，区分金属、绝缘体和小带隙体系的数值处理。

## 输入前提

- `<structure>`、`<pseudo>` 和 `<system>` 已记录。
- SCF 基础参数与本 workflow 目标一致。
- 已明确输出是否允许进入下游 workflow。

## 计算图

```text
<structure> + <pseudo> + k mesh
  -> pw.x scf with occupation/smearing choices
  -> occupation and Fermi-energy review
  -> smearing policy
```

## 需要的 QE 程序

`pw.x`

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

## 输入字段说明

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `occupations` | pw.x | 选择 fixed、smearing、tetrahedra 等占据策略 | 金属使用 fixed 或绝缘体无理由 smearing | output 显示 occupation scheme |
| `smearing` | pw.x | 展宽函数 | 不同 workflow 随意切换 | output 显示 smearing type |
| `degauss` | pw.x | 展宽宽度，单位 Ry | 过大改变物理解释 | output 显示 smearing width |
| `Fermi energy` | output | 占据和金属性审阅指标 | 能量零点不记录 | output 中 Fermi energy |

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

- 检查 occupation scheme 是否符合体系物理判断。
- 检查 Fermi energy、number of electrons 和 SCF iteration 是否稳定。
- 记录 smearing 对 total energy、force、DOS、phonon 的影响。

## 收敛性要求

- 金属体系需要同时检查 k mesh 和 smearing。
- DOS 图的平滑不等于物理可信，需保留 broadening 设置。
- phonon 中 smearing 可能影响频率，应记录并复查。

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| SCF 振荡 | 金属 fixed occupation 或 smearing 不合适 | 改用合适 occupation 并复查 k mesh |
| DOS 被过度抹平 | degauss 或后处理 broadening 太大 | 减小展宽并对比 bands |
| 不同步骤不可比 | SCF/NSCF/DOS smearing 不一致 | 统一或记录变更理由 |

## 通用学习模板

使用 `<system>`、`<structure>`、`<pseudo>`、`<k_mesh>`、`<ecutwfc>`、`<ecutrho>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
pw.scf.<system>.occ-<policy>.in
pw.scf.<system>.occ-<policy>.out
smearing-review.<system>.md
record.md
```

## 与其他 workflow 的关系

- SCF 收敛、relax、DOS、phonon 都可能受 occupation 策略影响。
- DOS/PDOS 能量展宽应与 SCF smearing 分开记录。

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
