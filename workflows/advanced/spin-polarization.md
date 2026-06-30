# Spin polarization workflow

## 页面定位

- 对应学习路线：[learn/09-feature-expansion-map.md](../../learn/09-feature-expansion-map.md)
- 上游依赖：基础 SCF、收敛性、结构和赝势记录
- 下游用途：高级物性分析或专题 workflow
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 适用问题

本页用于从普通 SCF 进入自旋极化 `pw.x` workflow 时的边界判断。适用的问题包括：

- 需要判断某个结构是否存在自旋极化基态。
- 需要比较不同初始磁矩设置是否收敛到同一电子态或不同电子态。
- 需要记录 total magnetization、absolute magnetization 和局域磁矩输出，并判断它们是否支持后续 bands、DOS、PDOS 或 phonon 计算。
- 需要把“非磁 SCF 已收敛”和“自旋极化 SCF 可用于下游”分开审阅。

本页不展开具体材料案例，不提供完整 input/output，也不替代磁性、交换关联或自旋相关物理教材。

## 进入该 workflow 前必须已经掌握什么

进入前应已经能独立完成并审阅基础 SCF：

- 能读懂 [SCF workflow](../ground-state/scf.md) 中的 `prefix/outdir`、赝势、cutoff、k 点、occupation/smearing、SCF convergence 和 warning 记录。
- 能说明 `occupations`、`smearing`、`degauss` 对金属或小带隙体系的数值影响，并知道 occupation 策略改变后需要重新审阅收敛性。
- 已有结构、赝势来源、QE 版本、运行命令和 output review 记录；不能只保留最后能量。
- 能区分 total magnetization、absolute magnetization、atomic magnetic moments 和投影磁矩等不同输出层级。
- 知道自旋极化会改变电子结构问题本身，不能把它当作普通 SCF 后处理开关。

## 关键 QE 程序或输入字段

主要程序仍是 `pw.x`。自旋极化属于 ground-state Hamiltonian 和自洽过程的一部分，不是独立后处理程序。

| 字段或程序 | 所属位置 | 审阅重点 | 常见误用 |
|---|---|---|---|
| `pw.x` `calculation='scf'` | `&CONTROL` | 先得到可信自旋极化 ground state | 直接从非磁 SCF output 推断自旋结论 |
| `nspin` | `&SYSTEM` | 共线自旋极化计算通常使用 `nspin=2` | 只写 `starting_magnetization`，忘记检查 output 中是否确实启用自旋 |
| `starting_magnetization(i)` | `&SYSTEM` | 为每个相关原子类型设置初始自旋极化，并在记录中保留所有尝试 | 把初始磁矩当作最终磁矩，或只记录“开了磁性” |
| `tot_magnetization` | `&SYSTEM` | 需要约束总磁矩时才使用，并单独标注为受约束比较 | 未说明约束条件就比较能量或磁矩 |
| `occupations` | `&SYSTEM` | 检查 fixed、smearing、tetrahedra 或 from_input 是否与体系和下游目标一致 | 开自旋后沿用非磁 occupation 策略而不复查 |
| `smearing` / `degauss` | `&SYSTEM` | 金属或近金属自旋态下审阅 Fermi 附近占据和收敛稳定性 | 用一个 smearing 结果直接判断磁性强弱 |
| `K_POINTS` | input card | 自旋态、能量差、DOS/PDOS 对 k 点可能更敏感 | 用粗 k 点比较近简并自旋态 |
| `prefix/outdir` | `&CONTROL` | 自旋和非磁 workflow 应清楚隔离或明确命名 | 复用旧 scratch 导致 output review 不能追溯 |

`noncolin`、`lspinorb` 和 DFT+U 属于相邻高级页。若同时引入这些开关，本页只能作为自旋极化前置审阅，不能覆盖 SOC 或 Hubbard 参数的完整边界。

## output review 要点

每一次自旋极化 SCF 至少应补充以下 output review 字段：

