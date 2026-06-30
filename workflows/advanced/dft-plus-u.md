# DFT+U workflow

## 页面定位

- 对应学习路线：[learn/09-feature-expansion-map.md](../../learn/09-feature-expansion-map.md)
- 上游依赖：基础 SCF、收敛性、结构和赝势记录
- 下游用途：高级物性分析或专题 workflow
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)
- 页面等级：B 级高级边界页。本页说明什么时候进入 DFT+U、需要审哪些输入和 output；不替代 DFT+U 理论教材，也不提供材料案例或通用 U 默认值。

## 适用问题

DFT+U 是在选定局域轨道流形上加入 Hubbard correction 的 `pw.x` workflow，用于普通半局域 DFT 难以合理描述局域电子占据、磁性态、能隙或价态趋势的场景。

适合进入本 workflow 的问题应同时满足：

- 目标物理问题明确依赖某些局域轨道的占据、磁矩、能隙、相对能量或价态判断。
- 已经能说明为什么普通 DFT 结果不足，而不是只为了“调大 band gap”。
- 有可追溯的 U/J/V 来源，例如文献同类设置、线性响应/DFPT 计算、项目内标定策略，或明确标记为敏感性扫描。
- 下游比较只在同一 Hubbard 定义、同一赝势族、同一结构/收敛策略下进行。

不适合本页直接解决的问题：

- 不知道目标 Hubbard 轨道或 U 来源，只想套用一个默认参数。
- 尚未完成基础 SCF、cutoff、k 点、occupation/smearing 和赝势记录。
- 需要完整推导 DFT+U 泛函、线性响应求 U 或 DFT+U+V 理论细节；这些应进入专题理论或官方文档。

## 进入该 workflow 前必须已经掌握什么

- [SCF workflow](../ground-state/scf.md)：能从 output 判断 `pw.x` 是否真正自洽收敛，而不只看 `JOB DONE`。
- cutoff/k 点/occupation 收敛：开 DFT+U 后 Hamiltonian 改变，旧收敛结论不能自动继承。
- PDOS/轨道判断：能区分元素标签、原子种类、目标轨道流形和 projected character。
- 磁性基础：能记录 `nspin`、初始磁矩、最终 total/absolute/local moments，并知道磁性解可能依赖初始条件。
- 赝势记录：能追踪赝势文件、泛函族、价电子构型、是否包含 Hubbard projector 所需的原子轨道信息。
- U/J/V 来源记录：能写清楚数值来自哪里、单位是什么、适用于哪个 projector 和哪套计算设置。

## 关键 QE 程序或输入字段

主要程序是 `pw.x`。DFT+U 会改变 ground-state Hamiltonian 的输入设置，不属于单独后处理步骤；因此 relax、SCF、NSCF、bands、DOS/PDOS 等下游任务都必须与同一 Hubbard 定义保持一致。

| 字段或程序 | 所在位置 | 用途 | 边界与审阅点 |
|---|---|---|---|
| `HUBBARD` card | `pw.x` input card | 现代 QE DFT+U/DFT+U+J/DFT+U+V 的 Hubbard 参数入口 | card 行必须写 projector 类型，例如 `HUBBARD (ortho-atomic)`；具体语法按 QE 版本官方文档核对 |
| `U label-manifold value` | `HUBBARD` card | 对指定 `ATOMIC_SPECIES` 标签和轨道流形施加 U | `label` 必须匹配 `ATOMIC_SPECIES`；`manifold` 如 `3d`、`2p`、`4f`；Hubbard 参数单位为 eV |
| `J0` / `J` / `B` / `E2` / `E3` | `HUBBARD` card | 进入 DFT+U+J 或更细的参数化 | 不应在不了解目标泛函形式时混用；来源和语法必须逐项记录 |
| `V` | `HUBBARD` card | 进入 DFT+U+V 的原子对相互作用 | 需要原子索引与相互作用对定义；不属于基础 DFT+U 入门默认项 |
| `ALPHA` | `HUBBARD` card | 线性响应计算 U/V 时的 perturbation 参数 | 它不是生产计算中随意调节的 U；应标明用途 |
| projector option | `HUBBARD (...)` | 定义 Hubbard projector 构造方式 | 官方列出 `atomic`、`ortho-atomic`、`norm-atomic`、`wf`、`pseudo`；没有通用默认值 |
| `lda_plus_u` / `Hubbard_U(i)` / `U_projection_type` | legacy `&SYSTEM` 风格 | 旧 QE 输入风格 | 只用于阅读或复现实验性旧输入；新文档应优先写现代 `HUBBARD` card，并标明 QE 版本边界 |
| `nspin` / `starting_magnetization` | `&SYSTEM` | DFT+U 常与自旋极化问题耦合 | 初始磁矩和最终磁矩都要记录；不同初始条件可能收敛到不同局域解 |
| `prefix` / `outdir` | `&CONTROL` | 连接 DFT+U SCF 与后续 NSCF/bands/DOS/PDOS | 不要让普通 DFT scratch 与 DFT+U scratch 混用 |

### `HUBBARD` card 与旧 `Hubbard_U` 写法边界

现代 QE 文档把 Hubbard 参数集中在 `HUBBARD` card 中，并在 card 名称后指定 projector 类型。旧教程或旧输入中常见的 `lda_plus_u = .true.`、`Hubbard_U(i)`、`Hubbard_J(...)`、`U_projection_type` 属于 legacy 风格。阅读旧文件时可以保留原样复现，但新学习笔记应写清楚：

- 使用的 QE 版本。
- 本次输入采用现代 `HUBBARD` card 还是 legacy `&SYSTEM` 字段。
- 如果从旧写法迁移到新写法，`ATOMIC_SPECIES` 中的标签、轨道流形、U/J 数值和 projector 类型是否一一对应。
- 不把 `Hubbard_U(i)` 的 species 序号写法与 `U Element-3d value` 的 label-manifold 写法混在同一份新输入中。

