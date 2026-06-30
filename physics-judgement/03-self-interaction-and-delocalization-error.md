# Self-Interaction and Delocalization Error

## 1. 这个主题解决什么判断问题？

本页帮助识别 GGA/LDA 可能导致电子过度离域、磁矩偏小、缺陷态过浅或局域轨道占据不合理的情况。

## 2. 物理图像

精确泛函应避免电子与自身电荷相互作用；近似泛函常残留 self-interaction error，并倾向把电荷分布过度离域。局域 d/f 电子、缺陷态、小极化子和电荷转移体系尤其敏感。

## 3. DFT/QE 中的近似来源

误差来自 exchange-correlation approximation，也与赝势价态、投影轨道、DFT+U/hybrid 选择、supercell 和 PDOS/Wannier 后处理相关。

## 4. 相关 QE 输入字段

| 字段 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| input_dft | pw.x | 定义交换关联或近似模型 | 把模型选择当作单纯数值收敛 |
| ecutwfc/ecutrho | pw.x | 确认基组误差不会主导判断 | 换物理模型后沿用旧 cutoff |
| K_POINTS 或 q-grid | pw.x/ph.x | 确认采样足以支撑目标性质 | 用绘图路径替代积分采样 |
| occupations/smearing/degauss | pw.x/dos.x | 控制占据和费米面附近积分 | 用过大 smearing 解释物理趋势 |
| HUBBARD card | pw.x | 对指定局域轨道施加 U 修正 | 为了匹配单一带隙随意调 U |
| nspin/starting_magnetization | pw.x | 检查局域磁矩是否形成 | 把初始磁矩当成最终磁性 |
| projwfc.x 投影设置 | projwfc.x | 审阅局域轨道占据和 PDOS | 把投影不完备当作精确轨道分解 |

## 5. output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| functional / pseudopotential 回显 | 模型是否与记录一致 | input、UPF、source record |
| SCF / perturbation convergence | 数值解是否达到当前目标 | conv_thr、tr2_ph、mixing |
| warning / symmetry / occupation 信息 | 物理模型或边界条件可能不一致 | workflow record 和 output review |
| 磁矩系统性偏小 | 过度离域或磁性构型未充分检查 | spin/DFT+U/hybrid |
| PDOS 局域峰过度展宽 | 投影、smearing 或泛函误差 | projwfc 与 smearing |
| 缺陷/电荷态分布不合理 | self-interaction 或 finite-size artifact | supercell、charge correction、functional |

## 6. 对 workflow 的影响

- [workflows/advanced/dft-plus-u.md](../workflows/advanced/dft-plus-u.md)
- [workflows/electronic/pdos.md](../workflows/electronic/pdos.md)
- [workflows/advanced/wannier90-overview.md](../workflows/advanced/wannier90-overview.md)

## 7. PASS / WARN / BLOCK 判断

| 状态 | 条件 | 允许进入的下游 |
|---|---|---|
| PASS | 磁矩、PDOS、occupation/localization 和结构/电荷态证据相互支持，且模型限制已记录 | 进入趋势解释或下游模型比较 |
| WARN | 局域态、磁矩或 band gap 对泛函、U、hybrid 或 supercell 敏感 | 只写离域误差风险和限定结论 |
| BLOCK | 用单一 GGA 图像解释明显局域 d/f、缺陷态或小极化子问题，且无投影/磁矩/电荷证据 | 停止物理解释，补模型或证据链 |

## 8. 常见误区

- 把 GGA 结果当作局域电子定量结论
- 不记录 U 或 hybrid 参数来源
- 只看总能不看轨道占据和磁矩
- 用 PDOS 单独证明价态
- 忽略 supercell 与电荷态误差

## 9. 与前沿常识的关系

DFT+U、hybrid、localized orbital 和 Wannier 分析可帮助识别离域误差，但它们引入新的模型选择，不能当作自动修复。

## 10. 最低掌握深度

- 能从磁矩偏小、band gap 偏小、PDOS 过度展宽、局域态扩散等 output 迹象识别离域误差风险。
- 能说明 DFT+U、hybrid、PDOS 和 Wannier 分析分别能提供什么证据，不能自动证明什么。
- 能把缺陷态、电荷态或局域 d/f 轨道判断写成证据链，而不是只看一张 DOS 图。
- 能在 record 中区分泛函误差、投影不完备和 finite-size artifact。

## 11. 对应仓库页面

- [theory-minimum/pseudopotentials.md](../theory-minimum/pseudopotentials.md)
- [theory-minimum/magnetism-soc-dftu.md](../theory-minimum/magnetism-soc-dftu.md)
- [workflows/advanced/dft-plus-u.md](../workflows/advanced/dft-plus-u.md)
- [workflows/electronic/pdos.md](../workflows/electronic/pdos.md)
- [workflows/advanced/wannier90-overview.md](../workflows/advanced/wannier90-overview.md)
- [standards/output-review-checklist.md](../standards/output-review-checklist.md)
- [standards/pass-warn-block.md](../standards/pass-warn-block.md)
- [references/source-index.md](../references/source-index.md)


## 结论强度

- Strong：本页中由 DFT/DFPT 基础理论、经典方法论文或 QE 官方文档直接支撑的判断。
- Moderate：本页中由综述、方法摘要或多个来源共同支持，但细节需要回到正文或官方文档核验的判断。
- Boundary：本页中用于限制解释范围的判断，不作为定量结论。
- Version-sensitive：涉及 QE、PHonon、Wannier90、EPW、Yambo 或其他工具字段和行为的判断，必须按当前官方文档复查。

## 12. 资料来源

- [Cohen, Mori-Sanchez and Yang, Science 321, 792 (2008)](https://doi.org/10.1126/science.1158722)
- [Cococcioni and de Gironcoli, Phys. Rev. B 71, 035105 (2005)](https://doi.org/10.1103/PhysRevB.71.035105)
- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
