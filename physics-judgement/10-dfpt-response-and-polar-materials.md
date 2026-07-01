# DFPT Response and Polar Materials

## 1. 这个主题解决什么判断问题？

本页帮助判断介电张量、Born effective charge、LO-TO splitting 和 polar phonon 结果是否有效。

## 物理图像

DFPT response 处理的是系统对小扰动的线性响应。对 polar materials，原子位移不仅改变局域键长和短程 force constants，还会改变宏观极化；极化产生的长程电场再反馈到动力学矩阵。这个长程项在 Gamma 附近具有方向依赖，因此纵向光学模式和横向光学模式可能出现 LO-TO splitting。

Dielectric tensor 描述宏观电场下极化如何响应，Born effective charge 描述原子位移如何改变极化。它们共同进入 polar phonon 的 non-analytic correction。由于这些量是 response quantities，它们比普通 total energy 更依赖 band gap、occupation、k mesh、functional、pseudopotential、structure 和 perturbation convergence。

QE 中 `epsil=.true.` 请求的是特定条件下的电场响应分支。输出张量并不自动证明 polar correction 可用于所有后处理：需要检查体系是否适合绝缘体电场响应、tensor symmetry 是否合理、warnings 是否解释、`dynmat.x`/`matdyn.x` 是否读取同一数据链，以及 LO-TO 方向和 ASR/zasr 设置是否记录。

## 3. DFT/QE 中的近似来源

近似来源包括 linear response、metal/insulator condition、electric-field perturbation、SCF/k mesh 收敛、`epsil` 分支、ASR 和非解析项后处理。

## 4. 相关 QE 输入字段

| 字段 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| input_dft | pw.x | 定义交换关联或近似模型 | 把模型选择当作单纯数值收敛 |
| ecutwfc/ecutrho | pw.x | 确认基组误差不会主导判断 | 换物理模型后沿用旧 cutoff |
| K_POINTS 或 q-grid | pw.x/ph.x | 确认采样足以支撑目标性质 | 用绘图路径替代积分采样 |
| occupations/smearing/degauss | pw.x/dos.x | 控制占据和费米面附近积分 | 用过大 smearing 解释物理趋势 |
| epsil | ph.x | 是否计算电场响应 | 在金属中解释 Born charge |
| trans | ph.x | 是否计算原子位移扰动 | 关闭后仍期待 phonon modes |
| lraman | ph.x | Raman 分支入口 | 未核对适用条件 |
| `asr` | `dynmat.x/matdyn.x` | 声学规则与非解析项配合 | 不记录 non-analytic 设置 |
| `zasr` | `q2r.x` | IFC 转换阶段的 acoustic sum rule 设置 | 混淆 `q2r.x` 与 `matdyn.x/dynmat.x` 的参数归属 |

## 5. output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| functional / pseudopotential 回显 | 模型是否与记录一致 | input、UPF、source record |
| SCF / perturbation convergence | 数值解是否达到当前目标 | conv_thr、tr2_ph、mixing |
| warning / symmetry / occupation 信息 | 物理模型或边界条件可能不一致 | workflow record 和 output review |
| dielectric tensor 缺失 | `epsil` 或物理条件不满足 | ph.x input/output |
| Born charge 数值异常 | 极性响应、收敛或金属边界问题 | SCF、k mesh、occupation |
| LO-TO 分裂未体现 | non-analytic correction 未使用或方向未记录 | dynmat/matdyn 设置 |

## 6. 对 workflow 的影响

- [workflows/phonon/dielectric-born-effective-charge.md](../workflows/phonon/dielectric-born-effective-charge.md)
- [workflows/phonon/gamma-phonon.md](../workflows/phonon/gamma-phonon.md)
- [workflows/phonon/ir-raman.md](../workflows/phonon/ir-raman.md)

## 7. PASS / WARN / BLOCK 判断

| 状态 | 条件 | 允许进入的下游 |
|---|---|---|
| PASS | 绝缘体条件、`epsil`、dielectric tensor、Born charge、LO-TO/non-analytic 设置和收敛均一致 | 进入 polar phonon、IR/Raman 或介电响应解释 |
| WARN | 小带隙、部分占据或 Born charge 对设置敏感 | 只做响应风险说明 |
| BLOCK | 在金属中解释 Born charge/dielectric 分支，或未记录 non-analytic correction 就解释 LO-TO | 停止极性响应结论 |

## 8. 常见误区

- 在金属中解释 Born charge
- 混淆 Gamma direct phonon 和 q-grid interpolation
- 不记录 non-analytic correction
- 把 Born charge 当静态价态
- 忽略 smearing 对极性响应的破坏

## 9. 与前沿常识的关系

polar phonon 需要 Born charge、dielectric tensor 和 non-analytic correction 一起审阅。金属、小带隙和部分占据体系需要保守处理。

## 10. 最低掌握深度

- 能说明 `epsil`、dielectric tensor、Born effective charge 和 LO-TO splitting 的适用条件。
- 能识别金属、小带隙或部分占据体系中电场响应解释的 WARN/BLOCK 风险。
- 能检查 non-analytic correction、ASR、Gamma limit 和 q-grid interpolation 是否被正确记录。
- 能避免把 Born effective charge 当作静态价态。

## 11. 对应仓库页面

- [theory-minimum/dielectric-born-charge.md](../theory-minimum/dielectric-born-charge.md)
- [theory-minimum/dfpt-phonons.md](../theory-minimum/dfpt-phonons.md)
- [workflows/phonon/dielectric-born-effective-charge.md](../workflows/phonon/dielectric-born-effective-charge.md)
- [workflows/phonon/gamma-phonon.md](../workflows/phonon/gamma-phonon.md)
- [workflows/phonon/ir-raman.md](../workflows/phonon/ir-raman.md)
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
