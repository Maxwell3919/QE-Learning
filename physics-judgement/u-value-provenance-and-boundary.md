# U Value Provenance and Boundary

## 本页解决什么问题

本页用于判断 DFT+U 中的 U 值是否有足够 provenance，以及能否支撑当前科研结论。DFT+U 改变的是局域子空间上的模型，不是为了让 band gap 接近预期而任意调节的数值参数。U 的解释必须绑定 projector、Hubbard manifold、pseudopotential、functional、结构状态和求取策略。

## 典型 output 现象

| output 现象 | 可能含义 | 回查动作 |
|---|---|---|
| Hubbard setup 回显缺失或与 input 不一致 | DFT+U 未按预期启用或语法/版本不匹配 | 查 `HUBBARD` card、QE version、stdout |
| occupation matrix / local occupation 随 U 大幅变化 | 局域子空间和自洽电子结构被模型改变 | 记录模型变化，不与无 U 直接混写 |
| gap、磁矩或结构随 U 改变 | U 对目标 observable 敏感 | 写 sensitivity 和 U provenance |
| output 有 projector、label、Hubbard warning | manifold 或 species label 风险 | 回查 `ATOMIC_SPECIES`、UPF、Hubbard card |
| DFT+U relax 与 bands/DOS 使用不同 U | workflow 传播错误 | 重新建立同一 U 数据链 |

## 可能原因

- 数值误差：SCF threshold、kmesh、smearing 或 cutoff 未对 DFT+U 后的目标量收敛。
- 模型误差：U 改变 localized manifold 的能量惩罚和 occupation，不是数值微调。
- 赝势误差：projector/manifold 与 PP valence 或 UPF 信息不匹配。
- 边界条件误差：局域环境、氧化态、结构状态或磁性构型改变了 U 的迁移边界。
- 后处理误差：PDOS 或 occupation matrix 被过度解释为充分价态证据。
- workflow 传播误差：同一图或结论中混用不同 U、functional、PP 或结构状态。
- 真实物理效应：局域电子、磁性或电荷有序可能对 U 敏感，但需要模型边界和对照支撑。

## 必查 QE input/output

| 对象 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `HUBBARD` card | `pw.x` | 指定 Hubbard atom label、manifold、projector、U/J/V | 不记录 projector 或作用通道 |
| legacy DFT+U fields | `pw.x` | 旧输入或旧文献复现边界 | 混用新旧语法且不记录 QE version |
| `hp.x` / DFPT U input | `hp.x` | linear-response / DFPT U 求取入口 | 只报 U 数值，不报求取设置 |
| `input_dft` | `pw.x` | U 与 functional 共同定义模型 | 换 functional 后沿用 U |
| pseudopotential / UPF | `pw.x` | valence、projector、semicore 和 XC 边界 | 换 PP 后沿用 U |
| `starting_magnetization` / final magnetization | `pw.x` | U 与磁态耦合 | 单一初态就宣称磁性结论 |
| Hubbard occupations / matrices | stdout | 局域子空间占据证据 | 当作无边界价态证明 |

## 判断规则

- U 作用于选定 localized manifold；没有 manifold 和 projector，U 数值本身不可解释。
- U 不是元素常数。它依赖 projector、PP、functional、结构、局域环境、磁性状态、求取方法和是否自洽更新。
- Literature U、empirical U、linear-response U、DFPT U、self-consistent U、U+V 是不同 provenance 类型。它们可以作为不同模型来源，但必须明确标注。
- 只为了让 gap 接近某个目标而调 U，若没有模型说明和 sensitivity，不足以支撑可靠物理结论。
- DFT+U 总能、结构、bands、phonon 或磁性比较必须在同一 U 模型和同一数据链下审阅。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | U provenance、projector/manifold、PP、functional、结构状态、QE version 和下游复算链完整；warning 已解释 | 允许进入带 DFT+U 边界的结构、磁性、bands/DOS 或 phonon 判断 |
| WARN | U 来源可追踪但迁移边界或 sensitivity 不充分；用于探索模型依赖 | 只允许趋势、对照或内部筛查 |
| BLOCK | U 无来源；跨 PP/functional/structure 混用；只因匹配 gap 取 U 且无说明；Hubbard setup/output 不可复查 | 不允许进入定量 DFT+U 结论 |

## 不能做出的过度结论

- 不能把 U 当作元素固定常数。
- 不能把 DFT+U 写成修复所有强关联问题的方法。
- 不能用同一 U 无说明迁移到不同 PP、functional、结构或局域环境。
- 不能只凭 gap 变大就宣称模型更可靠。
- 不能把 Hubbard occupation matrix 单独写成完整价态或电荷有序证明。

## 下游影响

- `relax/vc-relax`：U 改变 force、stress 和结构；后处理应基于同一 U 的 final static SCF。
- `bands/DOS/PDOS`：U 改变 gap、orbital ordering、magnetization 和 projection。
- `phonon/DFPT`：U 改变结构和 force constants；必要时需要对应的 DFT+U 响应边界。
- `Wannier/GW/BSE`：U 模型下的 electronic structure 只能作为特定模型输入，不能与无 U 结果混写。

## 与 theory-minimum / workflows / standards 的关系

- 理论回查：[Magnetism, SOC and DFT+U](../theory-minimum/magnetism-soc-dftu.md)、[Pseudopotentials](../theory-minimum/pseudopotentials.md)
- Workflow 回查：[DFT+U overview](../workflows/advanced/dft-plus-u.md)、[SCF](../workflows/ground-state/scf.md)、[PDOS](../workflows/electronic/pdos.md)
- 相关判断：[Numerical vs model error](numerical-vs-model-error.md)、[Band gap problem and delocalization](band-gap-problem-and-delocalization.md)
- 记录规范：[calculation record template](../standards/calculation-record-template.md)、[output review checklist](../standards/output-review-checklist.md)

## 来源与结论强度

| 结论 | 强度 | 来源边界 |
|---|---|---|
| U 不是元素常数，必须绑定 projector/manifold 和 provenance | Strong | Cococcioni/de Gironcoli、Timrov/Marzari/Cococcioni |
| `HUBBARD` card 和 `hp.x` 输入语法需查当前 QE 文档 | Version-sensitive | QE `INPUT_PW`、Hubbard input guide、HP docs |
| 用 U 拟合 gap 需要模型边界 | Boundary | DFT+U 方法文献与 band-gap judgement |
| DFT+U 结果应写 model sensitivity | Moderate | DFT error / reproducibility literature |

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE Hubbard input guide: <https://www.quantum-espresso.org/Doc/user_guide_PDF/Hubbard_input.pdf>
- Cococcioni and de Gironcoli, Linear response approach to the calculation of the effective interaction parameters in DFT+U.
- Timrov, Marzari and Cococcioni, Hubbard parameters from DFPT.
- 本仓库：[canonical literature](../references/canonical-literature.md)、[source index](../references/source-index.md)
