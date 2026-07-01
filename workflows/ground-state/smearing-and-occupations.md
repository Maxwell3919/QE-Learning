# Smearing and occupations workflow

## 本页解决什么问题

本页说明如何审阅 `occupations`、`smearing` 和 `degauss`，避免把占据策略当成无害的数值细节。occupation policy 会影响 SCF 收敛、total energy、forces、DOS、phonon 和后续 EPC/Fermi-surface 判断，尤其在金属、小带隙和接近费米面的体系中。

## 页面定位

- 对应学习路线：[learn/03-convergence-loop.md](../../learn/03-convergence-loop.md)
- 上游依赖：physical metal/insulator judgment and SCF input
- 下游用途：occupation policy for SCF, relax, DOS, phonon
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

建立 `occupations`、`smearing`、`degauss` 的可解释策略，区分金属、绝缘体和小带隙体系的数值处理。

## 上游依赖

- `<structure>`、`<pseudo>` 和 `<system>` 已记录。
- SCF 基础参数与本 workflow 目标一致。
- 已明确输出是否允许进入下游 workflow。
- 已有关于体系是金属、绝缘体还是小带隙体系的初步物理判断；若判断来自粗糙 SCF，应标注为 provisional。
- k-point mesh、cutoff、spin/SOC 和结构固定后，再比较 occupation/smearing；不要同时改变多个变量。

## 计算图

```text
<structure> + <pseudo> + k mesh
  -> pw.x scf with occupation/smearing choices
  -> occupation and Fermi-energy review
  -> smearing policy
```

## 需要的 QE 程序

`pw.x`

## 命令与文件边界

```bash
pw.x -in pw.scf.<system>.occ-<policy>.in > pw.scf.<system>.occ-<policy>.out
```

occupation policy review 应与 k-point convergence 共同记录。对于金属或小带隙体系，同一个 `degauss` 在 SCF、relax、NSCF、DOS 和 phonon 中可能产生不同影响；记录中必须写明该设置服务哪个 workflow，不应把用于 SCF 稳定的 smearing 自动当作最终 DOS 展宽。

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
  smearing = '<smearing_type>',
  degauss = <smearing_width>,
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
| `occupations` | pw.x | 选择 fixed、smearing、tetrahedra 等占据策略 | 金属使用 fixed 或绝缘体无理由 smearing | output 显示 occupation scheme |
| `smearing` | pw.x | 展宽函数 | 不同 workflow 随意切换 | output 显示 smearing type |
| `degauss` | pw.x | 展宽宽度，单位 Ry | 过大改变物理解释 | output 显示 smearing width |
| `Fermi energy` | output | 占据和金属性审阅指标 | 能量零点不记录 | output 中 Fermi energy |

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
| occupation scheme | output 的 occupation 说明 | QE 采用了哪种占据策略 | 不证明该策略物理合适 | 与体系判断不一致为 `WARN/BLOCK` |
| smearing type / degauss | output 和 input record | 展宽方法和宽度可复查 | 不证明 DOS 峰或 gap 可信 | 未记录 degauss 或单位混淆为 `WARN` |
| Fermi energy | output 的 Fermi energy | 占据和费米面语境可追踪 | 不证明 Fermi surface 已收敛 | Fermi level 随 mesh/smearing 大幅漂移为 `WARN` |
| SCF behavior | iteration、estimated accuracy | occupation 对收敛是否稳定 | 不证明其他 observable 稳定 | fixed occupation 导致金属 SCF 失败为 `BLOCK` |
| 下游一致性 | SCF/NSCF/DOS/phonon records | 不同步骤设置是否一致或有解释 | 不证明设置可跨 workflow 复用 | 无理由切换 policy 为 `WARN/BLOCK` |

## 输出判断标准

- 检查 occupation scheme 是否符合体系物理判断。
- 检查 Fermi energy、number of electrons 和 SCF iteration 是否稳定。
- 记录 smearing 对 total energy、force、DOS、phonon 的影响。

## 收敛与可靠性

- 金属体系需要同时检查 k mesh 和 smearing。
- DOS 图的平滑不等于物理可信，需保留 broadening 设置。
- phonon 中 smearing 可能影响频率，应记录并复查。
- 绝缘体若使用 fixed occupations 并稳定收敛，应避免无理由加入 smearing；金属若使用 fixed occupations 导致不稳定或不可解释占据，应重新审阅。
- degauss 过大会改变费米附近 occupation、总能和力的解释；过小可能导致金属 SCF 难以收敛。可接受范围必须由目标 observable 稳定性决定。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| `PASS` | occupation scheme 与物理判断一致；smearing/degauss 已记录；SCF 稳定；kmesh/smearing 联动对目标 observable 的影响已审阅 | 允许进入声明的 SCF、relax、NSCF、DOS 或 phonon 前置步骤 |
| `WARN` | 设置可运行但只适合探索；Fermi energy 对 kmesh/smearing 敏感；SCF 与 NSCF/DOS smearing 不一致但有记录 | 允许继续测试或探索，不应用于最终 DOS/phonon/EPC 解释 |
| `BLOCK` | 金属使用不合适 fixed occupation 导致收敛或占据异常；绝缘体/小 gap 体系用过大 smearing 改写物理解释；degauss 未记录；下游设置混乱 | 不允许进入依赖占据、Fermi level 或费米面细节的下游 |

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

## 下游影响

- SCF 收敛、relax、DOS、phonon 都可能受 occupation 策略影响。
- DOS/PDOS 能量展宽应与 SCF smearing 分开记录。

## 来源与边界

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- 本仓库 output review 标准：[standards/output-review-checklist.md](../../standards/output-review-checklist.md)
- Canonical literature: [references/canonical-literature.md](../../references/canonical-literature.md)

`occupations/smearing/degauss` 字段定义以 QE `INPUT_PW` 为准；smearing 作为 Brillouin-zone integration 和 SCF 稳定策略的角色是 stable；具体 smearing 宽度不是通用参数，必须由 k-point convergence 和目标 observable 审阅决定。
