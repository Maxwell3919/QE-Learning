# Numerical vs Model Error

## 1. 核心判断结论

- **Strong:** cutoff、k/q mesh、SCF threshold、smearing 属于 numerical convergence 问题。
- **Strong:** functional、pseudopotential、U、SOC、vdW、slab boundary 属于 model/boundary choice。
- **Strong:** numerical convergence 不能修复 band-gap problem、错误 U 来源、错误 pseudo 或错误边界条件。
- **Moderate:** DFT 结论应同时写 numerical uncertainty 和 model uncertainty。

## 2. 这个问题为什么会出现在 DFT/QE 中？

QE output 中“收敛”通常只说明某个离散化或自洽问题满足阈值。科研判断还要问：这个阈值对应的模型是否适合目标 observable？如果模型错了，加密 cutoff 或 k mesh 只会更精确地求出错误模型的结果。

## 3. 相关 QE input 字段

| 字段/设置 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `ecutwfc/ecutrho` | `pw.x` | basis discretization | 拿 cutoff 收敛证明 functional 正确 |
| `K_POINTS` | `pw.x` | BZ sampling | 用 k convergence 修复系统性 gap error |
| `conv_thr/tr2_ph` | `pw.x/ph.x` | SCF/response convergence | 忽略模型边界 |
| `input_dft/HUBBARD/vdw_corr/lspinorb` | `pw.x` | 物理模型选择 | 当作数值微调 |

## 4. output review 迹象

| output 迹象 | 可能原因 | 回查动作 |
|---|---|---|
| 目标量随 cutoff/k mesh 变化 | 数值误差未收敛 | 做单变量扫描 |
| 目标量对 functional/U/SOC/vdW 改变更敏感 | 模型不确定性主导 | 写 model sensitivity |
| `JOB DONE` 但 warning 未解释 | 程序完成不等于可信 | 按 output-review checklist 复查 |

## 5. PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | 数值收敛和模型边界均已记录，目标量允许下游使用 | 允许进入正式下游 |
| WARN | 数值已收敛但模型边界未完全验证 | 只能写条件性结论 |
| BLOCK | 模型条件不满足或来源不可追踪 | 不得用更密参数掩盖问题 |

## 6. 常见误区

- 把所有误差都归因于 k 点不够
- 把 pseudo 换库当普通重跑
- 把 U、SOC、vdW 当调参开关
- 只报告最终数字不报告 sensitivity
- 用单个 output 代替 uncertainty statement

## 7. 应回写到哪些 workflow 页面？

- [workflows/ground-state/cutoff-convergence.md](../workflows/ground-state/cutoff-convergence.md)
- [workflows/ground-state/kpoint-convergence.md](../workflows/ground-state/kpoint-convergence.md)
- [workflows/ground-state/smearing-and-occupations.md](../workflows/ground-state/smearing-and-occupations.md)

## 8. 应回写到哪些 theory-minimum 页面？

- [theory-minimum/plane-wave-cutoff.md](../theory-minimum/plane-wave-cutoff.md)
- [theory-minimum/pseudopotentials.md](../theory-minimum/pseudopotentials.md)
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
