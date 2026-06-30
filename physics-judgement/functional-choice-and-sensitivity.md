# Functional Choice and Sensitivity

## 1. 核心判断结论

- **Strong:** functional choice 是物理模型选择，不是普通数值参数。
- **Strong:** 更高 rung、hybrid、meta-GGA 或 vdW-corrected functional 不自动保证所有性质更准。
- **Moderate:** functional 会同时影响结构、磁性、band gap、phonon、EPC 和表面/界面性质。
- **Boundary:** 比较不同 functional 时，应把几何结构、pseudo、cutoff、k/q mesh 和后处理流程作为新模型重新审阅。

## 2. 这个问题为什么会出现在 DFT/QE 中？

XC functional 近似是 DFT 误差中无法通过简单收敛测试消除的部分。QE 中 `input_dft` 选择会改变能量面和响应性质；如果结构用一种 functional 优化、后处理换另一种 functional，需要说明这是模型切换而不是普通重算。

## 3. 相关 QE input 字段

| 字段/设置 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `input_dft` | `pw.x` | 选择 XC functional | 未记录 functional 或与 pseudo 不一致 |
| `vdw_corr` | `pw.x` | 添加 dispersion / nonlocal correction | 当作普适精度提升 |
| hybrid 相关字段 | `pw.x` | exact exchange / screened hybrid 设置 | 只为 gap 换模型但不重审结构 |
| `ecutfock` | `pw.x` | Fock exchange 相关 cutoff | 把 hybrid 数值设置沿用普通 GGA |

## 4. output review 迹象

| output 迹象 | 可能原因 | 回查动作 |
|---|---|---|
| lattice、magnetization、gap、phonon 对 functional 变化敏感 | 模型不确定性主导 | 写 sensitivity statement |
| pseudo XC 与 `input_dft` 不一致 | 模型来源混用 | 回查 pseudo 元信息 |
| hybrid/meta-GGA 后 warning 或性能问题 | 版本和实现敏感 | 回查官方文档 |

## 5. PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | functional 与目标 observable 匹配，pseudo/结构/后处理一致记录 | 允许进入下游 |
| WARN | 只做趋势或对比，但不同 functional 可能改变结论 | 允许带模型边界解释 |
| BLOCK | 用 functional 切换掩盖未收敛或未记录模型来源 | 停止定量比较 |

## 6. 常见误区

- 写“hybrid 一定更准”
- 把 PBE/PBEsol/meta-GGA 差异当数值误差
- relax 用一个模型、phonon 用另一个模型却不说明
- 不同 functional 总能直接横比
- 忽略 pseudo-functional consistency

## 7. 应回写到哪些 workflow 页面？

- [workflows/ground-state/relax.md](../workflows/ground-state/relax.md)
- [workflows/electronic/bands.md](../workflows/electronic/bands.md)
- [workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)
- [workflows/advanced/hybrid-functional-overview.md](../workflows/advanced/hybrid-functional-overview.md)

## 8. 应回写到哪些 theory-minimum 页面？

- [theory-minimum/dft-ks-scf.md](../theory-minimum/dft-ks-scf.md)
- [theory-minimum/pseudopotentials.md](../theory-minimum/pseudopotentials.md)


## 结论强度

- Strong：本页中由经典理论、方法论文或官方文档直接支持的判断。
- Moderate：本页中由摘要、综述或多个方法来源共同支持，但细节需要回到正文或官方文档核验的判断。
- Boundary：本页中用于限制解释范围的判断，不作为定量结论。
- Version-sensitive：涉及 QE、PHonon、Wannier90、EPW、Yambo 或其他工具字段和行为的判断，必须按当前官方文档复查。

## 结论来源

| 结论 | 强度 | 支撑文献/文档 | 是否需要全文核验 |
|---|---|---|---|
| 本页核心判断需要写入 output review，而不是只作为理论背景 | Strong | 官方文档 / 方法论文 | 否 |
| 具体字段、默认值和版本行为必须回查当前官方文档 | Version-sensitive | QE INPUT_* / 工具官方文档 | 是 |

## 9. 资料来源

- Hohenberg and Kohn, *Inhomogeneous Electron Gas*：ground-state DFT 边界。
- Kohn and Sham, *Self-Consistent Equations Including Exchange and Correlation Effects*：KS 辅助体系。
- Quantum ESPRESSO official documentation and `INPUT_*` references：QE 字段和程序行为核验。
- Giannozzi et al., Quantum ESPRESSO code papers：QE 方法和模块边界。
- [references/canonical-literature.md](../references/canonical-literature.md)：本仓库 canonical literature 分级。
- [references/source-index.md](../references/source-index.md)：公开来源入口。
