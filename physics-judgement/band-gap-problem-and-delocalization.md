# Band Gap Problem and Delocalization

## 本页解决什么问题

本页用于判断为什么 DFT band gap 是高风险结论。普通 bands/DOS workflow 给出的是 Kohn-Sham-level gap 或 Fermi-level crossing 证据；它可以服务趋势、模型对照和内部电子结构判断，但不能自动支撑准粒子、光学、输运或器件级结论。

## 典型 output 现象

| output 现象 | 可能含义 | 回查动作 |
|---|---|---|
| bands 显示小 gap 或 near crossing | 可能是真实小 gap、k-path 未覆盖、SOC/U 缺失或 smearing 边界 | 交叉审阅 DOS、Fermi level、kmesh、SOC/U |
| DOS 在 Fermi 附近接近零但 bands 有 crossing | uniform mesh、broadening、energy zero 或 path 覆盖不一致 | 回查 NSCF mesh、DOS 设置和 bands path |
| semilocal DFT gap 与预期差异大 | functional / delocalization / derivative discontinuity 边界 | 写 model boundary，不用加密 kmesh 解释一切 |
| DFT+U 或 hybrid 后 gap 变化 | 模型改变影响 localized states 或 exchange treatment | 记录 U/functional 来源和重新计算链 |
| PDOS 中局域态过宽或磁矩偏小 | delocalization、self-interaction 或 projector 解释边界 | 回查 PP、DFT+U、hybrid、Wannier 或 charge density 证据 |

## 可能原因

- 数值误差：kmesh、`nbnd`、smearing、energy grid、cutoff 或 SCF threshold 不足。
- 模型误差：semilocal functional 的 delocalization/self-interaction/derivative-discontinuity 边界。
- 赝势误差：valence、semicore、relativistic treatment 或 PP/functional mismatch 改变 gap。
- 边界条件误差：slab、charged system、有限尺寸或低维边界影响能级参考。
- 后处理误差：绘图零点、DOS broadening、PDOS projection 或 band sorting 未记录。
- workflow 传播误差：relax、SCF、NSCF、bands、DOS、PDOS 来自不同模型或 scratch。
- 真实物理效应：磁性、SOC、结构畸变、强关联或拓扑相关的 gap opening/closing 可能真实存在，但需要独立证据。

## 必查 QE input/output

| 对象 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `input_dft` | `pw.x` | XC 模型边界 | 只因 gap 不合预期就换 functional |
| `K_POINTS` | `pw.x` | path 与 uniform mesh 覆盖 | 用 band path 替代 DOS mesh |
| `occupations/smearing/degauss` | `pw.x` | Fermi 附近占据 | smearing 造成的填充被解释为金属性 |
| `nbnd` | `pw.x` | 导带和目标窗口覆盖 | 空带不足仍给 gap 或高能结论 |
| `noncolin/lspinorb` | `pw.x` | SOC 对 gap 和 degeneracy 的影响 | 重元素或近简并问题未做 SOC 边界 |
| `HUBBARD` card | `pw.x` | localized manifold 修正 | 为匹配 gap 随意调 U |
| PP loading | `pw.x` | valence / relativistic / functional consistency | 换 PP 后不重做收敛和 gap 审阅 |
| `fildos/filpdos/filband` | 后处理 | 图件数据来源 | 旧文件或不同模型混入同一图 |

## 判断规则

