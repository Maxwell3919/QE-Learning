# VC-relax workflow

## 本页解决什么问题

本页说明如何审阅 `pw.x vc-relax`：同时优化 cell 和 internal coordinates，并判断 final cell + final positions 是否可以进入 final static SCF 和响应性质下游。vc-relax 的输出是平衡结构候选，不是最终 electronic structure；晶胞变化还会传播到 k-point density、stress、symmetry 和后续 phonon 判断。

## 页面定位

- 对应学习路线：[learn/04-relaxation-loop.md](../../learn/04-relaxation-loop.md)
- 上游依赖：SCF parameter policy and cell relaxation decision
- 下游用途：optimized cell and coordinates for static SCF and response workflows
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

同时优化晶胞和原子位置，通过 force、stress、pressure 和 final cell 审阅判断结构是否可进入下游。

## 上游依赖

- `<structure>`、`<pseudo>` 和 `<system>` 已记录。
- SCF 基础参数与本 workflow 目标一致。
- 已明确输出是否允许进入下游 workflow。
- cutoff/k mesh/occupation 必须能支撑 force 和 stress 判断；只对 total energy 收敛的设置不足以支撑 vc-relax。
- 已明确晶胞自由度是否应该放开，以及低维、表面、真空或外压边界是否允许 cell 改变。
- `prefix/outdir` 只能复用同一结构优化分支的 scratch；不同 cell constraints、外压或结构来源不能混用。

## 计算图

```text
<initial_cell_and_structure> + <pseudo> + SCF policy
  -> pw.x vc-relax
  -> final CELL_PARAMETERS + ATOMIC_POSITIONS
  -> static SCF on final cell
```

## 需要的 QE 程序

`pw.x`

## 命令与文件边界

```bash
pw.x -in pw.vc-relax.<system>.in > pw.vc-relax.<system>.out
```

vc-relax 的下游结构应来自 output 末尾的 final `CELL_PARAMETERS` 和 `ATOMIC_POSITIONS`。晶胞改变后，原来的 k mesh 数字不一定对应同样的 reciprocal-space density；进入 final static SCF 前，应重新审阅 k-point density、cutoff policy 和 symmetry。

如果需要 restart，应记录 restart 来源、cell step 历史和是否改变过 `cell_dofree/press/press_conv_thr`。混用不同 cell 自由度或外压语境的 scratch 会破坏结构 provenance。

## 通用输入模板

```fortran
&CONTROL
  calculation = 'vc-relax',
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
&IONS
  ion_dynamics = '<ion_optimizer>',
/
&CELL
  cell_dynamics = '<cell_optimizer>',
  press = <target_pressure>,
  press_conv_thr = <pressure_threshold>,
  cell_dofree = '<cell_degrees_of_freedom>',
/
```

## 关键 QE 输入对象

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `calculation` | pw.x | 设置为 'vc-relax' | 不需要变晶胞却放开晶胞 | output 显示 variable-cell optimization |
| `cell_dynamics` | pw.x / &CELL | 晶胞优化算法 | 不记录 cell 优化方式 | output 显示 cell update |
| `press/press_conv_thr` | pw.x / &CELL | 目标压力和压力收敛 | stress 未收敛进入 phonon | output 中 pressure/stress tensor |
| `cell_dofree` | pw.x / &CELL | 限制晶胞自由度 | 低维体系放开真空方向 | output 中 final cell 变化 |
| `K_POINTS` | pw.x | cell 改变后 k 点密度可能变化 | 沿用旧 mesh 不复查密度 | final cell 与 k spacing 对照 |

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
| 程序结束 | output 末尾、错误栈 | vc-relax 运行到结束 | 不证明 force/stress 收敛 | 异常中止或 output 截断为 `BLOCK` |
| 每步电子 SCF | ionic/cell step 内 SCF convergence | force/stress 来自可接受电子态 | 不证明最终 cell 合理 | 关键步 SCF 失败为 `WARN/BLOCK` |
| final forces | final force table、total force | internal coordinates 是否达标 | 不证明 cell 达标 | force 未达目标为 `BLOCK` 或继续优化 |
| stress / pressure | stress tensor、pressure、`total stress` | cell 优化是否达目标语境 | 不证明所有应力相关性质收敛 | stress/pressure 未收敛为 `BLOCK` |
| final cell | final `CELL_PARAMETERS`、volume、angles | 下游晶胞来源 | 不证明 kmesh 仍合适 | 体积塌缩、真空方向异常、单位混乱为 `BLOCK` |
| cell constraints | `cell_dofree`、cell update history | 晶胞自由度可追踪 | 不证明约束物理正确 | 低维/真空方向无理由放开为 `WARN/BLOCK` |
| final static SCF plan | record / downstream input | 下游数据边清楚 | 不证明已完成性质计算 | 直接用 vc-relax output 做 bands/DOS/phonon 为 `WARN/BLOCK` |

