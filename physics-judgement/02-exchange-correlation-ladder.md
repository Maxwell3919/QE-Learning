# Exchange-Correlation Ladder

## 1. 这个主题解决什么判断问题？

本页帮助判断 exchange-correlation functional 选择如何改变结构、能带、磁性、phonon 和响应性质。

## 2. 物理图像

Jacob’s ladder 可作为理解 LDA、GGA、meta-GGA、hybrid 和 exact exchange/screened hybrid 的层次图。层级更高通常引入更多物理信息或非局域性，但不自动保证所有性质更准。

## 3. DFT/QE 中的近似来源

近似主要来自 exchange-correlation approximation，也会通过赝势生成泛函、结构优化、exact-exchange 采样、hybrid cutoff 和后处理链传播。泛函选择是模型选择，不是数值收敛参数。

## 4. 相关 QE 输入字段

| 字段 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| input_dft | pw.x | 定义交换关联或近似模型 | 把模型选择当作单纯数值收敛 |
| ecutwfc/ecutrho | pw.x | 确认基组误差不会主导判断 | 换物理模型后沿用旧 cutoff |
| K_POINTS 或 q-grid | pw.x/ph.x | 确认采样足以支撑目标性质 | 用绘图路径替代积分采样 |
| occupations/smearing/degauss | pw.x/dos.x | 控制占据和费米面附近积分 | 用过大 smearing 解释物理趋势 |
| exx_fraction/screening_parameter | pw.x | hybrid 或 screened hybrid 的模型参数 | 未记录来源或沿用默认理解 |
| ecutfock/nqx1/nqx2/nqx3 | pw.x | exact exchange 数值设置 | 未做资源和收敛审阅 |

## 5. output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| functional / pseudopotential 回显 | 模型是否与记录一致 | input、UPF、source record |
| SCF / perturbation convergence | 数值解是否达到当前目标 | conv_thr、tr2_ph、mixing |
| warning / symmetry / occupation 信息 | 物理模型或边界条件可能不一致 | workflow record 和 output review |
| lattice constant 或 phonon 对泛函敏感 | 结构和力常数受模型改变 | 重新 relax 与 phonon review |
| hybrid SCF 资源异常 | exact exchange 设置或并行策略不合适 | hybrid workflow 与 HPC record |

## 6. 对 workflow 的影响

- [workflows/advanced/hybrid-functional-overview.md](../workflows/advanced/hybrid-functional-overview.md)
- [workflows/ground-state/relax.md](../workflows/ground-state/relax.md)
- [workflows/electronic/bands.md](../workflows/electronic/bands.md)

## 7. PASS / WARN / BLOCK 判断

| 状态 | 条件 | 允许进入的下游 |
|---|---|---|
| PASS | 泛函、赝势 XC、结构优化、cutoff/k mesh 和后处理均在同一模型下审阅；hybrid/meta-GGA 的资源和采样风险已记录 | 进入结构、bands、phonon 或趋势比较 |
| WARN | 不同泛函趋势差异明显，或 hybrid/meta-GGA 设置尚未完成完整下游收敛 | 只做模型敏感性说明 |
| BLOCK | 赝势泛函与 `input_dft` 不一致，或换泛函后沿用旧结构/旧下游结论 | 停止定量比较，重建模型链 |

## 8. 常见误区

- 把高阶泛函当作普适更准
- 比较泛函时不重做结构优化
- 赝势泛函与 `input_dft` 不一致
- hybrid 下游仍沿用 GGA 的 bands/DOS 解释
- 只用一个性质评价泛函好坏

## 9. 与前沿常识的关系

GGA 带隙低估、meta-GGA/hybrid 改善某些性质但可能改变结构和磁性；r2SCAN/SCAN 等应作为模型选择审阅，不提供材料特定推荐。

## 10. 最低掌握深度

- 能区分 LDA/GGA/meta-GGA/hybrid 是物理模型层级，而不是数值收敛序列。
- 能检查 `input_dft`、赝势 XC 元信息、结构优化和后处理是否使用同一模型。
- 能说明 hybrid 或 exact-exchange 设置为什么需要额外审阅 `ecutfock`、Fock q mesh 和资源消耗。
- 能在比较泛函时写清哪些差异来自模型选择，哪些来自数值收敛。

## 11. 对应仓库页面

- [theory-minimum/dft-ks-scf.md](../theory-minimum/dft-ks-scf.md)
- [theory-minimum/pseudopotentials.md](../theory-minimum/pseudopotentials.md)
- [workflows/advanced/hybrid-functional-overview.md](../workflows/advanced/hybrid-functional-overview.md)
- [workflows/ground-state/relax.md](../workflows/ground-state/relax.md)
- [workflows/electronic/bands.md](../workflows/electronic/bands.md)
- [standards/output-review-checklist.md](../standards/output-review-checklist.md)
- [standards/pass-warn-block.md](../standards/pass-warn-block.md)
- [references/source-index.md](../references/source-index.md)


## 结论强度

- Strong：本页中由 DFT/DFPT 基础理论、经典方法论文或 QE 官方文档直接支撑的判断。
- Moderate：本页中由综述、方法摘要或多个来源共同支持，但细节需要回到正文或官方文档核验的判断。
- Boundary：本页中用于限制解释范围的判断，不作为定量结论。
- Version-sensitive：涉及 QE、PHonon、Wannier90、EPW、Yambo 或其他工具字段和行为的判断，必须按当前官方文档复查。

## 12. 资料来源

- [Perdew and Schmidt, AIP Conf. Proc. 577, 1 (2001)](https://doi.org/10.1063/1.1390175)
- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
- [Quantum ESPRESSO documentation](https://www.quantum-espresso.org/documentation/)
