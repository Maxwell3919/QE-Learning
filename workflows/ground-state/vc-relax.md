# VC-relax workflow

## 页面定位

- 对应学习路线：[learn/04-relaxation-loop.md](../../learn/04-relaxation-loop.md)
- 上游依赖：SCF parameter policy and cell relaxation decision
- 下游用途：optimized cell and coordinates for static SCF and response workflows
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

同时优化晶胞和原子位置，通过 force、stress、pressure 和 final cell 审阅判断结构是否可进入下游。

## 输入前提

- `<structure>`、`<pseudo>` 和 `<system>` 已记录。
- SCF 基础参数与本 workflow 目标一致。
- 已明确输出是否允许进入下游 workflow。

## 计算图

```text
<initial_cell_and_structure> + <pseudo> + SCF policy
  -> pw.x vc-relax
  -> final CELL_PARAMETERS + ATOMIC_POSITIONS
  -> static SCF on final cell
```

## 需要的 QE 程序

`pw.x`

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

## 输入字段说明

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `calculation` | pw.x | 设置为 'vc-relax' | 不需要变晶胞却放开晶胞 | output 显示 variable-cell optimization |
| `cell_dynamics` | pw.x / &CELL | 晶胞优化算法 | 不记录 cell 优化方式 | output 显示 cell update |
| `press/press_conv_thr` | pw.x / &CELL | 目标压力和压力收敛 | stress 未收敛进入 phonon | output 中 pressure/stress tensor |
| `cell_dofree` | pw.x / &CELL | 限制晶胞自由度 | 低维体系放开真空方向 | output 中 final cell 变化 |
| `K_POINTS` | pw.x | cell 改变后 k 点密度可能变化 | 沿用旧 mesh 不复查密度 | final cell 与 k spacing 对照 |

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

- 检查 final forces、stress tensor、pressure 和 final cell parameters。
- 确认 cell 自由度设置符合维度与物理问题。
- vc-relax 后应以 final cell/coordinates 重跑 static SCF。
- 物理边界见 [physics-judgement/relax-is-not-final-electronic-structure.md](../../physics-judgement/relax-is-not-final-electronic-structure.md)：cell 改变会传播到 k 点密度、stress、symmetry 和下游性质判断。

## 收敛性要求

- cutoff/k mesh 应对 stress 和 force 同时足够。
- cell 改变后需要复查 k 点密度和 cutoff policy。
- 低维体系应约束非物理晶胞方向，不在本页展开结构构建细节。

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

## 与其他 workflow 的关系

- vc-relax 输出 final cell 和 coordinates。
- final static SCF 是 bands/DOS/phonon 的清晰数据边。
- phonon 需要特别审阅残余 force/stress。

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
