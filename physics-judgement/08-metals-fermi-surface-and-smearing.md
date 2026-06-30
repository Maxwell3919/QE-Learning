# Metals, Fermi Surface and Smearing

## 1. 这个主题解决什么判断问题？

本页帮助判断金属体系中费米面采样、smearing 和 DOS/EPC/phonon 结论是否可靠。

## 2. 物理图像

金属在 Fermi energy 附近有部分占据态，有限 k mesh 会让占据和能量对采样敏感。Smearing 平滑占据并改善积分，但 degauss 不是材料真实带隙或谱峰。

## 3. DFT/QE 中的近似来源

近似来源包括 finite k sampling、occupation model、electronic temperature、smearing broadening、DOS 后处理和 DFPT/EPC 的费米面采样。

## 4. 相关 QE 输入字段

| 字段 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| input_dft | pw.x | 定义交换关联或近似模型 | 把模型选择当作单纯数值收敛 |
| ecutwfc/ecutrho | pw.x | 确认基组误差不会主导判断 | 换物理模型后沿用旧 cutoff |
| K_POINTS 或 q-grid | pw.x/ph.x | 确认采样足以支撑目标性质 | 用绘图路径替代积分采样 |
| occupations/smearing/degauss | pw.x/dos.x | 控制占据和费米面附近积分 | 用过大 smearing 解释物理趋势 |
| occupations | pw.x | 占据策略是否适合金属 | 金属用 fixed occupation |
| smearing/degauss | pw.x/dos.x | 平滑占据或 DOS 展宽 | 过大 degauss 解释物理峰 |
| K_POINTS | pw.x | 费米面采样 | k mesh 太粗进入 EPC/phonon |

## 5. output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| functional / pseudopotential 回显 | 模型是否与记录一致 | input、UPF、source record |
| SCF / perturbation convergence | 数值解是否达到当前目标 | conv_thr、tr2_ph、mixing |
| warning / symmetry / occupation 信息 | 物理模型或边界条件可能不一致 | workflow record 和 output review |
| Fermi energy 随 k mesh 漂移 | 费米面采样不足 | kpoint convergence |
| DOS at Fermi level 不稳定 | mesh 或 broadening 不足 | dos.x 设置 |
| phonon softening 对 degauss 敏感 | Kohn anomaly/EPC 或数值问题 | smearing scan 与 q-grid |

## 6. 对 workflow 的影响

- [workflows/ground-state/smearing-and-occupations.md](../workflows/ground-state/smearing-and-occupations.md)
- [workflows/electronic/dos.md](../workflows/electronic/dos.md)
- [workflows/advanced/electron-phonon-overview.md](../workflows/advanced/electron-phonon-overview.md)

## 7. PASS / WARN / BLOCK 判断

| 状态 | 条件 | 允许进入的下游 |
|---|---|---|
| PASS | k mesh、smearing、Fermi energy、DOS at Fermi level 和目标 observable 已共同收敛 | 进入金属 DOS、phonon 或 EPC 下游 |
| WARN | 结果对 degauss、mesh 或 DOS broadening 敏感 | 只做费米面采样风险说明 |
| BLOCK | 金属使用 fixed occupation，或用过大 smearing 造成的伪峰/伪 gap 支撑物理结论 | 停止金属性或 EPC/phonon 解释 |

## 8. 常见误区

- 金属使用 fixed occupations
- 用 degauss 造成的 DOS 尾态解释带隙
- DOS 峰位由 broadening 造成却当作物理峰
- phonon/EPC 未审阅费米面采样
- 不同 smearing 设置下直接比较小能量差

## 9. 与前沿常识的关系

Kohn anomaly、EPC 和输运对费米面采样高度敏感。金属 workflow 的可靠性通常由 k mesh、smearing 和目标 observable 共同决定。

## 10. 最低掌握深度

- 能区分 SCF occupation smearing 与 DOS broadening。
- 能从 Fermi energy、band occupation、DOS at Fermi level 和 k mesh convergence 判断金属结果是否可用。
- 能设计 k mesh / degauss 敏感性审阅，尤其用于 phonon 和 EPC。
- 能识别 degauss 过大造成的伪 gap、伪峰或虚假金属性判断。

## 11. 对应仓库页面

- [theory-minimum/smearing-metals.md](../theory-minimum/smearing-metals.md)
- [theory-minimum/kpoints-symmetry-kpath.md](../theory-minimum/kpoints-symmetry-kpath.md)
- [workflows/ground-state/smearing-and-occupations.md](../workflows/ground-state/smearing-and-occupations.md)
- [workflows/electronic/dos.md](../workflows/electronic/dos.md)
- [workflows/advanced/electron-phonon-overview.md](../workflows/advanced/electron-phonon-overview.md)
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
- [Giustino, Rev. Mod. Phys. 89, 015003 (2017)](https://doi.org/10.1103/RevModPhys.89.015003)
