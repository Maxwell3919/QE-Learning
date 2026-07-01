# U Value Provenance and Boundary

## 1. 核心判断结论

- **Strong:** Hubbard U 不是 universal element parameter。
- **Strong:** U 必须记录来源、Hubbard manifold、projector、functional、pseudopotential 和是否 self-consistent。
- **Strong:** DFT+U 不应作为单纯 gap-fitting 参数。
- **Version-sensitive:** QE `HUBBARD` card、hp.x 和 DFT+U+V 语法需按当前官方文档核验。

## 2. 这个问题为什么会出现在 DFT/QE 中？

DFT+U 通过局域子空间修正部分 self-interaction / delocalization 问题。U 的数值依赖投影子空间、赝势、泛函、结构和计算方法；因此同一个元素在不同环境下不能当作固定常数。

## 3. 相关 QE input 字段

| 字段/设置 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `HUBBARD` card | `pw.x` | 定义 Hubbard manifold 和 U/V | 未记录 projector/manifold |
| `lda_plus_u`/版本相关字段 | `pw.x` | 旧/新语法边界 | 混用不同 QE 版本语法 |
| `hp.x` 输入 | `hp.x` | linear-response / DFPT U 入口 | 不记录 U 计算设置 |
| `starting_magnetization` | `pw.x` | 与局域态和磁性构型耦合 | 把初态当最终磁性 |

## 4. output review 迹象

| output 迹象 | 可能原因 | 回查动作 |
|---|---|---|
| occupation matrix / magnetic moment 变化 | U 改变局域态 | 记录模型变化 |
| gap 随 U 单调调节 | 可能 gap fitting 风险 | 核验 U 来源 |
| 不同 U 改变结构/phonon | 模型敏感 | 写 sensitivity |

## 5. PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | U 来源、manifold、projector、版本和敏感性记录完整 | 允许进入 DFT+U 结论 |
| WARN | U 来源可追踪但未做 sensitivity 或自洽 | 只能写模型依赖趋势 |
| BLOCK | 未记录 U 来源或仅为匹配 gap 调 U | 不得写定量结论 |

## 6. 常见误区

- 把 U 当元素常数
- 只为匹配 gap 取 U
- 不同 U 总能横比不说明模型变化
- 忽略 U 对结构/phonon 的影响
- 未记录 QE 版本下的 `HUBBARD` 语法

## 7. 应回写到哪些 workflow 页面？

- [workflows/advanced/dft-plus-u.md](../workflows/advanced/dft-plus-u.md)
- [workflows/advanced/spin-polarization.md](../workflows/advanced/spin-polarization.md)

## 8. 应回写到哪些 theory-minimum 页面？

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
