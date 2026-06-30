# Band Gap Problem and Derivative Discontinuity

## 1. 这个主题解决什么判断问题？

本页帮助判断 QE bands/DOS 中的 Kohn-Sham gap 能解释什么，不能解释什么。

## 2. 物理图像

Kohn-Sham eigenvalue gap 不等同于 fundamental gap。近似泛函缺少或近似处理 derivative discontinuity，LDA/GGA 常低估带隙。光学 gap 还涉及激子和多体效应。

## 3. DFT/QE 中的近似来源

误差来自 exchange-correlation approximation、有限 k mesh、SOC、DFT+U/hybrid/GW 模型选择、DOS 展宽和能量零点处理。

## 4. 相关 QE 输入字段

| 字段 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| input_dft | pw.x | 定义交换关联或近似模型 | 把模型选择当作单纯数值收敛 |
| ecutwfc/ecutrho | pw.x | 确认基组误差不会主导判断 | 换物理模型后沿用旧 cutoff |
| K_POINTS 或 q-grid | pw.x/ph.x | 确认采样足以支撑目标性质 | 用绘图路径替代积分采样 |
| occupations/smearing/degauss | pw.x/dos.x | 控制占据和费米面附近积分 | 用过大 smearing 解释物理趋势 |
| nbnd | pw.x | 未占据带是否足以覆盖目标能区 | 带数不足却解释导带 |
| K_POINTS crystal_b | pw.x bands | 路径是否覆盖目标高对称线 | 用路径 gap 代替全 BZ gap |
| dos.x Emin/Emax/DeltaE | dos.x | 能量窗和分辨率是否足够 | DOS 能量零点不清 |

## 5. output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| functional / pseudopotential 回显 | 模型是否与记录一致 | input、UPF、source record |
| SCF / perturbation convergence | 数值解是否达到当前目标 | conv_thr、tr2_ph、mixing |
| warning / symmetry / occupation 信息 | 物理模型或边界条件可能不一致 | workflow record 和 output review |
| bands gap 与 DOS gap 不一致 | k-path、mesh、smearing 或能量零点问题 | bands/DOS workflow |
| 小带隙随 SOC 或泛函改变符号 | 拓扑/半金属判断敏感 | SOC、functional、k sampling |
| DFT gap 与实验差距大 | Kohn-Sham gap 解释边界 | hybrid/GW/BSE 或实验对照 |

## 6. 对 workflow 的影响

- [workflows/electronic/bands.md](../workflows/electronic/bands.md)
- [workflows/electronic/dos.md](../workflows/electronic/dos.md)
- [workflows/advanced/hybrid-functional-overview.md](../workflows/advanced/hybrid-functional-overview.md)

## 7. PASS / WARN / BLOCK 判断

| 状态 | 条件 | 允许进入的下游 |
|---|---|---|
| PASS | 已区分 KS gap、fundamental gap 和 optical gap；bands path、DOS mesh、SOC/smearing 已审阅 | 进入带隙趋势或限定解释 |
| WARN | gap 对 k mesh、SOC、泛函或 smearing 敏感，或只缺少 GW/BSE/实验对照 | 只作为趋势或起点 |
| BLOCK | 把普通 DFT KS gap 当定量实验 gap，或用 bands path 漏采样结果直接下结论 | 停止定量 gap 解释 |

## 8. 常见误区

- 把 Kohn-Sham gap 当作实验 fundamental gap
- 用 DOS smearing 造成的尾态判断金属性
- 忽略 SOC 对小带隙的影响
- 用 bands path 漏掉真实 VBM/CBM
- 把 scissor 当作自洽物理修正

## 9. 与前沿常识的关系

普通 DFT gap 常可作趋势；定量光谱或准粒子 gap 通常需要 hybrid、GW/BSE 或实验对照。小带隙和拓扑判断必须额外审阅 SOC、k mesh 和对称性。

## 10. 最低掌握深度

- 能区分 Kohn-Sham gap、fundamental gap 和 optical gap。
- 能检查 bands path 是否可能漏掉 VBM/CBM，以及 DOS dense mesh 是否支持 gap 判断。
- 能说明普通 DFT gap 何时只适合作为趋势，何时需要 hybrid、GW/BSE 或实验对照。
- 能识别 smearing、SOC 和小带隙体系对 gap 判断的风险。

## 11. 对应仓库页面

- [theory-minimum/kpoints-symmetry-kpath.md](../theory-minimum/kpoints-symmetry-kpath.md)
- [theory-minimum/magnetism-soc-dftu.md](../theory-minimum/magnetism-soc-dftu.md)
- [workflows/electronic/bands.md](../workflows/electronic/bands.md)
- [workflows/electronic/dos.md](../workflows/electronic/dos.md)
- [workflows/advanced/hybrid-functional-overview.md](../workflows/advanced/hybrid-functional-overview.md)
- [standards/output-review-checklist.md](../standards/output-review-checklist.md)
- [standards/pass-warn-block.md](../standards/pass-warn-block.md)
- [references/source-index.md](../references/source-index.md)


## 结论强度

- Strong：本页中由 DFT/DFPT 基础理论、经典方法论文或 QE 官方文档直接支撑的判断。
- Moderate：本页中由综述、方法摘要或多个来源共同支持，但细节需要回到正文或官方文档核验的判断。
- Boundary：本页中用于限制解释范围的判断，不作为定量结论。
- Version-sensitive：涉及 QE、PHonon、Wannier90、EPW、Yambo 或其他工具字段和行为的判断，必须按当前官方文档复查。

## 12. 资料来源

- [Perdew, Parr, Levy and Balduz, Phys. Rev. Lett. 49, 1691 (1982)](https://doi.org/10.1103/PhysRevLett.49.1691)
- [Onida, Reining and Rubio, Rev. Mod. Phys. 74, 601 (2002)](https://doi.org/10.1103/RevModPhys.74.601)
- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
