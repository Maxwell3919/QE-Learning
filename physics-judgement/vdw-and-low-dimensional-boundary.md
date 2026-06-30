# vdW and Low-Dimensional Boundary

## 1. 核心判断结论

- **Strong:** vdW correction 是模型选择，会改变结构并传递到 bands/phonon/work function。
- **Strong:** 不同 vdW 模型不能无条件横向比较。
- **Boundary:** 低维、层状、吸附和界面问题还必须同时审阅 vacuum、dipole 和 electrostatics。
- **Version-sensitive:** `vdw_corr` 支持模型和限制需查当前 QE 文档。

## 2. 这个问题为什么会出现在 DFT/QE 中？

半局域 GGA 通常不能完整描述长程 dispersion。vdW 修正不是后处理装饰，而会改变几何结构、结合能和响应性质；在低维模型中还会和周期边界、真空、偶极修正耦合。

## 3. 相关 QE input 字段

| 字段/设置 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `vdw_corr` | `pw.x` | 选择 vdW correction | 把 vdW 当普适精度提升 |
| `input_dft` | `pw.x` | 与 vdW 模型一起定义能量面 | relax/phonon 使用不同模型不说明 |
| `assume_isolated/tefield/dipfield` | `pw.x` | 低维/表面 electrostatics 边界 | 结构构建和电势审阅分离 |
| cell/vacuum settings | `CELL_PARAMETERS` | 低维周期镜像影响 | 真空不足仍解释表面性质 |

## 4. output review 迹象

| output 迹象 | 可能原因 | 回查动作 |
|---|---|---|
| 层间距/吸附高度对模型敏感 | vdW 主导结构 | 写模型 sensitivity |
| work function 随真空变化 | electrostatic boundary 不稳 | 检查 potential plateau |
| phonon 受 vdW 模型改变 | 结构能量面改变 | 用同一模型重算下游 |

## 5. PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | vdW 模型、结构、下游和边界条件一致记录 | 允许进入性质分析 |
| WARN | 模型可追踪但 sensitivity 未充分审阅 | 只写模型依赖趋势 |
| BLOCK | relax 和后处理模型混用或低维边界不可追踪 | 停止定量结论 |

## 6. 常见误区

- vdW 一定更准
- 无 vdW relax 后讨论层间结合
- 不同 vdW 模型直接比较总能
- 低维体系只看能量不看电势平台
- 忽略真空和偶极修正

## 7. 应回写到哪些 workflow 页面？

- [workflows/advanced/vdW-corrections.md](../workflows/advanced/vdW-corrections.md)
- [workflows/electronic/work-function.md](../workflows/electronic/work-function.md)
- [workflows/electronic/electrostatic-potential.md](../workflows/electronic/electrostatic-potential.md)

## 8. 应回写到哪些 theory-minimum 页面？

- [theory-minimum/finite-size-and-boundary-conditions.md](../theory-minimum/README.md)
- [theory-minimum/force-stress-relaxation.md](../theory-minimum/force-stress-relaxation.md)


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
