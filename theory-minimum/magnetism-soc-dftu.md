# Magnetism, SOC and DFT+U

## QE 中对应的问题

磁性、SOC 和 DFT+U 都属于 `pw.x` ground-state 问题的一部分。它们会改变 Hamiltonian、对称性、赝势要求、occupation 行为和下游数据链，因此不能当作普通 SCF 之后的绘图开关。

本页只回答最低边界问题：

- 什么时候 `nspin` 和 `starting_magnetization` 已经进入自旋极化问题。
- 什么时候 `noncolin`、`lspinorb` 和 fully relativistic pseudopotential 已经进入 noncollinear / SOC 问题。
- 什么时候 `HUBBARD` card 已经进入 DFT+U 问题。
- output review 至少要保留哪些磁矩、occupation、Hubbard 和 warning 证据，才能回到 workflow 判断 `PASS / WARN / BLOCK`。

本页不写完整磁性理论，不解释具体磁有序模型，不给具体材料案例，也不提供任何 U 默认值。

## 输入字段表

| 字段或对象 | 所属位置 | 最低含义 | 必查边界 |
|---|---|---|---|
| `nspin` | `&SYSTEM` | 共线自旋极化入口；常见自旋极化 SCF 使用 `nspin=2` | output 是否确认启用了预期自旋设置；不能只因写了 `starting_magnetization` 就认定是可信磁性计算 |
| `starting_magnetization(i)` | `&SYSTEM` | 按 atomic type 给初始自旋极化；用于 LSDA 或 noncollinear / SOC 计算的初始条件 | 初始磁矩不是最终磁矩；若同一元素不同子晶格需不同初值，应拆分 `ATOMIC_SPECIES` 标签并记录 |
| `occupations` / `smearing` / `degauss` | `&SYSTEM` | 决定电子占据处理；磁性金属或近金属结果可能对它敏感 | 开自旋、SOC 或 DFT+U 后不能自动继承非磁计算的 occupation 结论 |
| `noncolin` | `&SYSTEM` | 启用非共线自旋框架 | 一旦启用，不能再按普通 collinear up/down 通道解释所有磁矩输出 |
| `lspinorb` | `&SYSTEM` | 在 noncollinear 框架中启用 spin-orbit coupling | 它改变 Hamiltonian；后续 SCF、NSCF、bands、DOS 和投影分析必须沿用一致数据链 |
| fully relativistic pseudopotential | `ATOMIC_SPECIES` / UPF | SOC 的赝势前提 | 逐元素核对具体 UPF 是否支持 SOC；scalar-relativistic 与 fully relativistic 结果不能混作同一数据集 |
| `HUBBARD` card | `pw.x` input card | 现代 QE 中定义 Hubbard projector、通道和参数的入口 | card 中的 label 必须匹配 `ATOMIC_SPECIES`；轨道流形、projector、参数和单位都要记录 |
| legacy `lda_plus_u` / `Hubbard_U(i)` | `&SYSTEM` | 旧输入风格中的 DFT+U 入口 | 只用于阅读或复现旧输入；新记录应标明 QE 版本和现代/旧语法边界 |
| `prefix` / `outdir` | `&CONTROL` | 连接同一 Hamiltonian 下的 SCF 与后续任务 | 普通 DFT、自旋极化、SOC 和 DFT+U scratch 不应混用到无法追溯 |

## output review

最低 review 不追求完整物理论证，只确认 input 设置是否真的进入了 output，并判断是否允许进入对应 workflow 的下游步骤。

```markdown
## Magnetism / SOC / DFT+U Output Review

- QE program / version:
- Calculation type and prefix/outdir:
- Spin setting reported:
- Starting magnetization echoed:
- Occupation / smearing / degauss:
- SCF convergence:
- Total magnetization:
- Absolute magnetization:
- Atomic magnetic moments / magnetization projections:
- Noncollinear / spinor / SOC setting echoed:
- Pseudopotentials loaded:
- Fully relativistic / SOC-capable PP evidence:
- Hubbard input syntax:
- Hubbard projector and channels:
- Hubbard parameters and units:
- Hubbard occupations / occupation matrix printed:
- Warnings / symmetry messages:
- Compared starting states or Hamiltonians:
- Downstream data compatibility:
- PASS / WARN / BLOCK:
- Reason:
```

