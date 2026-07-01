# Uncertainty Statement Template

## 1. 核心判断结论

- **Strong:** DFT 结论应区分 numerical uncertainty、model uncertainty、pseudopotential uncertainty、finite-size/boundary uncertainty 和 workflow propagation error。
- **Strong:** 没有 uncertainty statement 的定量结论只能视为弱结论。
- **Moderate:** 结论强度应随目标 observable、上游质量和模型适用性分级。
- **Boundary:** AiiDA/provenance 工具可辅助记录，但不替代物理判断。

## 2. 这个问题为什么会出现在 DFT/QE 中？

可复现性不只是保存 input/output，还要说明哪些误差已经通过收敛测试降低，哪些误差来自模型选择，哪些仍是边界条件或版本敏感风险。

## 3. 相关 QE input 字段

| 字段/设置 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `prefix/outdir` | all | 数据链可追踪 | 混用 scratch |
| QE version / command | all | 程序和运行环境可复查 | 不记录版本 |
| pseudo source / `input_dft` | `pw.x` | 模型来源 | 只给文件名 |
| convergence table | workflow record | 数值误差证据 | 只有最终数值 |

## 4. output review 迹象

| output 迹象 | 可能原因 | 回查动作 |
|---|---|---|
| 只有单个结果无 sensitivity | uncertainty 不足 | 补 convergence/model table |
| 来源不可追踪 | provenance BLOCK | 停止定量结论 |
| 不同模型结果差异大 | model uncertainty 主导 | 写强度限制 |

## 5. PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | 数值、模型、pseudo、边界和 provenance 均有记录 | 允许正式结论 |
| WARN | 部分 uncertainty 未量化但边界已说明 | 只写条件结论 |
| BLOCK | 关键来源、参数或 output review 缺失 | 不得写定量结论 |

## 6. 常见误区

- 只报告一个最终数字
- 把收敛表当模型验证
- 不记录 pseudo/functional/U/SOC/vdW 来源
- 不说明 finite-size/boundary
- 把 provenance 工具当物理审稿人

## 7. 应回写到哪些 workflow 页面？

- [standards/output-review-checklist.md](../standards/output-review-checklist.md)
- [standards/calculation-record-template.md](../standards/calculation-record-template.md)
- [workflows/README.md](../workflows/README.md)

## 8. 应回写到哪些 theory-minimum 页面？

- [theory-minimum/README.md](../theory-minimum/README.md)
- [theory-minimum/dft-ks-scf.md](../theory-minimum/dft-ks-scf.md)


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
