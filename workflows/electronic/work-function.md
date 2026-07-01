# Work-function workflow

## 本页解决什么问题

本页说明如何审阅 QE 中基于 slab electrostatic potential 的 work function workflow。work function 需要同一计算链中的 Fermi energy 和 vacuum level；它不是从任意 potential 图上读一个数，也不能在真空平台、slab 方向、偶极边界或能量参考不清楚时给出定量结论。

本页只说明 QE workflow 对结构输入和静电后处理的要求。slab 构建、表面终止、真空和结构标准化的详细操作属于独立结构学习主题。

## 页面定位

- 对应学习路线：[learn/07-postprocessing-loop.md](../../learn/07-postprocessing-loop.md)
- 上游依赖：[workflows/ground-state/scf.md](../ground-state/scf.md)、[workflows/electronic/electrostatic-potential.md](electrostatic-potential.md)
- 物理边界：[physics-judgement/work-function-and-electrostatic-boundary.md](../../physics-judgement/work-function-and-electrostatic-boundary.md)
- 规范入口：[standards/output-review-checklist.md](../../standards/output-review-checklist.md)

## 上游依赖

- slab-like 结构的方向、真空区域、表面终止、是否对称和是否带偶极已记录。
- SCF 为 `PASS`，并记录 Fermi energy、occupations/smearing、k mesh、cutoff 和 warning。
- potential profile 已通过 [electrostatic-potential.md](electrostatic-potential.md) 审阅。
- vacuum plateau、average direction、energy reference、dipole correction 或低维边界设置可复查。
- work function 公式中的 `E_vac` 和 `E_F` 来自同一结构、同一 SCF 和同一 `prefix/outdir` 数据链。

## 计算图

```text
slab-like <structure>
  -> final static SCF
  -> pp.x potential extraction
  -> average.x planar potential profile
  -> E_vac - E_F review
  -> PASS / WARN / BLOCK
```

## 关键 QE 输入对象

| 字段 / 设置 | 程序 | 控制什么 | 常见风险 | Output 中如何验证 |
|---|---|---|---|---|
| `occupations/smearing/degauss` | `pw.x` | Fermi energy 和金属占据 | Fermi reference 不稳定 | SCF output 的 Fermi energy 和 occupation |
| `prefix/outdir` | `pw.x`, `pp.x` | 保证 potential 与 SCF 同源 | 混用不同计算链 | SCF 和 `pp.x` output |
| `plot_num` | `pp.x` | 选择 potential 类型 | 势类型错误 | `INPUT_PP` 和 `pp.x` output |
| average direction | `average.x` | 沿 slab 法向求平均 | 方向与 vacuum 不一致 | averaged profile 横轴和 slab 方向 |
| `assume_isolated` / electric-field or dipole-related settings | `pw.x` | 低维/偶极边界入口 | 未记录边界处理却解释平台 | SCF input/output |
| slab/vacuum geometry | 结构输入 | vacuum plateau 和周期镜像 | 结构构建不可复查 | record 和 SCF output |

## 命令与文件边界

```bash
pw.x -in pw.scf.<system>.in > pw.scf.<system>.out
pp.x -in pp.potential.<system>.in > pp.potential.<system>.out
average.x < average.<system>.in > average.<system>.out
```

work function review 文件应记录：SCF output 中的 `E_F`、average profile 中选取的 `E_vac`、plateau 选择规则、能量单位、是否做图像平移、以及 `E_vac - E_F` 是否来自同一数据链。不要把绘图脚本里平移后的曲线数值当作原始 output 证据。

电势平均的审阅顺序应保持固定：先确认 `pp.x` 读取的 `prefix/outdir` 与 SCF 一致，再确认 `plot_num` 和平均方向，随后检查 planar average 是否有 vacuum plateau，最后才选择 `E_vac` 并与 SCF 的 `E_F` 做差。若使用 macroscopic average、偶极修正、外电场或低维边界设置，应在记录中说明它们改变了什么边界条件，以及它们不能单独证明 slab 已经收敛。

对绝缘体或半导体表面，`E_F` 可能反映占据约定、缺陷/表面态或绘图参考，而不一定等同于实验化学势位置。正式 work-function 陈述应说明采用的 Fermi-level 或 chemical-potential convention；若只给 `E_vac - E_F` 而没有该约定，本页最多进入 `WARN`。