判断时至少看：

- `JOB DONE` 只说明程序结束；仍需检查 SCF convergence、warning 和关键物理量是否打印完整。
- output 是否确认了自旋、SOC 或 Hubbard 设置，而不是只看 input 文件。
- `starting_magnetization(i)` 是否被记录，并与最终 total/absolute/local moment 区分。
- total magnetization、absolute magnetization 和 atomic magnetic moments / projections 是否分别记录；三者不能互相替代。
- occupation/smearing/degauss 是否与 Fermi 附近占据、磁矩和后续 DOS/PDOS 目标一致。
- SOC 计算是否能从 output 与赝势记录中确认 loaded UPF 是 fully relativistic / SOC-capable。
- DFT+U output 是否回显 projector、label-manifold、参数数值、单位和 Hubbard occupations / occupation matrix。
- 后续 bands、DOS、PDOS 或响应计算是否读取同一 `prefix/outdir`、赝势、cutoff、k 点、occupation 和 Hamiltonian 设置下的数据。

## 常见错误

- 把自旋极化、SOC 或 DFT+U 当成后处理开关，而不是新的 ground-state 定义。
- 只记录“开了磁性”，没有记录 `nspin`、每个 atomic type 的 `starting_magnetization(i)` 和最终磁矩。
- 把 `starting_magnetization(i)` 当成最终磁矩，或把局域投影磁矩当成未经限定的充分物理结论。
- 把 total magnetization、absolute magnetization 和 atomic magnetic moments 混为一谈。
- 开自旋后沿用非磁 cutoff、k 点、occupation/smearing 结论，不重新判断目标性质是否稳定。
- 使用 scalar-relativistic 赝势却打开 `lspinorb`，或只记录元素名不记录实际 loaded UPF 文件。
- 目标是非零磁性态，却让所有原子类型从零初始磁化开始并直接接受零磁化结果。
- DFT+U 的 U/J/V 数值没有来源，或把不同 projector、赝势、泛函、结构和 QE 版本下的参数直接搬用。
- `HUBBARD` card 的 label、轨道流形或 projector 与 `ATOMIC_SPECIES`、赝势信息不一致。
- 普通 DFT、自旋极化、SOC、DFT+U 的 scratch 或后处理数据链混用，导致图表和 output review 不可追溯。

## 最低掌握深度

读完本页后，只需要达到以下深度：

- 能说清 `nspin=2` 与 `starting_magnetization(i)` 是自旋极化 SCF 的输入边界，且初始磁矩不等于最终磁矩。
- 能从 output review 中分别记录 total magnetization、absolute magnetization 和局域磁矩/投影磁矩。
- 能说清 `noncolin` 和 `lspinorb` 会进入 noncollinear / SOC Hamiltonian，不能作为普通后处理开关。
- 能说明 SOC 需要逐元素核对 fully relativistic / SOC-capable pseudopotential，并保留实际 loaded UPF 证据。
- 能说清 `HUBBARD` card 至少需要 projector、label-manifold、参数数值、单位和来源；不能使用无来源 U 默认值。
- 能检查 occupation/smearing/degauss 在磁性、SOC 或 DFT+U 计算中是否需要重新审阅。
- 能判断一次计算是否只够继续探索，还是足以让对应 workflow 进入 bands、DOS、PDOS 或其他下游任务。

## 对应 workflow

- [workflows/advanced/spin-polarization.md](../workflows/advanced/spin-polarization.md)
- [workflows/advanced/noncollinear-soc.md](../workflows/advanced/noncollinear-soc.md)
- [workflows/advanced/dft-plus-u.md](../workflows/advanced/dft-plus-u.md)

## 资料来源

- QE `pw.x` input reference, `&SYSTEM` fields and `HUBBARD` card: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE `Hubbard_input.pdf`, new DFT+Hubbard input notes: <https://www.quantum-espresso.org/Doc/user_guide_PDF/Hubbard_input.pdf>
- QE pseudopotentials page: <https://www.quantum-espresso.org/pseudopotentials/>
- 本仓库：[Spin polarization workflow](../workflows/advanced/spin-polarization.md)
- 本仓库：[Noncollinear SOC workflow](../workflows/advanced/noncollinear-soc.md)
- 本仓库：[DFT+U workflow](../workflows/advanced/dft-plus-u.md)
