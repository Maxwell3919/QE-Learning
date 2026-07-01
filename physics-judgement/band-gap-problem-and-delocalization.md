# Band Gap Problem and Delocalization

## 1. 核心判断结论

- **Strong:** 普通 LDA/GGA band gap problem 不是 k mesh 或 cutoff 不足的普通数值问题。
- **Strong:** KS gap、fundamental gap、optical gap 需要区分。
- **Moderate:** delocalization/self-interaction error 会影响局域态、缺陷态、电荷转移、磁矩和 PDOS 解释。
- **Boundary:** 定量 gap、excited-state 或 spectroscopy 目标通常需要 hybrid/GW/BSE/TDDFT 或实验对照。

## 2. 这个问题为什么会出现在 DFT/QE 中？

近似 XC functional 的 delocalization 和 derivative-discontinuity 问题会使普通 KS gap 与实验 fundamental gap 不一致。加密 k mesh 可改善采样，但不能改变这种模型误差来源。

## 3. 相关 QE input 字段

| 字段/设置 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `input_dft` | `pw.x` | 决定 XC 模型 | 把 GGA gap 当实验 gap |
| `HUBBARD` | `pw.x` | 修正局域子空间过度离域 | 为匹配 gap 随意调 U |
| hybrid/GW/BSE 入口 | advanced tools | 进入准粒子/激发态边界 | 把普通 bands 当光谱 |
| `projwfc.x` 投影设置 | `projwfc.x` | 检查局域态和 orbital contribution | 过度解释 Lowdin charge |

## 4. output review 迹象

| output 迹象 | 可能原因 | 回查动作 |
|---|---|---|
| gap 异常偏小或金属性矛盾 | functional / delocalization error 或 smearing/k mesh 问题 | 先排数值，再写模型边界 |
| PDOS 局域态过宽或磁矩偏小 | 过度离域或投影定义问题 | 回查 DFT+U/hybrid/Wannier |
| DOS 与 bands gap 不一致 | k mesh、zero、smearing 或数据链问题 | 交叉检查 SCF/NSCF/DOS |

## 5. PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | 只做 KS 趋势、态成分或相对比较，且解释边界明确 | 允许进入电子结构讨论 |
| WARN | 讨论 gap 数值但未使用更高层方法或实验对照 | 只能写趋势 |
| BLOCK | 用普通 DFT gap 支撑定量光学、输运或激发态结论 | 转 advanced/excited-state |

## 6. 常见误区

- 写“加密 k 点修复 GGA gap”
- 把 zero DOS at EF 直接当 experimental gap
- 用 U 只调 gap 不记录来源
- 忽略小带隙体系 smearing 风险
- PDOS 图像替代电荷局域性审查

## 7. 应回写到哪些 workflow 页面？

- [workflows/electronic/bands.md](../workflows/electronic/bands.md)
- [workflows/electronic/dos.md](../workflows/electronic/dos.md)
- [workflows/electronic/pdos.md](../workflows/electronic/pdos.md)
- [workflows/advanced/hybrid-functional-overview.md](../workflows/advanced/hybrid-functional-overview.md)

## 8. 应回写到哪些 theory-minimum 页面？

- [theory-minimum/dft-ks-scf.md](../theory-minimum/dft-ks-scf.md)
- [theory-minimum/magnetism-soc-dftu.md](../theory-minimum/magnetism-soc-dftu.md)


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
