# Spectroscopy, GW/BSE and Excited-State Boundaries

## 1. 这个主题解决什么判断问题？

本页帮助判断 ground-state DFT eigenvalues、光谱、GW/BSE、XAS、EELS 等激发态结论的边界。

## 2. 物理图像

Kohn-Sham eigenvalues 是辅助体系的本征值。光学吸收、准粒子能量、激子、芯能级吸收和 EELS 涉及激发态或响应函数，常需要 TDDFT、GW、BSE、XSpectra、TurboEELS 或 YAMBO/GWL 等专门方法。

## 3. DFT/QE 中的近似来源

近似来源包括 ground-state functional、unoccupied bands、screening、local-field effects、core-hole approximation、many-body correction、broadening 和外部接口。

## 4. 相关 QE 输入字段

| 字段 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| input_dft | pw.x | 定义交换关联或近似模型 | 把模型选择当作单纯数值收敛 |
| ecutwfc/ecutrho | pw.x | 确认基组误差不会主导判断 | 换物理模型后沿用旧 cutoff |
| K_POINTS 或 q-grid | pw.x/ph.x | 确认采样足以支撑目标性质 | 用绘图路径替代积分采样 |
| occupations/smearing/degauss | pw.x/dos.x | 控制占据和费米面附近积分 | 用过大 smearing 解释物理趋势 |
| nbnd | pw.x/response tools | 未占据态是否足够 | 带数不足解释光谱 |
| broadening/energy grid | spectroscopy tools | 谱线展宽与分辨率 | 把展宽峰当精确跃迁 |
| SOC/noncolin | pw.x | 重元素或选择定则 | 无 SOC 解释 SOC 敏感谱 |

## 5. output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| functional / pseudopotential 回显 | 模型是否与记录一致 | input、UPF、source record |
| SCF / perturbation convergence | 数值解是否达到当前目标 | conv_thr、tr2_ph、mixing |
| warning / symmetry / occupation 信息 | 物理模型或边界条件可能不一致 | workflow record 和 output review |
| DFT bands 与实验光谱差异大 | ground-state eigenvalue 边界 | GW/BSE/TDDFT |
| 光谱峰对 nbnd 敏感 | 未占据空间不足 | band convergence |
| 吸收边对 broadening 敏感 | 后处理展宽主导 | spectroscopy settings |

## 6. 对 workflow 的影响

- [workflows/electronic/bands.md](../workflows/electronic/bands.md)
- [workflows/advanced/hybrid-functional-overview.md](../workflows/advanced/hybrid-functional-overview.md)
- [learn/09-feature-expansion-map.md](../learn/09-feature-expansion-map.md)

## 7. PASS / WARN / BLOCK 判断

| 状态 | 条件 | 允许进入的下游 |
|---|---|---|
| PASS | DFT 起点、未占据带、展宽、SOC、screening 和所用激发态工具边界均记录 | 进入光谱/GW/BSE 限定解释 |
| WARN | 峰位对 nbnd、broadening、起点泛函或 SOC 敏感 | 只做谱形趋势说明 |
| BLOCK | 用 ground-state KS eigenvalues 直接解释所有实验光谱或准粒子能量 | 停止激发态定量解释 |

## 8. 常见误区

- 把 Kohn-Sham eigenvalues 当准粒子能量
- 用 ground-state DFT 直接解释所有光谱
- 不收敛 unoccupied bands
- 不记录 broadening
- 忽略 SOC 和 core-level 特殊处理

## 9. 与前沿常识的关系

GW 修正 quasiparticle gap，BSE 处理 exciton，TDDFT/XSpectra/TurboEELS 有各自适用边界。普通 DFT 可提供起点，不能自动给出所有激发态结论。

## 10. 最低掌握深度

- 能区分 DFT 起点、quasiparticle correction、optical absorption、exciton、XAS 和 EELS 的问题边界。
- 能检查 unoccupied bands、broadening、energy grid、SOC 和 screening 是否足以支持光谱解释。
- 能说明 ground-state DFT eigenvalues 不能自动替代 GW/BSE/TDDFT/XSpectra 等激发态方法。
- 能把工具名映射到具体物理问题，而不是堆叠软件名。

## 11. 对应仓库页面

- [theory-minimum/dft-ks-scf.md](../theory-minimum/dft-ks-scf.md)
- [theory-minimum/kpoints-symmetry-kpath.md](../theory-minimum/kpoints-symmetry-kpath.md)
- [workflows/electronic/bands.md](../workflows/electronic/bands.md)
- [workflows/advanced/hybrid-functional-overview.md](../workflows/advanced/hybrid-functional-overview.md)
- [learn/09-feature-expansion-map.md](../learn/09-feature-expansion-map.md)
- [standards/output-review-checklist.md](../standards/output-review-checklist.md)
- [standards/pass-warn-block.md](../standards/pass-warn-block.md)
- [references/source-index.md](../references/source-index.md)


## 结论强度

- Strong：本页中由 DFT/DFPT 基础理论、经典方法论文或 QE 官方文档直接支撑的判断。
- Moderate：本页中由综述、方法摘要或多个来源共同支持，但细节需要回到正文或官方文档核验的判断。
- Boundary：本页中用于限制解释范围的判断，不作为定量结论。
- Version-sensitive：涉及 QE、PHonon、Wannier90、EPW、Yambo 或其他工具字段和行为的判断，必须按当前官方文档复查。

## 12. 资料来源

- [Onida, Reining and Rubio, Rev. Mod. Phys. 74, 601 (2002)](https://doi.org/10.1103/RevModPhys.74.601)
- [Yambo documentation](https://www.yambo-code.eu/wiki/index.php/Main_Page)
- [Quantum ESPRESSO documentation](https://www.quantum-espresso.org/documentation/)
