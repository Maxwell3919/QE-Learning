# SOC and Symmetry Boundary

## 本页解决什么问题

本页用于判断 SOC / noncollinear 计算是否足以支撑 band splitting、band gap、magnetic anisotropy、spin texture、degeneracy 或拓扑相关解释。SOC 是 relativistic model choice，不是普通后处理开关；启用 SOC 后，pseudopotential、symmetry、k-path、spin setting、bands、DOS、PDOS 和下游数据链都需要重新审阅。

## 典型 output 现象

| output 现象 | 可能含义 | 回查动作 |
|---|---|---|
| output 回显 `noncolin` / spinor / SOC 设置 | 计算进入 noncollinear/SOC 分支 | 核对 PP、symmetry、磁性设置和下游数据链 |
| irreducible k-points 或 symmetry operation 改变 | SOC 或磁构型改变可用对称性 | 重新审阅 k-path 和 degeneracy |
| band degeneracy lifted 或 gap 改变 | SOC / symmetry / magnetic model 影响 eigenvalues | 与 non-SOC 结果分开记录，检查 energy zero |
| magnetization 分量复杂或不稳定 | noncollinear 态、初态或 occupation 敏感 | 回查 starting magnetization、SCF convergence、smearing |
| warning 涉及 pseudopotential、spin-orbit、symmetry | 模型前提可能不满足 | 进入 `WARN/BLOCK` 审阅 |

## 可能原因

- 数值误差：kmesh、smearing、SCF threshold、`nbnd` 或 mixing 对 SOC bands/DOS 不足。
- 模型误差：SOC、magnetism、DFT+U 或 hybrid 共同改变 Hamiltonian。
- 赝势误差：未使用 fully relativistic / SOC-capable pseudopotential，或 PP 与 functional 不一致。
- 边界条件误差：surface、low-dimensional 或 broken inversion 情境下 symmetry 和 degeneracy 解释复杂。
- 后处理误差：SOC/non-SOC bands、DOS、PDOS 或 Wannier files 混用。
- workflow 传播误差：SCF 是 SOC，bands/DOS 却读取非 SOC `prefix/outdir`，或反向混用。
- 真实物理效应：SOC splitting、magnetic anisotropy、spin texture 或 band inversion 可能真实存在，但需要额外证据。

## 必查 QE input/output

| 对象 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| fully relativistic PP | `ATOMIC_SPECIES` / UPF | SOC 计算前提 | 只看元素名，不看 UPF capability |
| `noncolin` | `pw.x` | 启用 noncollinear spinor 框架 | 只改 bands input，不重做 SCF |
| `lspinorb` | `pw.x` | 启用 spin-orbit coupling | 当作后处理绘图选项 |
| `starting_magnetization` / constraints | `pw.x` | 初始磁构型和约束 | 初始磁矩当最终磁性 |
| symmetry settings | `pw.x` | 影响 irreducible k-points、degeneracy、path 解释 | SOC 后沿用旧 symmetry 判断 |
| `K_POINTS` path / mesh | `pw.x` | SOC 后的 bands/DOS 数据链 | non-SOC k-path 解释不重审 |
| `projwfc.x` spinor output | `projwfc.x` | SOC/非共线投影解释 | 仍按普通 up/down PDOS 解释 |

## 判断规则

- SOC 计算必须从 SCF 数据链开始，而不是对已有无 SOC output 做后处理。若要比较 SOC 与 non-SOC 结果，也应分别保存两条完整数据链，并明确 energy zero、Fermi level、结构和 functional 是否一致。
- SOC 需要 fully relativistic / SOC-capable PP；若 PP 证据不清，SOC 结果不能进入定量解释。
- SOC 会改变可用 symmetry、irreducible k-points、band degeneracy 和 k-path 解释；结构或模型改变后应重审路径来源。无 SOC 的 little-group label、degeneracy 或 band crossing 解释不能自动迁移到 SOC Hamiltonian。
- SOC 的重要性不能按元素名或方法标签一刀切。应根据目标 observable、近简并、磁性、能带交叉和相对论 PP 证据判断。
- 拓扑、spin texture 和 magnetic anisotropy 结论需要额外 invariant、有效模型、角度/构型对照或 Wannier 等证据，本页只给边界。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | fully relativistic PP、`noncolin/lspinorb`、spin/magnetic setup、symmetry、k-path、SCF/NSCF/bands/DOS 数据链全部一致且可追踪 | 允许进入带 SOC 边界的 bands/DOS/PDOS 和进一步模型分析 |
| WARN | SOC 数据链可追踪，但 symmetry、磁构型、topology 或 MAE 证据不足 | 只允许定性对照或内部判断 |
| BLOCK | PP 不支持 SOC；SOC/non-SOC 文件混用；未重算下游却比较 SOC 结论；symmetry/k-path 未重审 | 不允许进入 SOC 相关科研结论 |

## 不能做出的过度结论

- 不能把 SOC 当作只需打开 `lspinorb` 的开关。
- 不能在未确认 fully relativistic PP 的情况下解释 SOC splitting。
- 不能沿用无 SOC 的 band degeneracy、k-path 或 PDOS 通道解释。
- 不能把一次 SOC bands 图直接写成拓扑结论。
- 不能把 SOC 对某类体系的重要性写成无条件规则。
- 不能把 SOC/non-SOC 的能带差异归因于自旋轨道效应，而不先排除结构、energy zero、Fermi level、PP 或 k-path 数据链差异。

## 下游影响

- `bands`：SOC 改变 splitting、degeneracy 和可能的 band inversion。
- `DOS/PDOS`：spinor 和 noncollinear 输出改变投影标签和分量解释。
- `Fermi surface`：SOC 和磁性会改变 crossing 和 degeneracy。
- `phonon/electronic downstream`：若目标属性对 SOC 敏感，下游应保持 SOC 数据链一致。
- `Wannier/topology`：需要回对 SOC QE bands，并补充 invariant 或有效模型验证。

## 与 theory-minimum / workflows / standards 的关系

- 理论回查：[Magnetism, SOC and DFT+U](../theory-minimum/magnetism-soc-dftu.md)、[K-points, symmetry and k-path](../theory-minimum/kpoints-symmetry-kpath.md)
- Workflow 回查：[Noncollinear SOC overview](../workflows/advanced/noncollinear-soc.md)、[bands](../workflows/electronic/bands.md)、[PDOS](../workflows/electronic/pdos.md)
- 相关判断：[Pseudopotential transferability](pseudopotential-transferability-and-functional-consistency.md)、[Kohn-Sham eigenvalue boundary](kohn-sham-eigenvalue-boundary.md)
- 记录规范：[output review checklist](../standards/output-review-checklist.md)

## 来源与结论强度

| 结论 | 强度 | 来源边界 |
|---|---|---|
| SOC 改变 Hamiltonian 和 symmetry 审阅 | Strong | relativistic DFT / QE official docs |
| fully relativistic PP 是 SOC 数据链前提 | Strong | QE PP 和 `INPUT_PW` 文档 |
| topology / spin texture 需要额外证据 | Boundary | advanced electronic-structure literature |
| `noncolin/lspinorb` 字段和输出格式 | Version-sensitive | 当前 QE `INPUT_PW` |

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE pseudopotentials page: <https://www.quantum-espresso.org/pseudopotentials/>
- Giannozzi et al., Quantum ESPRESSO code papers.
- 本仓库：[canonical literature](../references/canonical-literature.md)、[source index](../references/source-index.md)
