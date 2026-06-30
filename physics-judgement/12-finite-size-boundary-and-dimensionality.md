# Finite Size, Boundary and Dimensionality

## 1. 这个主题解决什么判断问题？

本页帮助判断周期边界、supercell、真空、维度和长程相互作用如何影响 QE 结论。

## 2. 物理图像

平面波周期性计算默认无限周期重复。有限 supercell 会引入镜像相互作用；低维体系需要用真空或截断/孤立模型隔离非周期方向；带电体系和极性响应尤其敏感。

## 3. DFT/QE 中的近似来源

近似来源包括 periodic boundary condition、finite supercell、vacuum thickness、charged background、k/q sampling dimensionality、ASR dimensionality 和 long-range electrostatics。

## 4. 相关 QE 输入字段

| 字段 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| input_dft | pw.x | 定义交换关联或近似模型 | 把模型选择当作单纯数值收敛 |
| ecutwfc/ecutrho | pw.x | 确认基组误差不会主导判断 | 换物理模型后沿用旧 cutoff |
| K_POINTS 或 q-grid | pw.x/ph.x | 确认采样足以支撑目标性质 | 用绘图路径替代积分采样 |
| occupations/smearing/degauss | pw.x/dos.x | 控制占据和费米面附近积分 | 用过大 smearing 解释物理趋势 |
| CELL_PARAMETERS | pw.x | 周期边界和维度模型 | 真空方向当作物理厚度 |
| K_POINTS | pw.x | 采样维度匹配 | 真空方向仍用密集 k 点 |
| assume_isolated | pw.x | 孤立或低维修正入口 | 未说明适用条件 |
| cell_dofree | pw.x vc-relax | 限制晶胞自由度 | 低维体系放开真空方向 |

## 5. output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| functional / pseudopotential 回显 | 模型是否与记录一致 | input、UPF、source record |
| SCF / perturbation convergence | 数值解是否达到当前目标 | conv_thr、tr2_ph、mixing |
| warning / symmetry / occupation 信息 | 物理模型或边界条件可能不一致 | workflow record 和 output review |
| 总能随 cell size 漂移 | finite-size artifact | supercell/vacuum convergence |
| phonon acoustic branch 异常 | ASR/维度/长程相互作用问题 | phonon debugging |
| charged system 不稳定 | 背景电荷和镜像相互作用 | 边界模型和校正 |

## 6. 对 workflow 的影响

- [workflows/ground-state/vc-relax.md](../workflows/ground-state/vc-relax.md)
- [workflows/phonon/phonon-debugging.md](../workflows/phonon/phonon-debugging.md)
- [workflows/electronic/work-function.md](../workflows/electronic/work-function.md)

## 7. PASS / WARN / BLOCK 判断

| 状态 | 条件 | 允许进入的下游 |
|---|---|---|
| PASS | supercell/vacuum、k/q sampling、维度模型和长程相互作用处理均与目标问题一致 | 进入低维、缺陷、表面或带电体系判断 |
| WARN | 结果随 cell size、vacuum、charge correction 或低维采样变化明显 | 只做有限尺寸风险说明 |
| BLOCK | 把有限 supercell 结果当稀释极限，或低维体系边界条件未记录 | 停止定量外推 |

## 8. 常见误区

- 把 supercell 结果当无限稀释极限
- 低维体系无约束 vc-relax
- 忽略真空方向 k/q 采样
- 带电体系不说明补偿和校正
- 把结构构造细节写入 QE 结论但无记录

## 9. 与前沿常识的关系

有限尺寸误差经常是模型边界问题，不能靠一次 SCF 收敛解决。低维、缺陷、表面和带电体系需要额外记录边界模型。

## 10. 最低掌握深度

- 能设计 supercell 或 vacuum 收敛检查，并说明它们影响哪些物理量。
- 能按 0D/1D/2D/3D 维度审阅 k/q sampling、ASR 和长程静电边界。
- 能识别 charged background、periodic image interaction 和低维长程相互作用风险。
- 能把 `assume_isolated`、`cell_dofree` 和真空方向约束写入 record。

## 11. 对应仓库页面

- [theory-minimum/kpoints-symmetry-kpath.md](../theory-minimum/kpoints-symmetry-kpath.md)
- [theory-minimum/force-stress-relaxation.md](../theory-minimum/force-stress-relaxation.md)
- [workflows/ground-state/vc-relax.md](../workflows/ground-state/vc-relax.md)
- [workflows/phonon/phonon-debugging.md](../workflows/phonon/phonon-debugging.md)
- [workflows/electronic/work-function.md](../workflows/electronic/work-function.md)
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
