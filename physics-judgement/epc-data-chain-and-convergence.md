# EPC Data Chain and Convergence

## 1. 核心判断结论

- **Strong:** EPC 可信度依赖 relaxed structure -> SCF -> DFPT phonon -> Wannier validation -> dense k/q interpolation -> observable 的完整链。
- **Strong:** 不能只看 `lambda` 或 `Tc` 判断 EPC 结论。
- **Strong:** k/q mesh、smearing、Fermi surface、phonon 和 Wannier window 都可能主导误差。
- **Version-sensitive:** EPW/QE electron-phonon 字段和文件链需按当前官方文档核验。

## 2. 这个问题为什么会出现在 DFT/QE 中？

EPC matrix elements 连接电子态、phonon 模式和 occupation/Fermi surface。它比普通 bands/DOS 更依赖上游一致性，并且不同 observable 对数据链的要求不同。

## 3. 相关 QE input 字段

| 字段/设置 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `fildvscf` | `ph.x` | 保存响应势供 EPC 使用 | 未保存/文件链断裂 |
| `electron_phonon` | `ph.x` | QE 内 EPC 模式入口 | 不查版本语义 |
| EPW k/q mesh | EPW | dense interpolation | 只看 coarse grid |
| Wannier window/projection | Wannier90/EPW | 插值质量前提 | 未回对 QE bands |

## 4. output review 迹象

| output 迹象 | 可能原因 | 回查动作 |
|---|---|---|
| lambda/Tc 对 mesh 敏感 | Fermi surface 或 q mesh 未收敛 | 做 observable-specific convergence |
| alpha2F 或 linewidth 异常 | phonon/Wannier/occupations 问题 | 回查上游链 |
| Wannier bands 不匹配 | 插值无效 | BLOCK EPC |

## 5. PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | 结构、SCF、DFPT、Wannier、dense k/q 和目标 observable 均通过审阅 | 允许报告 EPC 目标量 |
| WARN | 链条可追踪但 mesh/window/smearing 敏感 | 只写探索趋势 |
| BLOCK | 只给 lambda/Tc 或未验证 Wannier/phonon | 不得写 EPC 结论 |

## 6. 常见误区

- EPC 判断只看单个 lambda 数值
- Tc 数字不带收敛说明
- 未审 Fermi surface smearing
- phonon 有虚频仍算 EPC
- Wannier 未验证直接 EPW

## 7. 应回写到哪些 workflow 页面？

- [workflows/advanced/electron-phonon-overview.md](../workflows/advanced/electron-phonon-overview.md)
- [workflows/advanced/wannier90-overview.md](../workflows/advanced/wannier90-overview.md)
- [workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)

## 8. 应回写到哪些 theory-minimum 页面？

- [theory-minimum/dfpt-phonons.md](../theory-minimum/dfpt-phonons.md)
- [theory-minimum/smearing-metals.md](../theory-minimum/smearing-metals.md)


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
