# SOC and Symmetry Boundary

## 1. 核心判断结论

- **Strong:** SOC/noncollinear 不是只打开 `lspinorb`。
- **Strong:** SOC 判断需要 fully relativistic pseudopotential、`noncolin`、symmetry、磁构型和 bands/Wannier 后处理一致。
- **Boundary:** topology、magnetic anisotropy 和 spin texture 结论需要额外 invariant、surface state 或有效模型证据。
- **Version-sensitive:** noncollinear/SOC 字段和输出格式需查当前 `INPUT_PW`。

## 2. 这个问题为什么会出现在 DFT/QE 中？

SOC 改变 Hamiltonian 的自旋空间结构和对称性；noncollinear magnetism 又改变磁矩表达方式。许多 band degeneracy、splitting 和 topology 解释都依赖相对论 pseudo 与对称性处理。

## 3. 相关 QE input 字段

| 字段/设置 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `noncolin` | `pw.x` | 启用 noncollinear 计算 | 只改一个开关不改磁构型审阅 |
| `lspinorb` | `pw.x` | 启用 SOC | 无 fully relativistic pseudo |
| `starting_magnetization` | `pw.x` | 初始磁矩 | 当作最终磁性 |
| `noinv/nosym` 等 symmetry 设置 | `pw.x` | 影响 degeneracy 和 band interpretation | 未记录 symmetry reduction |

## 4. output review 迹象

| output 迹象 | 可能原因 | 回查动作 |
|---|---|---|
| magnetization components 改变 | noncollinear 态与初态不同 | 审阅总磁矩和局域磁矩 |
| band degeneracy lifted | SOC/symmetry 改变 | 比较 SOC/non-SOC 数据链 |
| topological claim 缺少 invariant | 解释层不足 | 转 Wannier/topology 边界 |

## 5. PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | pseudo、SOC 字段、symmetry、磁构型和下游数据链一致 | 允许进入 SOC 分析 |
| WARN | SOC 结果可追踪但 topology/MAE 等解释证据不足 | 只能写边界性结论 |
| BLOCK | 无相对论 pseudo 或混用 SOC/non-SOC bands 解释 | 停止结论 |

## 6. 常见误区

- 初始磁矩等于最终磁性
- SOC 只是 `lspinorb` 开关
- 无 SOC bands 解释拓扑
- 忽略多个磁构型比较
- 未记录 symmetry 改变

## 7. 应回写到哪些 workflow 页面？

- [workflows/advanced/noncollinear-soc.md](../workflows/advanced/noncollinear-soc.md)
- [workflows/advanced/spin-polarization.md](../workflows/advanced/spin-polarization.md)
- [workflows/advanced/wannier90-overview.md](../workflows/advanced/wannier90-overview.md)

## 8. 应回写到哪些 theory-minimum 页面？

- [theory-minimum/magnetism-soc-dftu.md](../theory-minimum/magnetism-soc-dftu.md)
- [theory-minimum/kpoints-symmetry-kpath.md](../theory-minimum/kpoints-symmetry-kpath.md)


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
