# Wannier 化与有效模型思维

## 1. 这个主题解决什么判断问题？

本页帮助判断 Wannier 化是否可靠，以及何时可以把 DFT 结果压缩成有效模型。

## 物理图像

Bloch states 是周期体系在 k 空间中的本征态，天然适合描述能带色散；Wannier functions 则把同一子空间重新表示为实空间局域轨道。这个转换不是改变 DFT 结果本身，而是选择一个 gauge 和一个目标子空间，把 `E_n(k)` 与矩阵元压缩成可插值的有效 Hamiltonian。

当目标 bands 是 isolated manifold 时，Wannier 化主要考验 projection 和局域化质量；当 bands 与其他能带纠缠时，还需要选择 outer window、frozen window 和 disentanglement 策略。窗口选择过窄会丢失目标态，过宽可能引入无关态；projection 看似合理也可能无法复现目标能区的 ab initio bands。

Wannier validation 的核心证据是 interpolated bands 在目标窗口内回对 QE bands，并且 projection、window、spread、spin/SOC 设置和接口文件都可复查。Small spread 是有用线索，但不是充分条件；用于 Berry curvature、transport、topology 或 EPC 时，还需要对目标 observable 的额外质量检查。

## 3. DFT/QE 中的近似来源

近似来源包括目标能窗选择、投影轨道、纠缠能带、k mesh、spin/SOC 约定、QE-to-Wannier interface 和下游 Berry/transport/EPC 插值。

## 4. 相关 QE 输入字段

| 字段 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| input_dft | pw.x | 定义交换关联或近似模型 | 把模型选择当作单纯数值收敛 |
| ecutwfc/ecutrho | pw.x | 确认基组误差不会主导判断 | 换物理模型后沿用旧 cutoff |
| K_POINTS 或 q-grid | pw.x/ph.x | 确认采样足以支撑目标性质 | 用绘图路径替代积分采样 |
| occupations/smearing/degauss | pw.x/dos.x | 控制占据和费米面附近积分 | 用过大 smearing 解释物理趋势 |
| nbnd | pw.x | 包含目标能窗的带数 | 带数不足进入 Wannier |
| projections | wannier90 | 初始局域轨道图像 | 投影与 PDOS/物理不一致 |
| dis_win/dis_froz | wannier90 | 纠缠窗口 | 窗口未记录导致不可复现 |
| pw2wannier90.x | interface | 生成 mmn/amn/eig | prefix/outdir 或 k mesh 不匹配 |

## 5. output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| functional / pseudopotential 回显 | 模型是否与记录一致 | input、UPF、source record |
| SCF / perturbation convergence | 数值解是否达到当前目标 | conv_thr、tr2_ph、mixing |
| warning / symmetry / occupation 信息 | 物理模型或边界条件可能不一致 | workflow record 和 output review |
| Wannier bands 偏离 QE bands | 投影、窗口或 k mesh 问题 | bands 对照 |
| spread 异常大 | 局域化质量差或能窗不合适 | projection/window |
| 接口文件缺失 | QE/Wannier 数据链断裂 | pw2wannier90 output |

## 6. 对 workflow 的影响

- [workflows/advanced/wannier90-overview.md](../workflows/advanced/wannier90-overview.md)
- [workflows/electronic/bands.md](../workflows/electronic/bands.md)
- [workflows/electronic/pdos.md](../workflows/electronic/pdos.md)

## 7. PASS / WARN / BLOCK 判断

| 状态 | 条件 | 允许进入的下游 |
|---|---|---|
| PASS | Wannier bands 在目标能窗复现 QE bands，projections/window/spread/interface 文件均记录 | 进入 Berry、transport、EPC 或有效模型分析 |
| WARN | spread、窗口或投影选择存在风险，但目标趋势仍可对照 | 只做限定插值或模型说明 |
| BLOCK | Wannier bands 不复现 QE bands，或接口文件/窗口/projection 不可追踪 | 停止下游插值结论 |

## 8. 常见误区

- 不和 QE bands 对照
- 窗口和 projections 不记录
- spread 小就认为物理正确
- SOC/spin 设置前后不一致
- 用 Wannier 插值修补上游 SCF/bands 问题

## 9. 与前沿常识的关系

Wannier 是有效模型入口，不是自动真值。Berry curvature、transport、topological analysis 和 EPC 都依赖 Wannier 基础质量。

## 10. 最低掌握深度

- 能用 QE bands 对照 Wannier-interpolated bands，判断目标能窗内是否可用。
- 能记录 projections、frozen/disentanglement window、spread 和接口文件完整性。
- 能说明 Wannier 化是有效模型构造，不是自动提高 DFT 精度。
- 能判断 Berry、transport 和 EPC 进入前需要哪些 Wannier 质量证据。

## 11. 对应仓库页面

- [theory-minimum/kpoints-symmetry-kpath.md](../theory-minimum/kpoints-symmetry-kpath.md)
- [theory-minimum/magnetism-soc-dftu.md](../theory-minimum/magnetism-soc-dftu.md)
- [workflows/advanced/wannier90-overview.md](../workflows/advanced/wannier90-overview.md)
- [workflows/electronic/bands.md](../workflows/electronic/bands.md)
- [workflows/electronic/pdos.md](../workflows/electronic/pdos.md)
- [standards/output-review-checklist.md](../standards/output-review-checklist.md)
- [standards/pass-warn-block.md](../standards/pass-warn-block.md)
- [references/source-index.md](../references/source-index.md)


## 结论强度

- Strong：本页中由 DFT/DFPT 基础理论、经典方法论文或 QE 官方文档直接支撑的判断。
- Moderate：本页中由综述、方法摘要或多个来源共同支持，但细节需要回到正文或官方文档核验的判断。
- Boundary：本页中用于限制解释范围的判断，不作为定量结论。
- Version-sensitive：涉及 QE、PHonon、Wannier90、EPW、Yambo 或其他工具字段和行为的判断，必须按当前官方文档复查。

## 12. 资料来源

- [Wannier90 documentation](https://wannier.org/)
- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
