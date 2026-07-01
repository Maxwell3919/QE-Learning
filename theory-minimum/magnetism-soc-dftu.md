# Magnetism, SOC and DFT+U

## 本页解决什么问题

本页说明在 QE 中遇到磁性、SOC 和 DFT+U 时，最低需要理解哪些 input/output 边界。它们都会改变 Hamiltonian、对称性、赝势要求、occupation 行为和下游解释方式，因此属于模型选择和结果解释边界，不是自动精度提升按钮。

本页不写完整自旋 DFT、非共线磁性或 Hubbard model 推导，也不给任何材料无关的 U 值、磁矩或 SOC 重要性判断。它只帮助读者回答：本次 QE 计算是否真的进入了预期的 spin / SOC / DFT+U 模型，output 中是否有足够证据允许进入 bands、DOS、PDOS、phonon 或高级 workflow。

## 最低掌握深度

- `nspin=2` 是 collinear spin-polarized calculation 的常见入口；`starting_magnetization(i)` 是初始条件，不是最终磁矩。
- `noncolin` 进入 noncollinear spin 框架；`lspinorb` 在该框架下启用 spin-orbit coupling。SOC 需要逐元素核对 fully relativistic / SOC-capable pseudopotential。
- DFT+U 作用于指定 localized manifold；U 依赖 projector、赝势、functional、结构、局域环境和求取策略，不是元素常数。
- 磁性、SOC 和 DFT+U 会影响 symmetry、irreducible k-points、band degeneracy、DOS/PDOS 标签、phonon/electronic downstream consistency。
- 任何 spin/SOC/DFT+U 结论都必须保留 input 设置、output 回显、warning、磁矩或 occupation 证据，以及同一 `prefix/outdir` 数据链。

## QE 中的对应对象

| 对象 | QE 位置 | 判断意义 | output 证据 |
|---|---|---|---|
| `nspin` | `&SYSTEM` | collinear spin 分支入口 | output 中 spin-polarized 设置、spin-resolved occupation 或 magnetization |
| `starting_magnetization(i)` | `&SYSTEM` | 每个 atomic type 的初始磁化 | input echo、初始/最终磁矩差异 |
| `tot_magnetization` / `constrained_magnetization` | `&SYSTEM` | 总磁矩或约束磁性边界 | constraint 信息、最终 total / absolute magnetization |
| `noncolin` | `&SYSTEM` | noncollinear spinor 计算 | noncollinear / spinor 设置回显，磁矩分量输出 |
| `lspinorb` | `&SYSTEM` | SOC Hamiltonian 入口 | SOC 设置回显、对称性变化、spinor bands |
| fully relativistic pseudopotential | `ATOMIC_SPECIES` / UPF | SOC 的赝势前提 | loaded UPF、relativistic 信息、warning |
| `HUBBARD` card | input card | Hubbard manifold、projector、U/J/V 参数 | Hubbard setup、occupation matrix、warning |
| legacy Hubbard fields | `&SYSTEM` | 旧输入或旧记录的 DFT+U 边界 | QE version、旧语法回显 |
| `input_dft` | `&SYSTEM` | functional 与 DFT+U/SOC 模型耦合 | XC 回显、PP/XC warning |
| `occupations/smearing/degauss` | `&SYSTEM` | 磁性金属、小带隙或 SOC 后占据边界 | Fermi energy、occupation scheme、band occupation |

相关 workflow：[SCF](../workflows/ground-state/scf.md)、[NSCF](../workflows/ground-state/nscf.md)、[bands](../workflows/electronic/bands.md)、[DOS](../workflows/electronic/dos.md)、[PDOS](../workflows/electronic/pdos.md)。相关标准：[output review checklist](../standards/output-review-checklist.md)。

## 核心概念

Collinear spin 把电子态分成自旋通道；noncollinear spin 用 spinor 描述更一般的磁矩方向。SOC 把自旋和轨道自由度耦合，会改变能带简并、对称性、磁各向异性和某些电子结构解释。DFT+U 则在选定局域子空间中修正 semilocal functional 对局域电子的部分误差。

这些设置不是“在同一计算上多看一个图”。它们改变的是计算模型本身。改变模型后，结构、SCF、NSCF、bands、DOS、PDOS 以及 phonon 等下游都需要保持同一数据链，不能把无 SOC、无 U 或不同 functional 的结果混入同一判断。

## 对 input 的影响

- `starting_magnetization(i)` 应按 atomic type 记录；若同一元素有不同磁性角色，应使用不同 species label 并说明原因。
- 磁性金属或小带隙体系中，`occupations/smearing/degauss` 与磁矩、Fermi energy、DOS、phonon 可能耦合，不能自动沿用非磁计算的判断。
- SOC 计算需要 `noncolin`、`lspinorb` 和 fully relativistic pseudopotential 同时满足；只写 `lspinorb` 不足以证明 SOC 数据链可信。
- DFT+U 输入必须记录 `HUBBARD` card 或当前 QE 版本对应语法、projector/manifold、参数单位、参数来源和作用的 species label。
- `input_dft`、赝势、Hubbard 参数和结构状态必须一致；换其中任一项都应视为模型变化。

