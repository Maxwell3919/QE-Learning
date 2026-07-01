# Relax workflow

## 本页解决什么问题

本页说明如何审阅 `pw.x relax`：在固定晶胞下优化 internal coordinates，并判断最终原子坐标是否可以进入 final static SCF 和后续性质计算。relax 的结果是结构状态，不是最终电子结构结论；它必须通过 final forces、ionic convergence、最后一次电子 SCF 和 output warning 审阅。

## 页面定位

- 对应学习路线：[learn/04-relaxation-loop.md](../../learn/04-relaxation-loop.md)
- 上游依赖：SCF parameter policy and checked structure
- 下游用途：optimized atomic coordinates for static SCF, bands, DOS, phonon
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

在固定晶胞下优化原子位置，通过力收敛和最终坐标审阅判断结构能否进入下游。

## 上游依赖

- `<structure>`、`<pseudo>` 和 `<system>` 已记录。
- SCF 基础参数与本 workflow 目标一致。
- 已明确输出是否允许进入下游 workflow。
- cutoff、k mesh、occupation/smearing 应足以支撑 force 判断；只对 total energy 收敛的参数不一定适合 relax。
- `prefix/outdir` 可用于同一 relax 的 restart，但不能混入不同结构、不同优化策略或不同晶胞约束的旧 scratch。
- 固定晶胞 relax 只优化原子坐标，不优化晶胞；若下游关心应力或晶胞，应改用或补做 `vc-relax`。

## 计算图

```text
<initial_structure> + <pseudo> + SCF policy
  -> pw.x relax
  -> final atomic positions + force review
  -> static SCF on relaxed structure
```

## 需要的 QE 程序

`pw.x`

## 命令与文件边界

```bash
pw.x -in pw.relax.<system>.in > pw.relax.<system>.out
```

`pw.relax.<system>.out` 中的 final coordinates 是下一步结构输入的来源；不要只复制原始 input 中的 `ATOMIC_POSITIONS`。relax 生成的 scratch 记录了优化过程中的电子状态，但下游 bands/DOS/phonon 应以优化后的结构重新运行 final static SCF，形成清晰的数据边。

若 relax 中发生 restart、step rejection、trust radius 调整或优化器切换，应写入 record。它们不必自动导致 `BLOCK`，但没有 provenance 的结构不能作为正式下游输入。

## 通用输入模板

```fortran
&CONTROL
  calculation = 'relax',
  prefix = '<system>',
  outdir = '<scratch_dir>',
  pseudo_dir = '<pseudo_dir>',
  forc_conv_thr = <force_threshold>,
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
&IONS
  ion_dynamics = '<ion_optimizer>',
/
```

## 关键 QE 输入对象

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `calculation` | pw.x | 设置为 'relax' | 用 SCF 代替结构优化 | output 显示 ionic minimization |
| `ion_dynamics` | pw.x / &IONS | 离子优化算法 | 失败后不分析优化历史 | output 显示 optimizer 状态 |
| `forc_conv_thr` | pw.x | 力收敛阈值 | 未按下游目标设置 | output 中 final forces |
| `ATOMIC_POSITIONS` | pw.x | 初始与最终坐标 | 只保存 input 坐标 | output 末尾 updated coordinates |

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
| 程序结束 | output 末尾、错误栈 | relax 运行到结束 | 不证明离子优化收敛 | 异常中止或 output 截断为 `BLOCK` |
| 每步电子 SCF | ionic step 内 SCF convergence / estimated accuracy | 每个离子步的力来自可接受电子态 | 不证明最终结构可用 | 关键步 SCF 未收敛为 `WARN/BLOCK` |
| final forces | final force table、total force、最大力分量 | internal coordinates 是否达到力准则 | 不证明 stress/cell 合理 | final force 未达目标为 `BLOCK` 或继续 relax |
| convergence message | bfgs/ionic minimization 停止原因 | 程序为何停止 | 不证明物理稳定 | 停止原因不是力收敛为 `WARN/BLOCK` |
| final coordinates | output 末尾 `ATOMIC_POSITIONS` | 下游结构来源 | 不证明已做 static SCF | 坐标来源不清或单位混乱为 `BLOCK` |
| stress / pressure | stress tensor 和 pressure 段 | 固定晶胞下的残余应力可记录 | 不证明晶胞已优化 | 下游关心应力但未处理为 `WARN` |
| warnings / restart | warning、restart、optimizer history | 优化风险可追踪 | 不证明风险已消除 | restart/step history 无记录为 `WARN` |

