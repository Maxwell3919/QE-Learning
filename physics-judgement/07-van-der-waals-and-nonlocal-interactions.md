# Van der Waals and Nonlocal Interactions

## 1. 这个主题解决什么判断问题？

本页帮助判断长程色散和非局域相互作用是否会改变结构、能带、吸附和界面结论。

## 2. 物理图像

半局域 GGA 对长程色散相互作用描述不足。D2/D3、TS、XDM 和 nonlocal vdW functionals 代表不同修正图像；它们可能显著改变层间距、吸附距离、表面能和低频 phonon。

## 3. DFT/QE 中的近似来源

近似来源包括 exchange-correlation functional、vdW correction model、赝势/泛函一致性、结构优化和后处理模型一致性。

## 4. 相关 QE 输入字段

| 字段 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| input_dft | pw.x | 定义交换关联或近似模型 | 把模型选择当作单纯数值收敛 |
| ecutwfc/ecutrho | pw.x | 确认基组误差不会主导判断 | 换物理模型后沿用旧 cutoff |
| K_POINTS 或 q-grid | pw.x/ph.x | 确认采样足以支撑目标性质 | 用绘图路径替代积分采样 |
| occupations/smearing/degauss | pw.x/dos.x | 控制占据和费米面附近积分 | 用过大 smearing 解释物理趋势 |
| vdw_corr | pw.x | 选择 vdW/dispersion 修正 | 未记录模型却比较结构 |
| input_dft | pw.x | 可能选择 nonlocal functional | relax 与后处理泛函不一致 |
| calculation=relax/vc-relax | pw.x | vdW 是否参与结构优化 | 只在 SCF 后处理切换模型 |

## 5. output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| functional / pseudopotential 回显 | 模型是否与记录一致 | input、UPF、source record |
| SCF / perturbation convergence | 数值解是否达到当前目标 | conv_thr、tr2_ph、mixing |
| warning / symmetry / occupation 信息 | 物理模型或边界条件可能不一致 | workflow record 和 output review |
| 层间距或吸附高度强烈变化 | vdW 模型主导结构 | relax/vc-relax review |
| phonon 低频层间模式敏感 | 力常数受结构和 vdW 影响 | phonon 与 cutoff |
| 不同 vdW 模型趋势不同 | 模型不确定性 | 来源和比较策略 |

## 6. 对 workflow 的影响

- [workflows/advanced/vdW-corrections.md](../workflows/advanced/vdW-corrections.md)
- [workflows/ground-state/relax.md](../workflows/ground-state/relax.md)
- [workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)

## 7. PASS / WARN / BLOCK 判断

| 状态 | 条件 | 允许进入的下游 |
|---|---|---|
| PASS | relax、SCF、phonon/后处理使用一致 vdW 模型，结构和低频模式已审阅 | 进入结构、界面或层间性质判断 |
| WARN | 不同 vdW 模型给出不同层间距、吸附高度或低频 phonon | 只做模型敏感性说明 |
| BLOCK | 未记录 vdW 模型，或 relax 与后处理切换模型后直接比较结果 | 停止定量结构/能量比较 |

## 8. 常见误区

- 未记录 vdW 模型时比较层间距
- relax 用一种泛函后处理换另一种泛函
- 把 vdW 修正当作普适精度提升
- 忽略低频 phonon 对结构的敏感性
- 只比较总能不看结构变化

## 9. 与前沿常识的关系

vdW 修正是模型选择。它可能改善长程相互作用，也可能改变误差抵消；需要把结构、能带和 phonon 放在同一模型下审阅。

## 10. 最低掌握深度

- 能说明 vdW 修正是模型选择，并检查 relax、SCF、phonon 和后处理是否使用同一模型。
- 能审阅层间距、吸附高度、界面距离和低频声子是否对 vdW 模型敏感。
- 能记录 `vdw_corr` 或 nonlocal functional 的来源和 output 回显。
- 能避免把 vdW correction 当作所有性质的普适精度提升。

## 11. 对应仓库页面

- [theory-minimum/dft-ks-scf.md](../theory-minimum/dft-ks-scf.md)
- [theory-minimum/force-stress-relaxation.md](../theory-minimum/force-stress-relaxation.md)
- [workflows/advanced/vdW-corrections.md](../workflows/advanced/vdW-corrections.md)
- [workflows/ground-state/relax.md](../workflows/ground-state/relax.md)
- [workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)
- [standards/output-review-checklist.md](../standards/output-review-checklist.md)
- [standards/pass-warn-block.md](../standards/pass-warn-block.md)
- [references/source-index.md](../references/source-index.md)


## 结论强度

- Strong：本页中由 DFT/DFPT 基础理论、经典方法论文或 QE 官方文档直接支撑的判断。
- Moderate：本页中由综述、方法摘要或多个来源共同支持，但细节需要回到正文或官方文档核验的判断。
- Boundary：本页中用于限制解释范围的判断，不作为定量结论。
- Version-sensitive：涉及 QE、PHonon、Wannier90、EPW、Yambo 或其他工具字段和行为的判断，必须按当前官方文档复查。

## 12. 资料来源

- [Tkatchenko, Adv. Funct. Mater. 25, 2054 (2015)](https://doi.org/10.1002/adfm.201402372)
- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