## 输出判断标准

- 检查 final forces、stress tensor、pressure 和 final cell parameters。
- 确认 cell 自由度设置符合维度与物理问题。
- vc-relax 后应以 final cell/coordinates 重跑 static SCF。
- 物理边界见 [physics-judgement/relax-is-not-final-electronic-structure.md](../../physics-judgement/relax-is-not-final-electronic-structure.md)：cell 改变会传播到 k 点密度、stress、symmetry 和下游性质判断。

## 收敛与可靠性

- cutoff/k mesh 应对 stress 和 force 同时足够。
- cell 改变后需要复查 k 点密度和 cutoff policy。
- 低维体系应约束非物理晶胞方向，不在本页展开结构构建细节。
- 程序完成、电子 SCF 收敛、力收敛和压力/应力收敛是不同层级；任何一个层级缺失都不能直接进入 phonon 或应力敏感下游。
- vc-relax 后进入 bands/DOS/phonon 前，应以 final cell + final coordinates 重新运行 static SCF，并重新审阅 `K_POINTS` 与 final cell 的关系。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| `PASS` | vc-relax 正常完成；关键 SCF 可接受；final force、stress/pressure 达到目标；cell constraints 合理；final cell/positions 可追踪；已计划或完成 final static SCF | 允许进入 final static SCF；final SCF 通过后可进入 bands/DOS/phonon/应力相关下游 |
| `WARN` | force 或 stress 接近阈值但只适合继续优化；cell 改变较大需重审 kmesh/cutoff；低维约束边界已说明但仍需复核 | 允许继续 vc-relax、参数排查或结构趋势探索；不得作为最终相稳定性或 phonon 依据 |
| `BLOCK` | force/stress 未收敛；final cell 非物理或不可追踪；cell freedom/外压设置错误；关键 SCF 失败；直接把 vc-relax output 当最终电子结构 | 不允许进入依赖平衡晶胞的下游 |

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| 真空方向被压缩或拉伸 | 低维体系 cell_dofree 设置不当 | 限制 cell 自由度并重跑 |
| pressure 未收敛 | press_conv_thr 或 stress 参数不足 | 检查 stress、cutoff、k mesh |
| 下游结果不可比 | cell 改变后沿用旧 k mesh | 按 final cell 复查 k 点密度 |

## 通用学习模板

使用 `<system>`、`<structure>`、`<pseudo>`、`<k_mesh>`、`<ecutwfc>`、`<ecutrho>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
pw.vc-relax.<system>.in
pw.vc-relax.<system>.out
pw.scf.<system>.final-cell.in
pw.scf.<system>.final-cell.out
record.md
```

## 下游影响

- vc-relax 输出 final cell 和 coordinates。
- final static SCF 是 bands/DOS/phonon 的清晰数据边。
- phonon 需要特别审阅残余 force/stress。

## 来源与边界

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- 本仓库 output review 标准：[standards/output-review-checklist.md](../../standards/output-review-checklist.md)
- 物理判断边界：[physics-judgement/relax-is-not-final-electronic-structure.md](../../physics-judgement/relax-is-not-final-electronic-structure.md)

`calculation='vc-relax'`、`&CELL`、`press`、`press_conv_thr`、`cell_dofree` 等字段定义以 QE `INPUT_PW` 为准，属于 version-sensitive 内容；final cell 需要 final static SCF 承接下游是 stable workflow 边界；具体压力、应力和力阈值由研究目标决定，不能写成通用值。