### 投影通道与赝势一致性

DFT+U 的 Hubbard correction 施加在被选择的 projector/轨道流形上；因此“对哪个元素加 U”不够，必须同时记录：

- `ATOMIC_SPECIES` 标签：例如同一元素若因磁性或价态被拆成多个 species，Hubbard 行应指向正确标签。
- 轨道流形：如 `3d`、`2p`、`4f`，必须与物理问题和赝势可用轨道信息一致。
- projector 类型：`atomic`、`ortho-atomic`、`norm-atomic`、`wf`、`pseudo` 的含义不同，结果不可无条件混比。
- 赝势前提：使用 `atomic`/`ortho-atomic`/`norm-atomic` 时依赖赝势中的原子轨道信息；使用 `pseudo` projector 时，官方文档特别要求带 +U 的原子赝势包含 all-electron atomic orbitals 信息。
- 力和应力：官方 `INPUT_PW` 当前页说明 forces/stress 目前只对 `atomic`、`ortho-atomic` 和 `pseudo` Hubbard projectors 实现；涉及 relax/vc-relax 时必须核对 projector 支持。

## output review 要点

```markdown
## DFT+U Output Review

- QE program / version:
- Input syntax: modern HUBBARD card / legacy lda_plus_u:
- Hubbard projector option:
- Hubbard channels: label-manifold list:
- Hubbard parameters and units:
- Pseudopotentials loaded:
- Pseudopotential / projector consistency:
- SCF convergence:
- Magnetic setup and final moments:
- Occupation matrix / Hubbard occupations printed:
- Total energy / band gap / Fermi level:
- Warnings or ignored Hubbard fields:
- Scratch separation from non-DFT+U runs:
- U/J/V source recorded:
- PASS / WARN / BLOCK:
- Reason:
- Allowed downstream workflows:
```

审阅重点：

- output 中是否回显 Hubbard 设置、projector 类型、目标 label-manifold 和参数数值。
- 是否有 warning 表明 Hubbard card、projector、赝势、对称性、自旋或 occupation 设置不符合预期。
- `ATOMIC_SPECIES` 读入的赝势文件是否与记录一致；不要只看 input 文件名。
- SCF 是否真正收敛，磁矩是否稳定；必要时比较不同初始磁矩的解。
- Hubbard occupations / occupation matrix 是否与预期局域轨道一致；若 output 中无法判断，应转入 PDOS/projwfc 等轨道审阅，而不是直接解释物性。
- 总能、能隙、局域磁矩、价态或下游性质对 U 的敏感性是否记录为扫描或误差来源。
- 普通 DFT 与 DFT+U 的比较是否只用于说明 Hamiltonian 改变后的趋势；不要把二者当作同一数值层级直接混排。

## 常见风险

- U 值无来源，或把不同 projector/赝势/泛函下的 U 值直接搬用。
- 只为了调 band gap 选 U，没有说明局域轨道、价态、磁性或实验/理论约束。
- 开 DFT+U 后沿用普通 DFT 的 cutoff、k 点、smearing、磁性初值和下游 scratch。
- 旧 `Hubbard_U(i)` 输入迁移到现代 `HUBBARD` card 时 species 序号与 label-manifold 对不上。
- 使用的赝势不支持所选 projector 或缺少所需原子轨道信息。
- 同一元素拆成多个 `ATOMIC_SPECIES` 标签后，只给其中一类加 U，却在记录中写成“对元素加 U”。
- 把未收敛的亚稳磁性解当作充分 ground-state 证据。
- 忽略 DFT+U 对 relax、phonon、bands、DOS/PDOS、Bader/charge analysis 等下游可比性的影响。

## 与基础 workflow 的关系

- 与 SCF：DFT+U 首先是新的 `pw.x` ground-state 定义。必须重新建立 DFT+U SCF 的 convergence/output review，不能只继承普通 DFT 的 `JOB DONE`。
- 与 relax：若结构优化使用 DFT+U，后续 SCF/bands/DOS 应使用同一 Hubbard 定义；若只在固定结构上开 DFT+U，应记录结构来自普通 DFT 还是 DFT+U relax。
- 与 bands/DOS/PDOS：后处理读取的 charge density、wavefunction 和 eigenvalues 必须来自同一 `prefix/outdir` 链。普通 DFT bands 与 DFT+U bands 只能作为不同 Hamiltonian 的对照。
- 与 phonon/DFPT：DFT+U 改变 ground state 和 response 前提；进入 phonon 前应先确认当前 QE 版本、projector、收敛性和下游程序支持范围。
- 与记录规范：每次 DFT+U 计算至少记录 QE 版本、输入语法、projector、U/J/V 来源、赝势来源、磁性初值、output review 和 PASS/WARN/BLOCK 判断。

## Physics judgement 回查

- DFT+U 的 U/J/V、projector、Hubbard manifold、赝势、磁态和 occupation matrix 是同一个模型 provenance；缺任一项都不应标记为 `PASS`。
- U 不是 universal element parameter，也不应作为单纯 gap-fitting 参数。
- 结论边界见 [physics-judgement/u-value-provenance-and-boundary.md](../../physics-judgement/u-value-provenance-and-boundary.md)。

## 资料来源

- QE `pw.x` input reference, `HUBBARD` card and `&SYSTEM` fields: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE `Hubbard_input.pdf`, new DFT+Hubbard input notes: <https://www.quantum-espresso.org/Doc/user_guide_PDF/Hubbard_input.pdf>
- QE documentation index: <https://www.quantum-espresso.org/documentation/>
