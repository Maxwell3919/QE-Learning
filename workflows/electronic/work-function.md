# Work-function workflow

## 页面定位

- 对应学习路线：[learn/07-postprocessing-loop.md](../../learn/07-postprocessing-loop.md)
- 上游依赖：slab SCF and electrostatic potential profile
- 下游用途：surface work-function estimate
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

从 slab 的方向平均电势中确定 vacuum level，并与 Fermi level 组合计算 work function。结构构建细节不展开，本页只说明 QE 输入和 output review 要求。

## 输入前提

- 上游 input、output、`record.md` 已可追溯。
- `<system>`、`<structure>`、`<pseudo>`、cutoff、k 点和 occupation 设置已记录。
- 已明确本 workflow 输出如何进入下游或图像解释。

## 计算图

```text
slab-like <structure>
  -> pw.x scf
  -> pp.x potential
  -> average.x profile
  -> vacuum level - Fermi energy
```

## 需要的 QE 程序

`pw.x`、`pp.x`、`average.x`

## 通用输入模板

```fortran
# SCF requires a slab-like structure already prepared.
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

# Then extract potential with pp.x and average.x.
&INPUTPP
  prefix = '<system>',
  outdir = '<scratch_dir>',
  plot_num = <potential_plot_num>,
  filplot = 'potential.<system>.dat',
/
```

## 输入字段说明

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `Fermi energy` | pw.x output | work function 的电子能级参考 | 没有记录能量零点 | SCF output 中 Fermi energy |
| `plot_num` | pp.x | 提取势相关量 | 势类型选择错误 | pp.x output |
| `idir` | average.x | 沿 slab 法向平均 | 方向错误 | average profile |
| `assume_isolated/dipole options` | pw.x | 低维边界和偶极修正相关设置 | 未记录 slab 边界处理 | pw.x output |

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

- profile 中应有可识别的 vacuum plateau。
- work function expression 和能量零点必须写入 record。
- 检查 vacuum thickness、dipole correction、slab 厚度和 k mesh 是否影响结果。
- work function 不是单行公式；若真空平台、平均方向、Fermi reference、偶极/边界条件或 charged-state 处理不可复查，应降级为 `WARN` 或 `BLOCK`。物理边界见 [physics-judgement/work-function-and-electrostatic-boundary.md](../../physics-judgement/work-function-and-electrostatic-boundary.md)。

## 收敛性要求

- vacuum level 需要随真空厚度和 slab 设置复查。
- Fermi level 来自同一 SCF，不应混用不同计算。
- 低维边界和偶极修正设置需要记录。

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| 没有平台 | 真空不足或方向错误 | 检查 slab cell 和 `idir` |
| 数值不可比 | 混用不同 Fermi reference | 统一 SCF 和 potential 数据源 |
| 表面两侧不一致 | 极性 slab 或偶极场 | 记录 dipole correction 和两侧平台 |

## 通用学习模板

使用 `<system>`、`<structure>`、`<pseudo>`、`<k_mesh>`、`<energy_window>`、`<output_format>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
pw.scf.<system>.in
pw.scf.<system>.out
pp.potential.<system>.in
average.<system>.in
work-function-review.<system>.md
record.md
```

## 与其他 workflow 的关系

- 依赖 electrostatic-potential workflow。
- 结构构建、slab、vacuum 细节在独立结构学习项目展开。

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE INPUT_PP reference: <https://www.quantum-espresso.org/Doc/INPUT_PP.html>
- QE PostProc guide: <https://www.quantum-espresso.org/Doc/pp_user_guide/>