```markdown
## Spin Output Review

- Spin setting reported:
- Starting magnetization echoed:
- Occupation / smearing / degauss:
- SCF convergence:
- Final total energy:
- Total magnetization:
- Absolute magnetization:
- Atomic magnetic moments / magnetization projections:
- Fermi energy / occupation behavior:
- Warnings / symmetry messages:
- Compared starting states:
- PASS / WARN / BLOCK:
- Reason:
- Allowed downstream workflows:
```

审阅时优先看这些问题：

- output 是否确认了预期的自旋设置，而不是只看 input。
- `starting_magnetization` 是否被记录并能和最终磁矩区分。
- total magnetization 和 absolute magnetization 是否都记录；二者不能互相替代。
- atomic magnetic moments 或投影磁矩是否只是用于解释局域趋势，而不是未经定义就当作充分物理结论。
- 多个初始磁矩设置是否分别达到 SCF convergence；若收敛到不同态，应比较总能、磁矩、占据和 warning，而不是只选磁矩最大的态。
- occupation/smearing/degauss 是否与 Fermi energy、占据行为和后续 DOS/PDOS 目标一致。
- 自旋极化 bands、DOS、PDOS 是否读取同一自旋设置下生成的 ground-state 数据，并保持 `prefix/outdir`、赝势、cutoff、k 点策略可追溯。
- `JOB DONE` 只说明程序结束；仍需检查 SCF convergence、warning 和磁矩输出是否完整。

## 常见风险

- 初始磁矩未记录，导致别人无法复现实验过哪些自旋态。
- 把 `starting_magnetization` 当成最终磁矩，或把 final magnetization 当成 input 设定。
- 把 total magnetization、absolute magnetization 和 atomic magnetic moments 混为一谈。
- 用受 `tot_magnetization` 约束的结果和未约束结果直接比较，却没有标注约束条件。
- 开自旋后沿用非磁 cutoff、k 点、smearing 收敛结论，不重新判断目标性质是否稳定。
- 只比较一个初始磁矩设置，忽略可能存在的亚稳态或收敛路径依赖。
- 用不一致的 `prefix/outdir` 串接 bands、DOS 或 PDOS，使下游读取到错误 ground-state 数据。
- 将自旋极化、SOC、DFT+U 同时打开后仍按普通共线自旋页审阅；这时应转到相邻高级 workflow。

## 与基础 workflow 的关系

- 自旋极化 SCF 是基础 SCF 的扩展，不是 SCF 之后的绘图步骤。基础 SCF 的所有审阅项仍然保留。
- 非磁 SCF 可以作为参考或初始判断，但不能自动作为自旋极化下游的前置状态。
- cutoff、k 点、occupation/smearing 在非磁计算中通过，不代表自旋态能量差、磁矩或 DOS 已经数值稳定。
- bands、DOS、PDOS 必须明确使用自旋极化 ground state，并在图表或记录中标明 spin channel 或自旋处理方式。
- phonon 或响应性质若基于自旋极化基态，应先确认 SCF 质量、磁矩状态和 occupation 策略足以支撑该下游任务。
- 本页的职责是定义进入自旋极化 workflow 的最低审阅边界；更深入的磁有序、非共线、SOC 或 DFT+U 问题应分别进入专题页。

## PASS / WARN / BLOCK 边界

| 状态 | 判据 |
|---|---|
| PASS | SCF 收敛；自旋设置、初始磁矩、occupation、total/absolute magnetization 和局域磁矩输出均已记录；下游读取路径清楚 |
| WARN | SCF 收敛但只测试了少量初始磁矩、磁矩较小或对 smearing/k 点敏感；可继续探索但不能下强结论 |
| BLOCK | SCF 未收敛、磁矩输出缺失、scratch 来源混乱、occupation 与目标不一致，或受约束/未约束结果混比而无记录 |

## 资料来源

- QE `pw.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE documentation: <https://www.quantum-espresso.org/documentation/>
- 本仓库：[SCF workflow](../ground-state/scf.md)
- 本仓库：[Magnetism, SOC and DFT+U](../../theory-minimum/magnetism-soc-dftu.md)
- 本仓库：[Output Review Checklist](../../standards/output-review-checklist.md)
