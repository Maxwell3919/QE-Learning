# Functional Choice and Sensitivity

## 本页解决什么问题

本页用于判断 exchange-correlation functional choice 是否足以支撑当前科研结论。functional 不是普通数值参数；改变 `input_dft`、hybrid、meta-GGA、vdW 或 DFT+U 设置，等于改变物理模型。模型改变后，结构、赝势、cutoff、k/q mesh、occupation 和所有下游结果都需要重新审阅。

## 典型 output 现象

| output 现象 | 可能含义 | 回查动作 |
|---|---|---|
| output 回显的 XC 与记录不一致 | `input_dft` 或 PP 元信息混用 | 核对 input、UPF 元信息和 SCF output |
| functional 切换后 lattice、force、stress 或 magnetization 变化明显 | 模型敏感性影响下游 | 不把差异归为普通收敛误差，写 model sensitivity |
| band gap、DOS near Fermi level 或 phonon 频率随 functional 改变 | 目标 observable 对模型敏感 | 回查 bands/DOS/phonon 数据链是否同一 functional |
| hybrid 或 meta-GGA 计算出现新的 warning / 收敛问题 | 版本、实现或数值设置敏感 | 查 QE 官方字段和当前版本限制 |
| relax 用一种 functional，bands/DOS/phonon 用另一种 functional | 数据链不一致 | 作为 `WARN` 或 `BLOCK` 处理，除非明确是模型对照 |

## 可能原因

- 数值误差：cutoff、k mesh、SCF threshold、smearing 未对目标 observable 收敛。
- 模型误差：LDA/GGA/meta-GGA/hybrid/vdW/DFT+U 描述不同物理近似。
- 赝势误差：PP 的 XC、valence、relativistic treatment 与 `input_dft` 不一致。
- 边界条件误差：surface、low-dimensional、charged 或 polar 系统的边界条件未与 functional 选择共同审阅。
- 后处理误差：同一图中混用不同 functional 的 energy reference、DOS broadening 或 projection。
- workflow 传播误差：结构用模型 A 优化，下游用模型 B 解释，却没有重新做 final static SCF 和后处理。
- 真实物理效应：某些 observable 对 exchange-correlation treatment 本来敏感，应写成模型依赖结论。

## 必查 QE input/output

| 对象 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `input_dft` | `pw.x` | 选择或覆盖 XC functional | 未记录 functional 或与 PP 不一致 |
| UPF metadata | `pw.x` / PP 文件 | PP 生成时的 XC、valence、relativistic 信息 | 只记录元素名，不记录实际 PP |
| `vdw_corr` | `pw.x` | dispersion 或 nonlocal correction 入口 | 当作对所有性质的单向改进 |
| hybrid / exact-exchange fields | `pw.x` | hybrid/screened hybrid 设置 | 只为改变 gap 而不重审结构和收敛 |
| `ecutfock` 等版本敏感字段 | `pw.x` | exact-exchange 数值设置 | 沿用普通 GGA 的数值策略 |
| forces / stress / phonon output | `pw.x` / `ph.x` | functional 对结构和响应的传播 | 换模型后直接复用旧下游 |

## 判断规则

- functional choice 是模型选择；cutoff 或 k mesh convergence 只能降低数值误差，不能证明该 functional 适合目标物理问题。
- `input_dft` 与 pseudopotential functional 应保持一致。若不一致，必须说明目的和边界；无说明时不能进入定量结论。
- LDA、GGA、meta-GGA、hybrid、vdW、DFT+U 不是线性精度阶梯。某个方法对一个 observable 更合适，不代表对 lattice、gap、magnetism、phonon、work function 或 vdW distance 都更合适。
- 比较不同 functional 时，应把每个 functional 当作独立模型：重新审阅结构、SCF、收敛、下游数据链和 energy reference。
- functional sensitivity 不能用“图像更接近预期”替代；应写清目标 observable、对照设置和仍未排除的模型误差。若同一 observable 同时对 PP、U、SOC、vdW 或 smearing 敏感，应把它写成模型组合的敏感性，而不是归因给单一 functional 标签。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | `input_dft`、PP functional、结构状态和下游数据链一致；目标 observable 的模型边界已写明；必要对照或 sensitivity 已记录 | 允许进入带边界的科研结论 |
| WARN | functional/PP 可追踪，但未充分做 functional sensitivity；当前结果只支撑趋势或学习判断 | 可进入探索性下游，结论必须写成模型依赖 |
| BLOCK | functional 与 PP 不一致且无说明；不同 functional 的结构、SCF 或后处理混用；把某 functional 写成普适提升 | 不允许进入定量比较或最终图件 |

## 不能做出的过度结论

- 不能说某个 rung、hybrid、meta-GGA、vdW 或 DFT+U 对所有性质更准。
- 不能把 functional 差异解释成 cutoff 或 k mesh 未收敛，除非已完成数值排查。
- 不能把用一种 functional relax 的结构直接用于另一种 functional 的高风险下游而不说明。
- 不能把 functional 改变后的 gap、磁矩或 phonon 差异写成充分物理机制。
- 不能把不同 functional 下的 total energy、stress、phonon 或 work function 混入同一张表而不标明模型分支。

## 下游影响

- `relax/vc-relax`：functional 改变 energy surface，会影响 forces、stress、cell 和内部坐标。
- `bands/DOS/PDOS`：functional 影响 KS eigenvalues、gap、orbital ordering 和 projection 解释。
- `phonon/DFPT`：functional 影响 force constants、soft modes、Born charge 和 dielectric response。
- `work function`：functional 通过结构、电势和 Fermi level 传播到表面性质。
- `vdW/DFT+U/hybrid`：这些设置应作为模型分支，而不是普通调参。

## 与 theory-minimum / workflows / standards 的关系

- 理论回查：[DFT, Kohn-Sham and SCF](../theory-minimum/dft-ks-scf.md)、[Pseudopotentials](../theory-minimum/pseudopotentials.md)
- Workflow 回查：[SCF](../workflows/ground-state/scf.md)、[bands](../workflows/electronic/bands.md)、[phonon dispersion](../workflows/phonon/phonon-dispersion-dfpt.md)、[hybrid overview](../workflows/advanced/hybrid-functional-overview.md)
- 判断回查：[Numerical vs model error](numerical-vs-model-error.md)、[Pseudopotential transferability](pseudopotential-transferability-and-functional-consistency.md)
- 记录规范：[output review checklist](../standards/output-review-checklist.md)、[calculation record template](../standards/calculation-record-template.md)

## 来源与结论强度

| 结论 | 强度 | 来源边界 |
|---|---|---|
| functional choice 是 DFT 模型近似 | Strong | HK/KS、PBE、QE `INPUT_PW` |
| 更高阶方法不自动保证所有 observable 更准 | Boundary | Jacob's ladder 与 functional error 文献；具体体系需另行验证 |
| `input_dft`、hybrid、vdW 字段行为依赖 QE 版本 | Version-sensitive | 当前 QE 官方 `INPUT_PW` |
| functional sensitivity 应写入结果边界 | Moderate | DFT error decomposition 与 reproducibility literature |

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- Perdew, Burke and Ernzerhof, Generalized Gradient Approximation Made Simple.
- Perdew et al., Jacob's ladder / exchange-correlation functional framework.
- Kim, Sim and Burke, Understanding and reducing errors in density functional theory.
- Lejaeghere et al., Reproducibility in density functional theory calculations of solids.
- 本仓库：[canonical literature](../references/canonical-literature.md)、[source index](../references/source-index.md)
