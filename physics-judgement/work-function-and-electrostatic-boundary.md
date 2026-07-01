# Work Function and Electrostatic Boundary

## 本页解决什么问题

本页用于判断 work function、electrostatic potential、vacuum level 和 slab electrostatics 的结果是否可用于科研结论。work function 不能从任意 potential 图上读出一个差值；它依赖同一条 SCF -> `pp.x` -> planar average -> output review 的证据链。

表面、界面和低维模型中，能量零点、真空平台、偶极、表面终止、Fermi level、slab 厚度、vacuum 区域和 smearing 都可能改变结论。若这些边界没有记录，数值结果只能作为未完成审阅的中间量。

## 典型 output 现象

| 现象 | 可能提示 | 不能直接说明什么 |
|---|---|---|
| planar average 在 vacuum 区域没有平台 | vacuum 不足、偶极场、带电边界或平均方向错误 | 不能报告定量 work function |
| Fermi energy 来源不清 | SCF/DOS 数据链或 occupation 设置不完整 | 不能定义同一能量参考下的差值 |
| potential 曲线有斜率 | 非对称 slab、外电场、偶极效应或周期镜像 | 不能把图中最高点当 vacuum level |
| 重新计算后 work function 漂移 | slab/vacuum/functional/smearing 敏感 | 不能只用一次单点结果写稳定结论 |
| `pp.x` 正常结束但图像异常 | 后处理读取成功，不代表上游可信 | 不能用可视化代替 output review |

## 可能原因

- 数值误差：SCF 未充分收敛，k mesh、smearing、cutoff 或 potential grid 不足。
- 模型误差：functional、vdW、SOC、DFT+U 或 pseudopotential 改变 surface electronic structure。
- 赝势误差：PP 与 functional、relativistic setting 或 target observable 不一致。
- 边界条件误差：slab 太薄、vacuum 不足、asymmetric slab、dipole correction 未审阅、charged slab 周期镜像未处理。
- 后处理误差：`plot_num` 选错，average direction 错误，potential reference 未说明，旧输出文件被覆盖。
- workflow 传播误差：relax、final SCF、DOS/potential 使用不同 `prefix/outdir` 或不同模型。
- 真实物理效应：表面偶极、吸附诱导 charge redistribution、极性表面或外场确实改变 vacuum level。

## 必查 QE input/output

| 对象 | 程序或文件 | 检查意义 |
|---|---|---|
| upstream final SCF | `pw.x` output | 确认 structure、PP、functional、occupation、Fermi energy 和 warning |
| slab cell、surface normal、vacuum region | input / record | 确认平均方向和周期边界 |
| `prefix/outdir` | `pw.x`、`pp.x` | 确认 potential 来自当前 SCF 数据 |
| `plot_num` | `pp.x` | 确认提取的是目标 electrostatic/local potential |
| planar average / macroscopic average 输入 | `average.x` 或等价流程 | 确认平均方向、单位和数据源 |
| `tefield`、`dipfield`、`assume_isolated`、ESM 相关设置 | `pw.x` | 判断偶极、外场或低维边界处理 |
| Fermi energy | SCF / DOS output | work function 差值必须使用同一计算链中的能量参考 |
| warnings / convergence / occupation | output | 判断 potential 与 Fermi level 是否可解释 |

## 判断规则

1. work function 至少需要同一计算链中的 vacuum level 与 Fermi energy。二者来源、单位和 energy reference 必须可追溯。
2. vacuum plateau 是必要证据。没有稳定平台时，不能从曲线局部读数给定量 work function。
3. asymmetric slab、外电场、charged slab 或强表面偶极需要单独说明 boundary condition；否则 work function 只能作为未完成审阅结果。
4. visualization 只帮助发现问题，不构成物理证明。结论必须回到 SCF、`pp.x`、average 和 calculation record。
5. 跨 slab thickness、vacuum、dipole setting、functional 或 termination 的比较必须写清模型和边界差异。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | SCF、slab 构型、potential file、average direction、vacuum plateau、Fermi energy、energy reference、boundary condition 和 warning 均已审阅 | 可报告同一模型下的 work function 或趋势 |
| WARN | plateau 可读但 vacuum/slab/dipole/smearing sensitivity 未完全审阅 | 只允许写趋势或内部比较 |
| BLOCK | 无 vacuum plateau；Fermi level 未记录；potential reference 不明；slab 构型或平均方向不清；从图像直接读数 | 停止定量 work function 结论 |

## 不能做出的过度结论

- 不能用未审查的 electrostatic potential 图给定量 work function。
- 不能跨不同 slab thickness、vacuum、dipole correction 或 functional 直接比较数值。
- 不能把 `pp.x` 正常结束当作 electrostatic boundary 已可信。
- 不能把 semiconductor/insulator surface 的 Fermi-level convention 省略后仍给强定量结论。
- 不能把 surface work function 的变化全部归因于化学效应而不审阅结构、smearing 和电势平台。

## 下游影响

- `workflows/electronic/work-function.md`：决定 work function 表述是否可进入正式记录。
- `workflows/electronic/electrostatic-potential.md`：决定 potential extraction 和 average 是否可解释。
- `physics-judgement/vdw-and-low-dimensional-boundary.md`：vdW、vacuum 和 slab electrostatics 常互相耦合。
- `physics-judgement/uncertainty-statement-template.md`：work function 需要带 boundary 和 convergence statement。

## 与 theory-minimum / workflows / standards 的关系

- 最低理论回查：[theory-minimum/dft-ks-scf.md](../theory-minimum/dft-ks-scf.md)、[theory-minimum/smearing-metals.md](../theory-minimum/smearing-metals.md)、[theory-minimum/force-stress-relaxation.md](../theory-minimum/force-stress-relaxation.md)。
- workflow 回查：[workflows/electronic/electrostatic-potential.md](../workflows/electronic/electrostatic-potential.md)、[workflows/electronic/work-function.md](../workflows/electronic/work-function.md)、[workflows/ground-state/scf.md](../workflows/ground-state/scf.md)。
- 记录规范：[standards/output-review-checklist.md](../standards/output-review-checklist.md)、[standards/calculation-record-template.md](../standards/calculation-record-template.md)。

## 来源与结论强度

| 结论 | 强度 | 来源边界 |
|---|---|---|
| work function 需要 vacuum level 与 Fermi energy 的同一数据链 | Strong | surface electrostatics 基本定义与 workflow 审阅 |
| `pp.x` potential extraction 与 `plot_num` 属于 QE 后处理字段，需要查官方文档 | Version-sensitive | QE `INPUT_PP` 与 PostProc guide |
| `tefield/dipfield/assume_isolated` 等边界字段随 QE 版本和计算目标变化 | Version-sensitive | QE `INPUT_PW` |
| vacuum plateau 不稳定时只能做趋势或停止结论 | Boundary | 本仓库 output review 与 work function workflow |

## 资料来源

- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)：电场、偶极、低维边界和 SCF 输入字段。
- [QE INPUT_PP](https://www.quantum-espresso.org/Doc/INPUT_PP.html)：`pp.x` 后处理输入字段。
- [QE PostProc documentation](https://www.quantum-espresso.org/Doc/pp_user_guide/)：`pp.x`、`average.x` 相关后处理入口。
- [workflows/electronic/work-function.md](../workflows/electronic/work-function.md)：本仓库 work function workflow。
- [references/canonical-literature.md](../references/canonical-literature.md)：canonical literature 分级。
