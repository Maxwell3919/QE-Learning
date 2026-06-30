# Magnetism, Spin and SOC

## 1. 这个主题解决什么判断问题？

本页帮助判断自旋极化、磁性构型、非共线磁性和 SOC 是否足以支撑能带、总能和拓扑相关结论。

## 2. 物理图像

磁性结果依赖自旋自由度、初始磁矩、局域磁矩定义、对称性和相对论效应。SOC 会把自旋和轨道运动耦合，重元素体系、磁各向异性和拓扑能带尤其敏感。

## 3. DFT/QE 中的近似来源

近似来源包括 collinear/noncollinear 模型、SOC 赝势、初始磁性构型、smearing、k mesh、DFT+U 和 symmetry reduction。

## 4. 相关 QE 输入字段

| 字段 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| input_dft | pw.x | 定义交换关联或近似模型 | 把模型选择当作单纯数值收敛 |
| ecutwfc/ecutrho | pw.x | 确认基组误差不会主导判断 | 换物理模型后沿用旧 cutoff |
| K_POINTS 或 q-grid | pw.x/ph.x | 确认采样足以支撑目标性质 | 用绘图路径替代积分采样 |
| occupations/smearing/degauss | pw.x/dos.x | 控制占据和费米面附近积分 | 用过大 smearing 解释物理趋势 |
| nspin | pw.x | 是否启用共线自旋 | 用非磁设置解释磁性体系 |
| starting_magnetization(i) | pw.x | 给出初始自旋破缺 | 把初始值当最终磁矩 |
| noncolin/lspinorb | pw.x | 启用非共线/SOC | 无相对论赝势时解释 SOC |
| angle1/angle2 | pw.x | 非共线初始方向 | 未记录方向约定 |

## 5. output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| functional / pseudopotential 回显 | 模型是否与记录一致 | input、UPF、source record |
| SCF / perturbation convergence | 数值解是否达到当前目标 | conv_thr、tr2_ph、mixing |
| warning / symmetry / occupation 信息 | 物理模型或边界条件可能不一致 | workflow record 和 output review |
| total/absolute magnetization 不一致 | 局域补偿或多子格磁性 | 构型和投影磁矩 |
| SOC 后 symmetry 改变 | 相对论模型影响能带 | SOC input 与 pseudo |
| 不同初态收敛到不同磁矩 | 多稳态 | 多构型比较 |

## 6. 对 workflow 的影响

- [workflows/advanced/spin-polarization.md](../workflows/advanced/spin-polarization.md)
- [workflows/advanced/noncollinear-soc.md](../workflows/advanced/noncollinear-soc.md)
- [workflows/electronic/bands.md](../workflows/electronic/bands.md)

## 7. PASS / WARN / BLOCK 判断

| 状态 | 条件 | 允许进入的下游 |
|---|---|---|
| PASS | 磁性构型、最终磁矩、SOC 赝势、symmetry reduction 和 bands/DOS 解释一致 | 进入磁性或 SOC 相关下游 |
| WARN | 多个磁态能量接近，或 SOC/非共线设置改变小能隙和磁矩 | 只做构型敏感性说明 |
| BLOCK | 把初始磁矩当最终磁性，或无全相对论赝势时解释 SOC/topology | 停止磁性/SOC 结论 |

## 8. 常见误区

- 把初始磁矩当作最终磁性
- 只算一个磁性构型
- 没有 fully relativistic pseudopotential 就解释 SOC
- SOC 后仍用无 SOC bands 判断拓扑
- 不记录磁矩单位和投影定义

## 9. 与前沿常识的关系

SOC 对重元素、拓扑、磁各向异性和小带隙能带劈裂敏感。磁性结论通常需要多构型、收敛和输出磁矩证据共同支撑。

## 10. 最低掌握深度

- 能区分 SOC、非共线磁性、全相对论赝势、总磁矩、绝对磁矩和局域磁矩。
- 能从 output 中审阅初始磁矩、最终磁矩、symmetry reduction 和 SOC 回显。
- 能说明为什么磁性体系通常需要比较多个磁性构型。
- 能判断无 SOC 与有 SOC bands 不能混作同一拓扑或能带劈裂证据。

## 11. 对应仓库页面

- [theory-minimum/magnetism-soc-dftu.md](../theory-minimum/magnetism-soc-dftu.md)
- [theory-minimum/smearing-metals.md](../theory-minimum/smearing-metals.md)
- [workflows/advanced/spin-polarization.md](../workflows/advanced/spin-polarization.md)
- [workflows/advanced/noncollinear-soc.md](../workflows/advanced/noncollinear-soc.md)
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

- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
- [Quantum ESPRESSO documentation](https://www.quantum-espresso.org/documentation/)