- Kohn-Sham gap、fundamental gap 和 optical gap 是不同对象。普通 QE bands/DOS 直接给出的是 DFT 模型下的 eigenvalue gap 或 Fermi-level crossing 证据。
- Semilocal functional 的 gap 问题不是普通 kmesh 或 cutoff 不足；数值收敛只能降低 numerical error，不能消除模型误差。
- “semilocal 常低估 gap”是经验性边界，不是对所有体系、所有磁态、所有 SOC 情况的统一定律。
- 小带隙、半金属、磁性、SOC、DFT+U、hybrid、GW/BSE 都会改变 gap 判断层级。
- DOS 与 bands 不一致时，先排查文件链、energy reference、kmesh、smearing 和 projection，再讨论物理机制。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | 作为 DFT-level gap 或 metallicity statement：SCF/NSCF/bands/DOS/PDOS 数据链一致，k-path/mesh/energy reference/模型边界清楚 | 允许写带 DFT 模型边界的趋势、相对比较或交叉审阅结论 |
| WARN | 数值链可追踪但 functional、SOC、U、hybrid/GW 或实验对照不足 | 只允许趋势或内部判断，不给定量激发态结论 |
| BLOCK | 直接把普通 DFT gap 写成准粒子、光学、实测或器件级结论；用 U/hybrid/GW 结果但不说明模型和收敛边界 | 停止发布 gap 结论，回到模型和来源审阅 |

## 不能做出的过度结论

- 不能说加密 k 点可以修复 semilocal band-gap problem。
- 不能只看 band plot 或 DOS 平滑曲线就给定量 gap。
- 不能没有 SOC 边界却讨论对 SOC 敏感的 gap、简并或 band inversion。
- 不能用 DFT+U/hybrid 改变 gap 后不记录模型来源和下游重算链。
- 不能把 PDOS 中某个峰的宽窄单独写成电荷局域或价态证据。

## 下游影响

- `bands/DOS/PDOS`：决定 gap、metallicity 和态成分解释能否进入记录。
- `Fermi surface`：小 gap 或半金属边界不清时，费米面图像可能不可定量解释。
- `DFT+U/SOC/hybrid`：gap 变化必须作为模型变化记录。
- `GW/BSE/TDDFT/XSpectra`：若研究目标是准粒子或光学响应，应进入 excited-state 边界。

## 与 theory-minimum / workflows / standards 的关系

- 理论回查：[DFT, Kohn-Sham and SCF](../theory-minimum/dft-ks-scf.md)、[Smearing and metals](../theory-minimum/smearing-metals.md)、[Magnetism, SOC and DFT+U](../theory-minimum/magnetism-soc-dftu.md)
- Workflow 回查：[bands](../workflows/electronic/bands.md)、[DOS](../workflows/electronic/dos.md)、[PDOS](../workflows/electronic/pdos.md)、[Fermi surface](../workflows/electronic/fermi-surface.md)
- 相关判断：[Kohn-Sham eigenvalue boundary](kohn-sham-eigenvalue-boundary.md)、[Functional choice and sensitivity](functional-choice-and-sensitivity.md)、[U value provenance](u-value-provenance-and-boundary.md)
- 记录规范：[output review checklist](../standards/output-review-checklist.md)、[pass-warn-block](../standards/pass-warn-block.md)

## 来源与结论强度

| 结论 | 强度 | 来源边界 |
|---|---|---|
| KS gap、fundamental gap、optical gap 需要区分 | Strong | KS 与 excited-state canonical literature |
| delocalization/self-interaction/derivative discontinuity 是模型误差来源 | Strong | Mori-Sanchez/Cohen/Yang、DFT error literature |
| semilocal gap 判断不能靠 kmesh 修复 | Boundary | 数值误差与模型误差区分 |
| U、SOC、hybrid、GW/BSE 的字段和 workflow | Version-sensitive | 当前 QE / external tool docs |

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE INPUT_BANDS reference: <https://www.quantum-espresso.org/Doc/INPUT_BANDS.html>
- Mori-Sanchez, Cohen and Yang, Current limitations of density functional theory.
- Perdew, Parr, Levy and Balduz, Density-functional theory for fractional particle number.
- Onida, Reining and Rubio, Electronic excitations: density-functional versus many-body Green's-function approaches.
- 本仓库：[canonical literature](../references/canonical-literature.md)、[source index](../references/source-index.md)
