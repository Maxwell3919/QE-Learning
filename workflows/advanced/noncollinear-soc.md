# Noncollinear SOC workflow

## 页面定位

- 对应学习路线：[learn/09-feature-expansion-map.md](../../learn/09-feature-expansion-map.md)
- 上游依赖：基础 SCF、收敛性、结构和赝势记录
- 下游用途：高级物性分析或专题 workflow
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 适用问题

本页用于判断何时从基础 collinear SCF 进入 noncollinear / spin-orbit coupling (SOC) 计算，以及进入后必须额外审阅哪些输入和输出边界。

适合的问题类型：

- 体系的自旋方向不能用单一 up/down 轴描述，需要非共线磁矩。
- 目标性质显式依赖 SOC，例如 SOC splitting、磁各向异性、自旋纹理或含重元素体系的能带细节。
- 需要比较同一结构在 scalar-relativistic、noncollinear 和 SOC Hamiltonian 下的结果差异。
- 后续 bands、DOS、投影分析或专题 workflow 需要读取带 spinor/SOC 信息的基态数据。

不适合在本页展开的内容：

- SOC 的完整理论教材。
- 具体材料的 input/output 案例。
- 未完成基础 SCF、赝势和收敛性记录时直接解释 SOC 小能量差。

## 进入该 workflow 前必须已经掌握什么

进入前至少应能独立完成并审阅：

- [SCF workflow](../ground-state/scf.md)：能判断 `pw.x` SCF 是否真正自洽，而不是只看 `JOB DONE`。
- [Spin polarization workflow](spin-polarization.md)：能区分初始磁矩、总磁矩、绝对磁矩和局域磁矩。
- [Magnetism, SOC and DFT+U](../../theory-minimum/magnetism-soc-dftu.md)：理解磁性、SOC 和 DFT+U 都会改变 Hamiltonian、对称性和下游可比性。
- 赝势记录：能说明每个元素使用的 UPF 文件来源、泛函族、价电子设置、是否支持目标 relativistic / SOC 用途。
- 收敛性记录：cutoff、k 点、occupation/smearing 已针对基础计算收敛；进入 SOC 后仍需要重新审阅目标性质是否对这些参数敏感。

## 关键 QE 程序或输入字段

| 字段或程序 | 所属位置 | 用途 | 边界说明 |
|---|---|---|---|
| `pw.x` | 程序 | 执行 noncollinear / SOC 的 SCF、NSCF 或 bands 计算 | 本页只覆盖 `pw.x` 层面的进入条件和 output review。 |
| `noncolin` | `&SYSTEM` | 启用非共线自旋计算 | SOC 计算通常需要与非共线框架一起考虑；一旦启用，不能再按普通 collinear up/down 输出理解磁性。 |
| `lspinorb` | `&SYSTEM` | 允许 noncollinear code 使用带 spin-orbit 信息的赝势 | 不是单独的绘图开关；它改变 Hamiltonian，后续 bands/DOS/投影数据链需要保持一致。 |
| `starting_magnetization(i)` | `&SYSTEM` | 为每个原子种类设置初始自旋极化 | QE 文档说明它用于 LSDA 或 non-collinear/spin-orbit 计算；在较新 QE 中可用磁矩值或每价电子磁化比例表达。非零磁性态必须显式给出非零初始磁化或使用相应约束，否则可能停留在零磁化态。 |
| `angle1(i)` | `&SYSTEM` | 设置第 `i` 个原子种类初始磁化方向与 z 轴夹角 | 只用于 noncollinear 计算，单位为度；它描述初始方向，不等同于最终物理磁矩方向。 |
| `angle2(i)` | `&SYSTEM` | 设置初始磁化在 x-y 平面投影与 x 轴夹角 | 只用于 noncollinear 计算，单位为度；必须和 `starting_magnetization(i)` 一起记录。 |
| `starting_spin_angle` | `&SYSTEM` | 控制 SOC 磁性情形下初始波函数的 spin-angle 初始化方式 | 这是初始波函数边界，不是最终态判据；是否使用应记录原因。 |
| `constrained_magnetization` / `fixed_magnetization` / `lambda` | `&SYSTEM` | 需要约束非共线磁矩时使用 | 不属于本页基础入口；若使用，应另建专题记录并说明约束物理含义。 |
| fully relativistic pseudopotential | `ATOMIC_SPECIES` / UPF 文件 | SOC 的赝势前提 | 必须逐元素核对赝势是否包含 SOC 所需信息；同一材料的 scalar-relativistic 与 fully relativistic 结果不能混作同一数据集。 |

`starting_magnetization` 和角度字段的边界：

