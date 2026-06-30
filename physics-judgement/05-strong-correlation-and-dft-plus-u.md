# Strong Correlation and DFT+U

## 1. 这个主题解决什么判断问题？

本页帮助判断局域 d/f 电子、Mott/charge-transfer 物理和 DFT+U 修正是否影响 QE 结论。

## 2. 物理图像

强关联体系中，电子局域化、轨道占据和磁性构型可能决定基态。DFT+U 通过对选定局域子空间加入能量惩罚，减少某些过度离域问题，但它本身是带参数的模型。

## 3. DFT/QE 中的近似来源

近似来源包括 GGA/LDA delocalization、局域投影定义、U/J/V 参数来源、赝势 valence、磁性构型和结构优化。

## 4. 相关 QE 输入字段

| 字段 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| input_dft | pw.x | 定义交换关联或近似模型 | 把模型选择当作单纯数值收敛 |
| ecutwfc/ecutrho | pw.x | 确认基组误差不会主导判断 | 换物理模型后沿用旧 cutoff |
| K_POINTS 或 q-grid | pw.x/ph.x | 确认采样足以支撑目标性质 | 用绘图路径替代积分采样 |
| occupations/smearing/degauss | pw.x/dos.x | 控制占据和费米面附近积分 | 用过大 smearing 解释物理趋势 |
| HUBBARD card | pw.x | 定义作用轨道和 U/J/V | 不记录 U 来源或语法版本 |
| lda_plus_u/Hubbard_U(i) | pw.x | legacy DFT+U 入口 | 混用旧语法与新 card |
| starting_magnetization | pw.x | 探索磁性构型 | 只算一个初态 |

## 5. output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| functional / pseudopotential 回显 | 模型是否与记录一致 | input、UPF、source record |
| SCF / perturbation convergence | 数值解是否达到当前目标 | conv_thr、tr2_ph、mixing |
| warning / symmetry / occupation 信息 | 物理模型或边界条件可能不一致 | workflow record 和 output review |
| occupation matrix 不合理 | 投影子空间或 U 设置可疑 | HUBBARD、pseudo、magnetism |
| 不同 U 改变结构/磁性 | 模型变化显著 | 重新 relax 与 bands/phonon |
| SCF 收敛到不同局域态 | 多稳态或初态依赖 | 多初始条件审阅 |

## 6. 对 workflow 的影响

- [workflows/advanced/dft-plus-u.md](../workflows/advanced/dft-plus-u.md)
- [workflows/advanced/spin-polarization.md](../workflows/advanced/spin-polarization.md)
- [workflows/electronic/pdos.md](../workflows/electronic/pdos.md)

## 7. PASS / WARN / BLOCK 判断

| 状态 | 条件 | 允许进入的下游 |
|---|---|---|
| PASS | U/J/V 来源、`HUBBARD` 语法版本、投影子空间、occupation matrix 和多磁态比较均可追踪 | 进入 DFT+U 下游 workflow |
| WARN | U 来源合理但结果对 U 或初始磁性态敏感 | 只做参数敏感性或模型比较 |
| BLOCK | 为匹配单一 band gap 调 U，或不记录 U 来源/投影通道/语法版本 | 停止强关联结论 |

## 8. 常见误区

- 为了匹配带隙随意调 U
- 不记录 U 来源
- 不同 U 总能直接比较却不说明模型变化
- 认为 DFT+U 适合所有强关联问题
- 忽略结构和 phonon 对 U 的敏感性

## 9. 与前沿常识的关系

U 可来自文献、linear response、hp.x 或自洽策略。U 的来源和版本是物理模型的一部分，不能在记录中省略。

## 10. 最低掌握深度

- 能说明 DFT+U 的物理意图是修正指定局域子空间，而不是通用拟合旋钮。
- 能检查 U/J/V 来源、`HUBBARD` 语法版本、赝势标签和 occupation matrix。
- 能设计至少两个初始磁性/占据状态的审阅策略，避免把单一亚稳态当基态。
- 能判断 DFT+U 对结构、bands、phonon 和总能比较引入了新的模型边界。

## 11. 对应仓库页面

- [theory-minimum/magnetism-soc-dftu.md](../theory-minimum/magnetism-soc-dftu.md)
- [theory-minimum/pseudopotentials.md](../theory-minimum/pseudopotentials.md)
- [workflows/advanced/dft-plus-u.md](../workflows/advanced/dft-plus-u.md)
- [workflows/advanced/spin-polarization.md](../workflows/advanced/spin-polarization.md)
- [workflows/electronic/pdos.md](../workflows/electronic/pdos.md)
- [standards/output-review-checklist.md](../standards/output-review-checklist.md)
- [standards/pass-warn-block.md](../standards/pass-warn-block.md)
- [references/source-index.md](../references/source-index.md)


## 结论强度

- Strong：本页中由 DFT/DFPT 基础理论、经典方法论文或 QE 官方文档直接支撑的判断。
- Moderate：本页中由综述、方法摘要或多个来源共同支持，但细节需要回到正文或官方文档核验的判断。
- Boundary：本页中用于限制解释范围的判断，不作为定量结论。
- Version-sensitive：涉及 QE、PHonon、Wannier90、EPW、Yambo 或其他工具字段和行为的判断，必须按当前官方文档复查。

## 12. 资料来源

- [Cococcioni and de Gironcoli, Phys. Rev. B 71, 035105 (2005)](https://doi.org/10.1103/PhysRevB.71.035105)
- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
- [Quantum ESPRESSO documentation](https://www.quantum-espresso.org/documentation/)
