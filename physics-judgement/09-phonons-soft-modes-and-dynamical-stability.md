# Phonons, Soft Modes and Dynamical Stability

## 1. 这个主题解决什么判断问题？

本页帮助判断 phonon 虚频、软模和动力学稳定性结论是否可信。

## 2. 物理图像

声子频率来自 dynamical matrix 的本征值。acoustic modes 对整体平移应接近零；soft mode 表示某个集体位移方向曲率很小。imaginary/negative frequency 可能来自真实相不稳定，也可能来自数值噪声或上游结构问题。

## 3. DFT/QE 中的近似来源

近似来源包括 SCF 精度、结构残余力、cutoff、k/q sampling、smearing、ASR、IFC 插值、二维/极性长程相互作用和 harmonic approximation。

## 4. 相关 QE 输入字段

| 字段 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| input_dft | pw.x | 定义交换关联或近似模型 | 把模型选择当作单纯数值收敛 |
| ecutwfc/ecutrho | pw.x | 确认基组误差不会主导判断 | 换物理模型后沿用旧 cutoff |
| K_POINTS 或 q-grid | pw.x/ph.x | 确认采样足以支撑目标性质 | 用绘图路径替代积分采样 |
| occupations/smearing/degauss | pw.x/dos.x | 控制占据和费米面附近积分 | 用过大 smearing 解释物理趋势 |
| tr2_ph | ph.x | 响应方程收敛 | 默认粗略设置进入软模判断 |
| ldisp/nq1/nq2/nq3 | ph.x | q-grid 与 IFC 范围 | q-grid 太粗解释 dispersion |
| `zasr` | `q2r.x` | 在 IFC 转换层施加 acoustic sum rule | 用 ASR 掩盖未收敛问题 |
| `asr` | `matdyn.x/dynmat.x` | 在频率/模式后处理层施加 acoustic sum rule | 未区分后处理修正和真实收敛 |

## 5. output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| functional / pseudopotential 回显 | 模型是否与记录一致 | input、UPF、source record |
| SCF / perturbation convergence | 数值解是否达到当前目标 | conv_thr、tr2_ph、mixing |
| warning / symmetry / occupation 信息 | 物理模型或边界条件可能不一致 | workflow record 和 output review |
| Gamma acoustic modes 明显偏离零 | 结构、ASR 或数值收敛问题 | relax、SCF、ASR |
| 局部 q 点小虚频 | q-grid、k mesh、smearing 或真实软模 | 参数扫描和模式分析 |
| 整支频率漂移 | cutoff/IFC 插值不足 | cutoff 和 q-grid convergence |

## 6. 对 workflow 的影响

- [workflows/phonon/phonon-debugging.md](../workflows/phonon/phonon-debugging.md)
- [workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)
- [workflows/phonon/phonon-dos.md](../workflows/phonon/phonon-dos.md)

## 7. PASS / WARN / BLOCK 判断

| 状态 | 条件 | 允许进入的下游 |
|---|---|---|
| PASS | 结构、SCF、cutoff、k/q grid、smearing、ASR 和模式分析均支持频率结论 | 进入 phonon DOS、热力学或稳定性讨论 |
| WARN | 小虚频、声学支漂移或局部 soft mode 对参数敏感 | 只做待复查软模记录 |
| BLOCK | 未经复查直接把虚频判为真实不稳定，或用 ASR 替代收敛测试 | 停止动力学稳定性结论 |

## 8. 常见误区

- 把任何虚频都当真实不稳定
- 用 ASR 替代收敛测试
- 未优化结构就解释 soft mode
- q-grid 太粗却解释细节
- 忽略有限温度和 anharmonicity 边界

## 9. 与前沿常识的关系

动力学稳定性不同于热力学稳定性；0 K harmonic phonon 不能自动代表有限温度自由能稳定。软模需要结合模式、收敛和物理边界判断。

## 10. 最低掌握深度

- 能从 Gamma acoustic modes、ASR 前后、残余力、cutoff、k/q mesh 和 smearing 判断虚频风险来源。
- 能区分数值噪声、结构未平衡、q-grid 插值不足和真实软模。
- 能说明 dynamical stability 与 thermodynamic stability 的区别。
- 能把 negative frequency 结论写成带证据的 PASS/WARN/BLOCK，而不是直接下物理结论。

## 11. 对应仓库页面

- [theory-minimum/dfpt-phonons.md](../theory-minimum/dfpt-phonons.md)
- [theory-minimum/force-stress-relaxation.md](../theory-minimum/force-stress-relaxation.md)
- [workflows/phonon/phonon-debugging.md](../workflows/phonon/phonon-debugging.md)
- [workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)
- [workflows/phonon/phonon-dos.md](../workflows/phonon/phonon-dos.md)
- [standards/output-review-checklist.md](../standards/output-review-checklist.md)
- [standards/pass-warn-block.md](../standards/pass-warn-block.md)
- [references/source-index.md](../references/source-index.md)


## 结论强度

- Strong：本页中由 DFT/DFPT 基础理论、经典方法论文或 QE 官方文档直接支撑的判断。
- Moderate：本页中由综述、方法摘要或多个来源共同支持，但细节需要回到正文或官方文档核验的判断。
- Boundary：本页中用于限制解释范围的判断，不作为定量结论。
- Version-sensitive：涉及 QE、PHonon、Wannier90、EPW、Yambo 或其他工具字段和行为的判断，必须按当前官方文档复查。

## 12. 资料来源

- [Baroni et al., Rev. Mod. Phys. 73, 515 (2001)](https://doi.org/10.1103/RevModPhys.73.515)
- [QE INPUT_PH](https://www.quantum-espresso.org/Doc/INPUT_PH.html)
- [QE PHonon user guide](https://www.quantum-espresso.org/Doc/ph_user_guide/)