- `starting_magnetization(i)` 是按 atomic type 设置的初始条件；若同一元素的不同子晶格需要不同初始磁矩，应在 `ATOMIC_SPECIES` 中拆成不同种类并分别记录。
- `angle1(i)`、`angle2(i)` 只定义初始磁矩方向；最终磁矩、能量差和可比性必须从 output 与重复计算中判断。
- 对 noncollinear/SOC，所有原子种类都从零 `starting_magnetization` 出发会引入时间反演对称的零磁化边界；如果目标是非零磁性态，这通常不是合适入口。
- NSCF 或 bands 阶段可以保留与 SCF 一致的初始磁矩字段用于记录一致性，但物理解释应以生成基态的 SCF 设置为准。

SOC 赝势要求：

- `lspinorb=.true.` 只有在对应赝势支持 spin-orbit 时才有意义；不能只看元素名相同。
- 每个元素都要记录 UPF 文件名、来源、版本或库、交换关联泛函族、赝势类型以及是否为 fully relativistic / SOC-capable。
- 赝势、cutoff、k 点、occupation 策略改变后，原有非 SOC 收敛结论只能作为参考，不能自动继承为 SOC 结论。

## output review 要点

SOC/noncollinear output review 应在基础 SCF review 之外增加以下字段：

```markdown
## Noncollinear / SOC Output Review

- pw.x calculation type:
- QE version:
- `noncolin` setting echoed:
- `lspinorb` setting echoed:
- Pseudopotentials loaded:
- SOC-capable / fully relativistic PP evidence:
- Initial magnetic species and `starting_magnetization(i)`:
- Initial magnetic angles `angle1(i)` / `angle2(i)`:
- Time-reversal / symmetry behavior:
- SCF convergence:
- Final magnetic moment information:
- Energy comparison target:
- Numerical tolerance for energy difference:
- Downstream data compatibility:
- PASS / WARN / BLOCK:
- Reason:
```

审阅重点：

- output 开头的计算类型、QE version、temporary directory、prefix/outdir 是否和记录一致。
- output 是否回显 noncollinear、spin-orbit 或 spinor 相关设置；若没有明确证据，不把结果标记为 SOC 数据。
- `ATOMIC_SPECIES` 加载的 UPF 文件是否就是记录中的 fully relativistic / SOC-capable 赝势。
- SCF 是否达到自洽收敛；`JOB DONE` 仍然只表示程序结束。
- 初始磁矩和角度是否与本次物理假设一致；不同初始方向得到的能量差是否大于数值误差。
- 对称性、时间反演、k 点约化是否与目标磁性态一致；零初始磁化导致的零磁化边界要单独标记。
- 后续 bands/DOS/投影是否使用同一 `prefix/outdir` 数据链和同一 SOC/noncollinear 设置。

## 常见风险

- 使用 scalar-relativistic 赝势却打开 `lspinorb`，或只记录元素名不记录具体 UPF 文件。
- 把 `noncolin` / `lspinorb` 当作后处理开关，忽略它们会改变基态 Hamiltonian 和后续数据链。
- 未记录 `starting_magnetization(i)`、`angle1(i)`、`angle2(i)`，导致磁性态不可复现。
- 目标是非零磁性态，却让所有种类从零初始磁化开始并默认接受零磁化结果。
- 直接比较 SOC 与非 SOC 总能、能带或磁各向异性能量，但没有保持结构、赝势族、cutoff、k 点、smearing 和收敛阈值可比。
- 小能量差没有通过更严格 k 点、cutoff、SCF 阈值和重复初始方向检查，就被解释为物理结论。
- bands / DOS 沿用非 SOC 的 scratch 或输入设置，造成图像来自不同 Hamiltonian。
- 把初始角度当作最终磁矩方向，或把 total/absolute/local magnetic moment 混为一谈。

## 与基础 workflow 的关系

- SCF 是入口：noncollinear/SOC 首先仍然是 `pw.x` ground-state workflow，必须满足基础 SCF 的输入、收敛和 output review 要求。
- Spin polarization 是最近上游：进入 SOC 前，应先能解释 collinear 自旋极化计算中的初始磁矩、最终磁矩和不同初始态。
- 赝势和收敛性是硬边界：SOC 改变赝势要求和 Hamiltonian，基础非 SOC 的 cutoff/k 点结论需要重新标记为“参考”或“已在 SOC 下复查”。
- 下游 bands/DOS/PDOS 必须沿用 SOC/noncollinear 的基态数据链；不能用非 SOC SCF 作为 SOC bands 的隐含前提。
- phonon、DFPT 或更复杂响应计算若叠加 SOC，应另查对应程序文档；本页不把 `pw.x` 的 SOC 边界外推成所有后处理程序的通用保证。
- PASS / WARN / BLOCK 应按基础 workflow 继续执行：只要赝势能力、初始磁性边界、SCF 收敛或下游一致性无法确认，就不能标为 PASS。

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE pseudopotentials page: <https://www.quantum-espresso.org/pseudopotentials/>
- 本仓库：[SCF workflow](../ground-state/scf.md)
- 本仓库：[Spin polarization workflow](spin-polarization.md)
- 本仓库：[Magnetism, SOC and DFT+U](../../theory-minimum/magnetism-soc-dftu.md)