## 输出判断标准

- 检查最后一次电子 SCF 是否收敛。
- 检查 final forces、convergence message 和 updated coordinates。
- relax 完成后通常需要用最终结构重跑 static SCF，作为 bands/DOS/phonon 的上游。
- 物理边界见 [physics-judgement/relax-is-not-final-electronic-structure.md](../../physics-judgement/relax-is-not-final-electronic-structure.md)：relax 输出结构不是最终电子结构结论。

## 收敛与可靠性

- cutoff/k mesh 应对 force 收敛足够。
- 力阈值应与下游目标匹配；phonon 前需要更严格结构审阅。
- 固定晶胞 relax 不审阅 stress 作为晶胞优化准入。
- 程序完成、电子 SCF 收敛和离子力收敛是不同层级；三者都不能替代 cutoff/kmesh/occupation 对 forces 的收敛测试。
- relax 完成后进入 electronic/phonon 下游前，应以 final coordinates 重新运行 `calculation='scf'`，并审阅该 final static SCF 的 output。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| `PASS` | relax 正常完成；关键 ionic step 的 SCF 可接受；final forces 达到目标；final coordinates 可追踪；warning 已解释；已计划或完成 final static SCF | 允许进入 final static SCF；final SCF 通过后可进入 bands/DOS/phonon 等下游 |
| `WARN` | 力接近阈值但只适合继续优化或探索；restart/optimizer history 可追踪；stress 较大但固定晶胞边界已说明 | 允许继续 relax、调整参数或做低风险结构筛选；不得直接进入最终 phonon |
| `BLOCK` | 离子优化未收敛；final coordinates 不可追踪；关键 SCF 失败；约束/单位/原子顺序错误；直接把 relax output 当最终电子结构 | 不允许进入 final SCF 之外的性质下游，先修复结构优化 |

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| JOB DONE 但结构不可用 | 离子优化未收敛或最后 SCF 异常 | 检查 final force 和 convergence message |
| 坐标未更新到记录 | 只保存 input 文件 | 从 output 提取 final coordinates |
| phonon 出现虚频 | 结构残余力过大或参数不够严格 | 收紧 relax 与 SCF 参数 |

## 通用学习模板

使用 `<system>`、`<structure>`、`<pseudo>`、`<k_mesh>`、`<ecutwfc>`、`<ecutrho>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
pw.relax.<system>.in
pw.relax.<system>.out
pw.scf.<system>.relaxed.in
pw.scf.<system>.relaxed.out
record.md
```

## 下游影响

- relax 输出结构应进入 static SCF。
- bands/DOS 使用 static SCF/NSCF，不直接把 relax 输出当最终电子结构。
- phonon 依赖更严格的结构和 SCF 审阅。

## 来源与边界

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- 本仓库 output review 标准：[standards/output-review-checklist.md](../../standards/output-review-checklist.md)
- 物理判断边界：[physics-judgement/relax-is-not-final-electronic-structure.md](../../physics-judgement/relax-is-not-final-electronic-structure.md)

`calculation='relax'`、`forc_conv_thr`、`ion_dynamics` 等字段定义以 QE `INPUT_PW` 为准；relax 输出结构需要 final static SCF 承接下游是 stable workflow 边界；具体 force threshold 由目标 observable 决定，不能写成通用值。