## 对 output review 的影响

| output 证据 | 支持的判断 | 不能证明什么 |
|---|---|---|
| spin / noncollinear / SOC 设置回显 | 计算进入预期 Hamiltonian 分支 | 磁态或 SOC 结论已经物理可信 |
| total / absolute magnetization | 总磁矩和绝对磁化可复查 | 局域磁矩、磁有序或构型竞争已充分判断 |
| atomic magnetization / projections | 局域投影趋势可审阅 | 投影磁矩是未经限定的完整磁矩定义 |
| symmetry / irreducible k-points | 模型改变后的对称性可复查 | 原无 SOC k-path 和简并解释仍可沿用 |
| loaded pseudopotentials | 实际 UPF 文件可追踪 | PP 支持 SOC 或 DFT+U projector 已充分验证 |
| Hubbard setup / occupation matrix | U 作用流形和占据输出可追踪 | U 来源、迁移性或强关联模型已充分成立 |
| warnings | projector、symmetry、occupation 或文件链风险 | warning 可忽略 |

## 常见误区

- 把 `starting_magnetization(i)` 当作最终磁矩。
- 只记录“开了磁性”，不记录 `nspin`、初始磁化、最终 total/absolute/local magnetization。
- 打开 SOC 后沿用无 SOC 的 k-path、band degeneracy 和 DOS/PDOS 解释。
- 使用不支持 SOC 的 pseudopotential，却解释 SOC band splitting。
- U 值不说明来源，或把 U 当作元素常数。
- DFT+U 被写成普适修正，或只为匹配 band gap 调整 U。
- hybrid、DFT+U、SOC 混用后不说明模型边界。
- 磁矩、occupation 或 Hubbard occupation 未收敛，却进入 bands/DOS 结论。

## PASS / WARN / BLOCK 关联

| 状态 | 理论依据 | 下游准入 |
|---|---|---|
| PASS | spin/SOC/DFT+U 设置在 input 和 output 中一致；PP、functional、occupation、磁矩或 Hubbard 证据可追踪；warning 已解释 | 可进入同一模型下的 SCF 下游、bands、DOS、PDOS 或进一步敏感性分析 |
| WARN | 模型数据链可追踪，但磁性构型、U 来源、SOC 重要性或 occupation 敏感性尚未充分对照 | 只允许趋势性或探索性解释，记录中必须写明边界 |
| BLOCK | PP 不支持目标 SOC；U 无来源；spin/SOC/DFT+U 与下游文件混用；磁性或 Hubbard 相关 warning 未解释 | 不允许进入定量电子结构、磁性、gap、phonon 或高阶方法结论 |

## 下游影响

- `bands/DOS/PDOS`：spin/SOC/DFT+U 改变 band splitting、gap、Fermi level、投影通道和图例含义。
- `phonon/DFPT`：磁态、functional、U 和 SOC 可能改变结构和力常数；下游必须使用同一模型。
- `Fermi surface`：SOC 与磁性会改变 band crossing 和 degeneracy。
- `Wannier/EPC/GW/BSE`：这些高级 workflow 必须继承同一 spin/SOC/DFT+U 数据链，并额外验证模型边界。

## 与 physics-judgement 的边界

本页只解释最低概念和 QE input/output 对应关系。结果可信度判断回到：

- [Functional choice and sensitivity](../physics-judgement/functional-choice-and-sensitivity.md)
- [Pseudopotential transferability and functional consistency](../physics-judgement/pseudopotential-transferability-and-functional-consistency.md)
- [U value provenance and boundary](../physics-judgement/u-value-provenance-and-boundary.md)
- [SOC and symmetry boundary](../physics-judgement/soc-and-symmetry-boundary.md)
- [Band gap problem and delocalization](../physics-judgement/band-gap-problem-and-delocalization.md)

## 来源与边界

- Stable: `nspin`、`starting_magnetization`、`noncolin`、`lspinorb`、`input_dft`、`occupations` 和 `HUBBARD` card 的字段含义以 QE `INPUT_PW` 为准。
- Version-sensitive: `HUBBARD` card、projector 选项、旧 DFT+U 输入和 `hp.x` 工作流需查当前 QE 文档。
- Boundary: 本页不判断某类 functional、U、SOC 或磁性构型对某个物理体系一定更优。

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE Hubbard input guide: <https://www.quantum-espresso.org/Doc/user_guide_PDF/Hubbard_input.pdf>
- QE pseudopotentials page: <https://www.quantum-espresso.org/pseudopotentials/>
- Cococcioni and de Gironcoli, Linear response approach to DFT+U.
- Timrov, Marzari and Cococcioni, DFPT Hubbard parameter papers.
- 本仓库：[canonical literature](../references/canonical-literature.md)、[output review checklist](../standards/output-review-checklist.md)
