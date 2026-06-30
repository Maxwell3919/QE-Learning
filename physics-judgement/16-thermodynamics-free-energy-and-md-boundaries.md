# Thermodynamics, Free Energy and MD Boundaries

## 1. 这个主题解决什么判断问题？

本页帮助判断 total energy、free energy、phonon free energy、AIMD 和 NEB 结果能说明哪些稳定性问题。

## 2. 物理图像

0 K DFT total energy 是电子基态能量近似，不自动等于实验温度下的热力学稳定性。自由能还可能包含 zero-point energy、harmonic phonon contribution、configurational entropy、electronic entropy 和 finite-temperature sampling。

## 3. DFT/QE 中的近似来源

近似来源包括 harmonic approximation、finite-temperature occupation、AIMD sampling、thermostat、cell size、time step、metastability 和 reaction path model。

## 4. 相关 QE 输入字段

| 字段 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| input_dft | pw.x | 定义交换关联或近似模型 | 把模型选择当作单纯数值收敛 |
| ecutwfc/ecutrho | pw.x | 确认基组误差不会主导判断 | 换物理模型后沿用旧 cutoff |
| K_POINTS 或 q-grid | pw.x/ph.x | 确认采样足以支撑目标性质 | 用绘图路径替代积分采样 |
| occupations/smearing/degauss | pw.x/dos.x | 控制占据和费米面附近积分 | 用过大 smearing 解释物理趋势 |
| calculation=md/vc-md | pw.x | AIMD 动力学入口 | 采样不足得出热力学结论 |
| ion_temperature/tempw | pw.x/cp.x | 温度控制 | 温控参数未记录 |
| dt/nstep | pw.x/cp.x | 时间步和采样长度 | 短轨迹当统计平均 |
| neb.x path fields | neb.x | 反应路径能垒 | NEB 当自由能路径 |

## 5. output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| functional / pseudopotential 回显 | 模型是否与记录一致 | input、UPF、source record |
| SCF / perturbation convergence | 数值解是否达到当前目标 | conv_thr、tr2_ph、mixing |
| warning / symmetry / occupation 信息 | 物理模型或边界条件可能不一致 | workflow record 和 output review |
| 能量差很小 | 模型/数值/熵贡献可能同量级 | uncertainty table |
| MD 温度漂移 | 积分或 thermostat 问题 | MD output |
| 未经复查的 phonon 虚频 | harmonic free energy 解释需要先暂停 | phonon stability、ASR、收敛和模式分析 |

## 6. 对 workflow 的影响

- [workflows/advanced/molecular-dynamics.md](../workflows/advanced/molecular-dynamics.md)
- [workflows/advanced/neb-reaction-path.md](../workflows/advanced/neb-reaction-path.md)
- [workflows/phonon/phonon-dos.md](../workflows/phonon/phonon-dos.md)

## 7. PASS / WARN / BLOCK 判断

| 状态 | 条件 | 允许进入的下游 |
|---|---|---|
| PASS | 0 K 能量、ZPE/harmonic free energy、AIMD 采样或 NEB barrier 的问题边界已分清 | 进入对应稳定性或路径讨论 |
| WARN | 能量差与数值/模型误差同量级，或 MD/phonon 采样不足 | 只做限定热力学说明 |
| BLOCK | 存在未经复查的虚频、采样不足或把 NEB 能垒当 free-energy barrier | 停止有限温度稳定性结论 |

## 8. 常见误区

- 用 0 K 总能直接代表所有温度稳定性
- 短 AIMD 轨迹给出定量热力学
- NEB 能垒当 free-energy barrier
- 未区分数值噪声、ASR/收敛问题和真实软模时直接做 harmonic free energy
- 不记录 thermostat 和采样长度

## 9. 与前沿常识的关系

有限温度稳定性需要明确 free-energy 模型。AIMD、harmonic phonon 和 NEB 回答的问题不同，不能互相替代。

## 10. 最低掌握深度

- 能区分 0 K total energy、zero-point energy、harmonic phonon free energy、AIMD sampling 和 NEB barrier。
- 能判断何时 harmonic approximation 被虚频、软模或 anharmonicity 破坏。
- 能审阅 MD 的 time step、trajectory length、thermostat、temperature drift 和采样不足。
- 能说明 0 K DFT 能量差不能直接代表所有实验温度下稳定性。

## 11. 对应仓库页面

- [theory-minimum/force-stress-relaxation.md](../theory-minimum/force-stress-relaxation.md)
- [theory-minimum/dfpt-phonons.md](../theory-minimum/dfpt-phonons.md)
- [workflows/advanced/molecular-dynamics.md](../workflows/advanced/molecular-dynamics.md)
- [workflows/advanced/neb-reaction-path.md](../workflows/advanced/neb-reaction-path.md)
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

- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
- [Baroni et al., Rev. Mod. Phys. 73, 515 (2001)](https://doi.org/10.1103/RevModPhys.73.515)
