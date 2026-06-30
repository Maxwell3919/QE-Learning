# Surfaces, Interfaces and Electrostatics

## 1. 这个主题解决什么判断问题？

本页帮助判断 slab、界面、真空、电势平台和 work function 相关结论是否可信。

## 2. 物理图像

周期边界条件会让 slab 与其周期镜像相互作用。work function 需要 Fermi level 与真空电势平台共同定义；表面偶极、真空厚度、面方向和电势平均方式都会改变结果。

对金属表面，Fermi level 通常可作为电子化学势参考。对半导体或绝缘体表面，必须说明 Fermi level 约定、掺杂/电荷态条件，或改为报告 VBM/CBM 相对真空能级的 band alignment；不能把金属 work-function 公式无条件搬到所有 slab。

## 3. DFT/QE 中的近似来源

近似来源包括 finite-size boundary、vacuum artifact、dipole correction、electrostatic potential averaging、charged slab/image interaction 和 ESM/isolated-boundary 模型。

## 4. 相关 QE 输入字段

| 字段 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| input_dft | pw.x | 定义交换关联或近似模型 | 把模型选择当作单纯数值收敛 |
| ecutwfc/ecutrho | pw.x | 确认基组误差不会主导判断 | 换物理模型后沿用旧 cutoff |
| K_POINTS 或 q-grid | pw.x/ph.x | 确认采样足以支撑目标性质 | 用绘图路径替代积分采样 |
| occupations/smearing/degauss | pw.x/dos.x | 控制占据和费米面附近积分 | 用过大 smearing 解释物理趋势 |
| assume_isolated | pw.x | 孤立边界模型入口 | 未说明边界模型 |
| tefield/dipfield | pw.x | 外场/偶极修正入口 | 未记录方向和 slab 几何 |
| plot_num | pp.x | 电势或电荷密度输出 | 把 3D 图像当定量平台 |
| average.x direction | average.x | 平面平均方向 | 平均方向与表面法向不一致 |

## 5. output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| functional / pseudopotential 回显 | 模型是否与记录一致 | input、UPF、source record |
| SCF / perturbation convergence | 数值解是否达到当前目标 | conv_thr、tr2_ph、mixing |
| warning / symmetry / occupation 信息 | 物理模型或边界条件可能不一致 | workflow record 和 output review |
| 真空平台不平坦 | 真空不足、偶极或带电伪影 | slab 设置和 potential |
| work function 随真空厚度漂移 | finite-size/boundary 未收敛 | vacuum convergence |
| 电势零点混乱 | 后处理参考不一致 | pp.x/average.x 与 Fermi energy |

## 6. 对 workflow 的影响

- [workflows/electronic/electrostatic-potential.md](../workflows/electronic/electrostatic-potential.md)
- [workflows/electronic/work-function.md](../workflows/electronic/work-function.md)
- [workflows/electronic/charge-density.md](../workflows/electronic/charge-density.md)

## 7. PASS / WARN / BLOCK 判断

| 状态 | 条件 | 允许进入的下游 |
|---|---|---|
| PASS | 真空平台、Fermi/VBM/CBM 参考、偶极方向、真空厚度和电势平均方向均可追踪 | 进入 work function 或 band alignment 记录 |
| WARN | 电势平台或 work function 对真空/偶极修正敏感 | 只做边界条件敏感性说明 |
| BLOCK | 无稳定真空平台、Fermi-level 约定不清，或带电/极性 slab 边界未说明 | 停止定量 work-function 解释 |

## 8. 常见误区

- 把 work function 写成一行公式后不看电势平台
- 真空厚度未收敛
- 偶极修正方向未记录
- 带电 slab 不说明补偿背景
- 结构构建问题混入 QE workflow 结论

## 9. 与前沿常识的关系

work function 的公式需要电势平台、Fermi level、表面方向、真空收敛和偶极修正共同支撑。本页不展开结构构建，结构操作仍属独立 Structure-Learning。

## 10. 最低掌握深度

- 能检查 vacuum level 是否有稳定平台，平均方向是否沿表面法向。
- 能说明金属 work function、半导体/绝缘体 band alignment 和 Fermi-level 约定的区别。
- 能审阅真空厚度、偶极修正方向、charged slab 边界和电势零点。
- 能判断 work function 结果何时只能作为 WARN 记录。

## 11. 对应仓库页面

- [theory-minimum/kpoints-symmetry-kpath.md](../theory-minimum/kpoints-symmetry-kpath.md)
- [theory-minimum/kpoints-symmetry-kpath.md](../theory-minimum/kpoints-symmetry-kpath.md)
- [workflows/electronic/electrostatic-potential.md](../workflows/electronic/electrostatic-potential.md)
- [workflows/electronic/work-function.md](../workflows/electronic/work-function.md)
- [workflows/electronic/charge-density.md](../workflows/electronic/charge-density.md)
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
- [QE INPUT_PP](https://www.quantum-espresso.org/Doc/INPUT_PP.html)
- [QE PostProc documentation](https://www.quantum-espresso.org/Doc/pp_user_guide/)
