# Electron-Phonon Coupling Physical Judgement

## 1. 这个主题解决什么判断问题？

本页帮助判断 EPC、phonon linewidth、Eliashberg function、Tc 或输运相关结果是否可信。

## 物理图像

Electron-phonon coupling 描述电子态在原子集体振动下如何被散射。一个声子模式改变离子势和电子 Hamiltonian，电子从一个 `k` 态散射到另一个 `k+q` 态；耦合强度由 electron-phonon matrix elements、phonon frequencies、电子 occupation 和 Fermi surface 共同决定。

这使 EPC 成为多级数据链问题。Ground-state SCF 提供电子密度和势；NSCF 或 Wannier 前置提供电子态；`ph.x` 提供 phonon perturbations；Wannier/EPW 把电子、声子和 EPC matrix elements 插值到 dense k/q meshes；最后才得到 linewidth、Eliashberg function、lambda、mobility 或 superconducting Tc 等 observable。任一环节的误差都会传到最终数值。

EPC 对金属 Fermi surface、smearing/broadening、q-grid、Wannier band validation、polar correction、functional、pseudopotential 和 SOC/U 模型都敏感。因此一个 `lambda` 或 `Tc` 数字本身不是结论；它必须附带上游 phonon 质量、Wannier 质量、dense mesh convergence 和目标 observable 的边界说明。

## 3. DFT/QE 中的近似来源

近似来源包括 functional、pseudopotential、SCF/NSCF k mesh、phonon q-grid、smearing、Wannier interpolation、EPW coarse/fine mesh 和温度/近似输运模型。

## 4. 相关 QE 输入字段

| 字段 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| input_dft | pw.x | 定义交换关联或近似模型 | 把模型选择当作单纯数值收敛 |
| ecutwfc/ecutrho | pw.x | 确认基组误差不会主导判断 | 换物理模型后沿用旧 cutoff |
| K_POINTS 或 q-grid | pw.x/ph.x | 确认采样足以支撑目标性质 | 用绘图路径替代积分采样 |
| occupations/smearing/degauss | pw.x/dos.x | 控制占据和费米面附近积分 | 用过大 smearing 解释物理趋势 |
| ph.x q-grid | ph.x | phonon 与 EPC 粗网格 | phonon 未收敛进入 EPC |
| degauss/smearing | pw.x/EPW | 费米面积分 | 用单一 smearing 得出定量结论 |
| projections/windows | Wannier90/EPW | 插值质量 | Wannier bands 未对照 QE bands |
| fine k/q mesh | EPW | EPC 积分 | 细网格不足却报告 Tc |

## 5. output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| functional / pseudopotential 回显 | 模型是否与记录一致 | input、UPF、source record |
| SCF / perturbation convergence | 数值解是否达到当前目标 | conv_thr、tr2_ph、mixing |
| warning / symmetry / occupation 信息 | 物理模型或边界条件可能不一致 | workflow record 和 output review |
| lambda 或 linewidth 对 mesh 敏感 | 费米面或 q-grid 采样不足 | k/q convergence |
| Eliashberg function 峰不稳定 | phonon/Wannier/interpolation 问题 | phonon DOS 与 EPC |
| Tc 随 smearing 大变 | 积分和模型不确定性 | smearing scan |

## 6. 对 workflow 的影响

- [workflows/advanced/electron-phonon-overview.md](../workflows/advanced/electron-phonon-overview.md)
- [workflows/advanced/wannier90-overview.md](../workflows/advanced/wannier90-overview.md)
- [workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)

## 7. PASS / WARN / BLOCK 判断

| 状态 | 条件 | 允许进入的下游 |
|---|---|---|
| PASS | SCF、phonon、Wannier、coarse/fine k/q mesh、smearing 和 EPC 输出均通过审阅 | 进入 EPC、linewidth、Eliashberg 或限定 Tc 讨论 |
| WARN | lambda、linewidth 或 Eliashberg function 对 mesh/smearing/functional 敏感 | 只做敏感性或探索记录 |
| BLOCK | phonon 或 Wannier 未通过审阅却解释 EPC/Tc/输运 | 停止 EPC 定量结论 |

## 8. 常见误区

- phonon 未通过就解释 EPC
- Wannier bands 不对照
- 只报 Tc 不报 mesh/smearing
- 把单一泛函 EPC 当最终物性
- 忽略 polar/low-dimensional long-range 处理

## 9. 与前沿常识的关系

EPC 对 k/q mesh、Fermi surface、functional、phonon 和 Wannier 质量敏感。EPW 是高级工具，本页只定义进入 EPW 前的物理审阅边界。

## 10. 最低掌握深度

- 能列出 EPC 前置门槛：SCF、dense electron sampling、phonon q-grid、Wannier bands 对照和 EPW mesh。
- 能从 lambda、linewidth、Eliashberg function 或 Tc 对 k/q mesh、smearing、functional 的敏感性识别风险。
- 能说明 EPC 结果比普通 bands/DOS 更依赖整条数据链。
- 能区分 exploratory EPC 记录和可用于定量结论的 PASS 证据。

## 11. 对应仓库页面

- [theory-minimum/dfpt-phonons.md](../theory-minimum/dfpt-phonons.md)
- [theory-minimum/smearing-metals.md](../theory-minimum/smearing-metals.md)
- [workflows/advanced/electron-phonon-overview.md](../workflows/advanced/electron-phonon-overview.md)
- [workflows/advanced/wannier90-overview.md](../workflows/advanced/wannier90-overview.md)
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

- [Giustino, Rev. Mod. Phys. 89, 015003 (2017)](https://doi.org/10.1103/RevModPhys.89.015003)
- [EPW documentation](https://docs.epw-code.org/)
- [Baroni et al., Rev. Mod. Phys. 73, 515 (2001)](https://doi.org/10.1103/RevModPhys.73.515)
