# Relax workflow

## 页面定位

- 对应学习路线：[learn/04-relaxation-loop.md](../../learn/04-relaxation-loop.md)
- 上游依赖：SCF parameter policy and checked structure
- 下游用途：optimized atomic coordinates for static SCF, bands, DOS, phonon
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

在固定晶胞下优化原子位置，通过力收敛和最终坐标审阅判断结构能否进入下游。

## 输入前提

- `<structure>`、`<pseudo>` 和 `<system>` 已记录。
- SCF 基础参数与本 workflow 目标一致。
- 已明确输出是否允许进入下游 workflow。

## 计算图

```text
<initial_structure> + <pseudo> + SCF policy
  -> pw.x relax
  -> final atomic positions + force review
  -> static SCF on relaxed structure
```

## 需要的 QE 程序

`pw.x`

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

## 输入字段说明

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `calculation` | pw.x | 设置为 'relax' | 用 SCF 代替结构优化 | output 显示 ionic minimization |
| `ion_dynamics` | pw.x / &IONS | 离子优化算法 | 失败后不分析优化历史 | output 显示 optimizer 状态 |
| `forc_conv_thr` | pw.x | 力收敛阈值 | 未按下游目标设置 | output 中 final forces |
| `ATOMIC_POSITIONS` | pw.x | 初始与最终坐标 | 只保存 input 坐标 | output 末尾 updated coordinates |

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

- 检查最后一次电子 SCF 是否收敛。
- 检查 final forces、convergence message 和 updated coordinates。
- relax 完成后通常需要用最终结构重跑 static SCF，作为 bands/DOS/phonon 的上游。

## 收敛性要求

- cutoff/k mesh 应对 force 收敛足够。
- 力阈值应与下游目标匹配；phonon 前需要更严格结构审阅。
- 固定晶胞 relax 不审阅 stress 作为晶胞优化准入。

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

## 与其他 workflow 的关系

- relax 输出结构应进入 static SCF。
- bands/DOS 使用 static SCF/NSCF，不直接把 relax 输出当最终电子结构。
- phonon 依赖更严格的结构和 SCF 审阅。

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
