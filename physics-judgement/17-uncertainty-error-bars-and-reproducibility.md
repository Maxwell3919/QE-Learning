# Uncertainty, Error Bars and Reproducibility

## 1. 这个主题解决什么判断问题？

本页帮助把 DFT 结果写成带不确定性边界的判断，而不是只给单个数值。

## 2. 物理图像

DFT 不确定性至少包括数值不确定性、模型不确定性、赝势不确定性、finite-size 误差和 workflow 传播误差。可靠记录要说明哪些误差已通过收敛测试降低，哪些仍属于模型选择。

## 3. DFT/QE 中的近似来源

误差来源包括 convergence tolerance、functional choice、pseudopotential、finite k/q mesh、supercell/vacuum、smearing、DFPT response、post-processing 和 provenance。


### 通用误差来源表

| 误差来源 | 可否通过收敛测试降低 | 需要记录什么 | 对结论的影响 |
|---|---|---|---|
| cutoff / FFT grid | 可以 | `ecutwfc`、`ecutrho`、赝势、目标量变化 | 影响总能、力、应力、phonon |
| k/q sampling | 可以 | mesh、shift、irreducible points、q-grid | 影响金属、DOS、phonon、EPC |
| exchange-correlation functional | 不能只靠收敛测试 | `input_dft`、结构是否重优化、比较策略 | 改变结构、gap、磁性、phonon |
| pseudopotential | 不能只靠单次收敛测试 | UPF 来源、版本、XC、valence、relativistic 信息 | 改变所有下游 workflow |
| finite-size / boundary | 需要尺寸或边界模型测试 | cell、vacuum、charge correction、isolated model | 影响表面、缺陷、二维和电势 |
| workflow propagation | 通过 provenance 和重复审阅降低 | `prefix/outdir`、command、QE version、record | 混用上游会让结论不可追溯 |

## 4. 相关 QE 输入字段

| 字段 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| conv_thr/tr2_ph | pw.x/ph.x | 数值残差是否足够小 | 只看作业结束 |
| input_dft/HUBBARD/vdw_corr | pw.x | 模型选择 | 不记录模型来源 |
| prefix/outdir | 多程序 | 数据链可追溯 | 混用 scratch |
| QE version/command | 运行环境 | 可复现性 | 只保存图不保存命令 |

## 5. output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| 结果对 cutoff/k mesh 敏感 | 数值误差未收敛 | convergence report |
| 不同泛函给出不同结论 | 模型不确定性 | functional comparison |
| record 缺少 pseudo/source | 结果不可追溯 | citation/source policy |

## 6. 对 workflow 的影响

- [standards/calculation-record-template.md](../standards/calculation-record-template.md)
- [standards/output-review-checklist.md](../standards/output-review-checklist.md)
- [learn/08-hpc-and-reproducibility.md](../learn/08-hpc-and-reproducibility.md)
- [workflows/ground-state/cutoff-convergence.md](../workflows/ground-state/cutoff-convergence.md)
- [workflows/ground-state/kpoint-convergence.md](../workflows/ground-state/kpoint-convergence.md)
- [workflows/phonon/phonon-debugging.md](../workflows/phonon/phonon-debugging.md)

## 7. PASS / WARN / BLOCK 判断

| 状态 | 条件 | 允许进入的下游 |
|---|---|---|
| PASS | 主要误差来源已记录，关键数值误差低于目标判断需求 | 进入结论或下游 |
| WARN | 仍有模型或数值不确定性，但足以支持限定趋势 | 进入带限制说明的记录 |
| BLOCK | 缺少来源、收敛或数据链证据 | 停止定量结论 |

## 8. 常见误区

- 只报告单个数值
- 把收敛阈值当误差条
- 没有 functional/pseudo uncertainty
- 图像可复现但数据链不可复现
- 忽略 workflow propagation error

## 9. 与前沿常识的关系

近年计算材料研究越来越强调 reproducibility、workflow provenance 和 sensitivity analysis。误差说明不必总是统计误差条，但必须说清主要不确定性来源。

## 10. 最低掌握深度

- 能区分 numerical uncertainty 与 model uncertainty
- 能写出误差来源表
- 能说明哪些误差可收敛、哪些需模型对照
- 能把 PASS/WARN/BLOCK 写入 record

## 11. 对应仓库页面

- [standards/calculation-record-template.md](../standards/calculation-record-template.md)
- [standards/citation-and-source-policy.md](../standards/citation-and-source-policy.md)
- [theory-minimum/dft-ks-scf.md](../theory-minimum/dft-ks-scf.md)
- [standards/output-review-checklist.md](../standards/output-review-checklist.md)
- [standards/pass-warn-block.md](../standards/pass-warn-block.md)
- [references/source-index.md](../references/source-index.md)


## 结论强度

- Strong：本页中由 DFT/DFPT 基础理论、经典方法论文或 QE 官方文档直接支撑的判断。
- Moderate：本页中由综述、方法摘要或多个来源共同支持，但细节需要回到正文或官方文档核验的判断。
- Boundary：本页中用于限制解释范围的判断，不作为定量结论。
- Version-sensitive：涉及 QE、PHonon、Wannier90、EPW、Yambo 或其他工具字段和行为的判断，必须按当前官方文档复查。

## 12. 资料来源

- [Quantum ESPRESSO documentation](https://www.quantum-espresso.org/documentation/)
- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
- [Baroni et al., Rev. Mod. Phys. 73, 515 (2001)](https://doi.org/10.1103/RevModPhys.73.515)
