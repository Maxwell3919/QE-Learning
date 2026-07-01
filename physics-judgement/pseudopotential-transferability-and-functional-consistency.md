# Pseudopotential Transferability and Functional Consistency

## 1. 核心判断结论

- **Strong:** pseudopotential 是 electron-ion interaction model，不只是文件名。
- **Strong:** 换 pseudopotential 后必须重做 cutoff、k mesh 和目标 observable 收敛。
- **Strong:** pseudo 的 XC、valence、relativistic 类型必须与 `input_dft`、SOC、DFT+U 目标一致或有说明。
- **Boundary:** transferability 不能由单个 `JOB DONE` 或单次 cutoff 收敛证明。

## 2. 这个问题为什么会出现在 DFT/QE 中？

Plane-wave DFT 将 core-electron 作用封装进 pseudopotential / PAW 数据。不同 NC/USPP/PAW 文件对 `ecutrho`、force、stress、phonon、SOC 和 EPC 的敏感性不同。

## 3. 相关 QE input 字段

| 字段/设置 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `ATOMIC_SPECIES` | `pw.x` | 指定 UPF 文件和 species 标签 | 标签和 UPF 来源混乱 |
| `pseudo_dir` | `pw.x` | 定位 pseudo 文件 | 不同 run 读取不同库 |
| `ecutwfc/ecutrho` | `pw.x` | 与 pseudo 类型强相关 | 沿用旧 pseudo 的 cutoff |
| `lspinorb/noncolin` | `pw.x` | SOC 需要相对论 pseudo 条件 | 用 scalar-relativistic pseudo 解释 SOC |

## 4. output review 迹象

| output 迹象 | 可能原因 | 回查动作 |
|---|---|---|
| loaded pseudopotentials 与记录不一致 | 读取错误文件或库 | 核对 output header 和 input |
| 建议 cutoff 与实际 cutoff 不匹配 | basis 风险 | 重做 cutoff scan |
| SOC run 中 pseudo 类型不清 | 相对论模型不满足 | 回查 UPF 元信息 |

## 5. PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | UPF 来源、XC、valence、类型、cutoff 和目标量收敛记录完整 | 允许进入下游 |
| WARN | pseudo 可追踪但 transferability 或目标量敏感性未充分验证 | 只允许趋势判断 |
| BLOCK | pseudo 来源不明、模型混用或换 pseudo 后未重做收敛 | 停止下游 |

## 6. 常见误区

- 把 pseudo 当普通输入附件
- 只看 recommended cutoff 不做目标量检查
- 混用 functional 与 pseudo
- 忽略 semicore/valence 差异
- SOC 不查 fully relativistic 条件

## 7. 应回写到哪些 workflow 页面？

- [workflows/ground-state/cutoff-convergence.md](../workflows/ground-state/cutoff-convergence.md)
- [workflows/advanced/noncollinear-soc.md](../workflows/advanced/noncollinear-soc.md)
- [workflows/advanced/pseudopotential-generation-overview.md](../workflows/advanced/pseudopotential-generation-overview.md)

## 8. 应回写到哪些 theory-minimum 页面？

- [theory-minimum/pseudopotentials.md](../theory-minimum/pseudopotentials.md)
- [theory-minimum/plane-wave-cutoff.md](../theory-minimum/plane-wave-cutoff.md)


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