## 通用审阅模板

```markdown
## Work-function review

- SCF output:
- Potential output:
- Average profile:
- Slab normal direction:
- Vacuum region:
- Fermi energy source:
- Vacuum level source:
- Dipole / boundary treatment:
- Work-function expression:
- PASS / WARN / BLOCK:
- Reason:
```

## Output review

| 检查项 | 从哪里看 | 能证明什么 | 不能证明什么 | WARN/BLOCK 触发 |
|---|---|---|---|---|
| SCF 可信度 | `pw.scf` output、SCF review | `E_F` 来自可审阅 ground state | slab electrostatics 已可信 | SCF 为 `BLOCK` 则 work function 为 `BLOCK` |
| slab 方向和 vacuum | structure record、CELL、profile 横轴 | 平均方向和真空区域可复查 | 真空足够或平台平坦 | slab 法向不清为 `WARN/BLOCK` |
| potential profile | `pp.x`、`average.x` output、profile data | `E_vac` 候选区可审阅 | 平台一定适合定量 | 无平坦 plateau 为 `BLOCK` |
| Fermi energy | SCF output | `E_F` 来源明确 | 其与所有后处理能量零点自动一致 | Fermi reference 未记录为 `BLOCK` |
| dipole/boundary | input/output、record | 偶极或低维边界处理可追踪 | 修正后必然收敛 | 非对称 slab 未处理且平台倾斜为 `WARN/BLOCK` |
| 两侧平台 | averaged profile | 非对称 slab 或偶极效应是否可见 | 两侧差异一定是物理效应 | 两侧平台不一致且无边界说明为 `WARN/BLOCK` |
| 同一数据链 | `prefix/outdir`、file timestamps、record | `E_vac` 与 `E_F` 同源 | model error 已消除 | 混用不同计算结果为 `BLOCK` |

## 收敛与可靠性

- work function 对结构、表面终止、slab 厚度、真空区域、偶极修正、k mesh、smearing 和电势平均方法敏感。
- 真空平台不平坦、平均方向错误或 `E_F` 来源不明时，不能给定量 work function。
- 对表面两侧不等价的 slab，应分别审阅两侧平台或说明采用的边界处理。
- work function 是后处理结果，可信度继承 ground-state SCF 和 electrostatic potential workflow。
- dipole correction、external field 或 isolated-boundary 设置应作为边界条件记录和敏感性对象；不要把它们写成自动修复真空平台的通用步骤。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| `PASS` | SCF 和 potential workflow 为 `PASS`；vacuum plateau 清楚；`E_F` 和 `E_vac` 同源；方向、单位、边界和公式已记录 | 允许写入受边界限制的 work-function 结果和 figure |
| `WARN` | plateau 可见但边界、偶极、真空或 slab 收敛仍不完整 | 只允许趋势或探索性比较 |
| `BLOCK` | 上游 SCF 为 `BLOCK`；无 vacuum plateau；Fermi level 未记录；potential reference 不明；slab 构型不可复查；混用不同数据链 | 不允许给出 work-function 数值 |

## 常见误区

- 把 work function 写成不带条件的单行公式。
- 从不平坦或倾斜的真空电势中取 `E_vac`。
- 混用不同计算的 Fermi energy 和 potential profile。
- 不记录 slab 法向、真空区域和偶极边界。
- 用可视化曲线截图替代 output review。
- 写固定真空厚度或 slab 厚度作为通用答案。

## 下游影响

work function 结果会影响 surface/interface comparison、band alignment、figures 和 publication record。若本页为 `WARN`，下游只能写趋势或方法边界；若为 `BLOCK`，所有基于该 work function 的结论都应停止。

## 来源与边界

- QE `pw.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE `pp.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_PP.html>
- QE PostProc guide: <https://www.quantum-espresso.org/Doc/pp_user_guide/>
- 物理边界：[physics-judgement/work-function-and-electrostatic-boundary.md](../../physics-judgement/work-function-and-electrostatic-boundary.md)
- 本仓库规范：[standards/output-review-checklist.md](../../standards/output-review-checklist.md)
