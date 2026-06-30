# Relaxation Loop

## 能力目标

理解结构优化在 QE 中的角色，能区分 fixed-cell `relax`、variable-cell `vc-relax` 和低维体系的晶胞约束边界；能从 output 中读取 final coordinates、forces、stress、pressure 和 convergence message，并判断结构是否允许进入 static SCF、bands、DOS 或 phonon。

## 完成判据

- 能区分 `relax` 和 `vc-relax` 的优化对象、输入控制和 output 审阅重点。
- 能说明 force、stress 和 pressure 的关系，以及它们分别用于哪些准入判断。
- 能说明 fixed-cell relax 与 cell relaxation 的边界：什么时候只优化原子位置，什么时候需要优化晶胞。
- 能说明为什么 `relax` / `vc-relax` 后需要用最终结构重跑 static SCF。
- 能从 output 中找到 final coordinates、forces、stress、pressure 和 convergence message。
- 能说明低维体系为什么不能无脑 `vc-relax`，并知道本页只做边界说明，不展开结构构建细节。
- 能说明结构操作学习将作为独立项目展开；本仓库只说明 QE 对结构输入的要求。

## `relax` 与 `vc-relax`

`relax` 是 fixed-cell relaxation：晶胞参数保持 input 中给定的值，只优化原子位置。它的核心问题是：在这个晶胞内，原子受到的力是否已经足够小，最终坐标是否可以作为后续 static SCF 的结构输入。

`vc-relax` 是 variable-cell relaxation：原子位置和晶胞参数都会被优化。它的核心问题不只是原子力是否收敛，还包括 stress / pressure 是否达到目标，final `CELL_PARAMETERS` 是否符合物理问题和维度约束。

| 项目 | `relax` | `vc-relax` |
|---|---|---|
| 优化对象 | 原子位置 | 原子位置和晶胞 |
| 主要控制 | `calculation = 'relax'`、`&IONS`、`forc_conv_thr` | `calculation = 'vc-relax'`、`&IONS`、`&CELL`、`press`、`press_conv_thr`、`cell_dofree` |
| output 必看 | final coordinates、final forces、convergence message、最后一次 SCF 状态 | final coordinates、final cell、final forces、stress tensor、pressure、convergence message、最后一次 SCF 状态 |
| 下游结构来源 | output 末尾更新后的 `ATOMIC_POSITIONS` | output 末尾更新后的 `CELL_PARAMETERS` 和 `ATOMIC_POSITIONS` |
| 常见误用 | 只看到 `JOB DONE` 就接受结构 | 未审阅 stress / pressure，或低维体系放开非物理晶胞方向 |

## Force、stress 与 pressure

Force 描述原子位置方向上的能量梯度，用来判断原子是否在当前晶胞内达到平衡。fixed-cell `relax` 的主要结构准入证据是 final forces 和 total force 是否达到目标阈值。

Stress 描述能量对晶胞形变的响应，是晶胞优化和外压判断的证据。`vc-relax` 中不能只看 force；如果 final forces 收敛但 stress / pressure 不合理，结构仍然不能直接作为“晶胞已优化”的结果。

Pressure 可以理解为 stress tensor 的标量摘要，常用于和 `press` / `press_conv_thr` 对照。实际审阅时不要只记录 pressure 一个数，还要检查 stress tensor、cell update 和 final cell 是否符合物理约束。

简化判断：

- `relax`：重点看 final coordinates 和 forces；stress 可作为诊断信息，但不是 fixed-cell 晶胞优化完成的证据。
- `vc-relax`：同时看 final coordinates、forces、stress、pressure 和 final cell。
- phonon / response workflow 前，残余 force 和晶胞状态都需要更严格审阅。

## Fixed-cell relax 与 cell relaxation 的边界

选择 `relax` 还是 `vc-relax`，应按本轮计算是否允许晶胞变化决定。

用 `relax` 的典型边界：

- 晶胞来自可信来源，本轮只想消除原子内部坐标的残余力。
- 下游比较要求晶胞保持一致，例如只比较同一固定晶胞下的不同内部坐标。
- 低维体系的真空层、表面法向或非周期方向不希望被优化器改变。

用 `vc-relax` 的典型边界：

- 晶格常数、晶胞形状或体积本身就是优化目标。
- 需要在给定目标压力下寻找晶胞平衡状态。
- 下游任务需要优化后的 cell 和 coordinates 作为统一上游。

边界判断必须记录到 output review：本轮是否允许 cell 改变，允许哪些自由度改变，为什么这些自由度和物理问题一致。

## 为什么优化后还要 static SCF

`relax` / `vc-relax` 的目标是搜索结构，过程中会经历多个 ionic / cell step，每一步内部都有电子 SCF。这个过程的 output 可以提供最终结构，但不应直接等同于后续 bands、DOS、PDOS、phonon 所需的清晰 ground-state 数据边。

优化完成后应使用 final coordinates，或 final cell + final coordinates，重新写一个 static `scf` input，并用已经确定的 cutoff、k mesh、occupations 和 pseudopotentials 重跑。这样做有四个作用：

- 把“结构搜索过程”和“最终结构上的 ground-state 计算”分开记录。
- 确认最终结构上的电子 SCF 独立收敛。
- 给 bands、DOS、phonon 和后处理提供清晰、可复查的上游文件。
- 在 `vc-relax` 后复查 final cell 对 k 点密度、cutoff policy 和后续参数的影响。

因此，本阶段的下游准入应以 final structure 已提取、final static SCF 已通过 output review 为准，而不只看 `relax` 或 `vc-relax` 文件是否结束。

## Output 中看什么

阅读 `pw.x` relaxation output 时，至少按下面顺序审阅：

1. 计算类型：确认 output 中显示的是 `relax` 还是 `vc-relax`，不要把普通 `scf` output 当成结构优化结果。
2. 每个 ionic step 的电子 SCF：检查最后一次电子 SCF 是否正常收敛，是否有需要解释的 warning。
3. Final coordinates：在 output 末尾找到 updated / final `ATOMIC_POSITIONS`，记录坐标单位和来源，不要只复制 input 坐标。
4. Forces：记录 final forces、total force 或对应收敛信息，判断是否达到 `forc_conv_thr` 或本项目设定阈值。
5. Stress / pressure：`vc-relax` 必须记录 stress tensor、pressure、目标压力和压力收敛状态；`relax` 中如有 stress 输出，只作为固定晶胞诊断信息。
6. Final cell：`vc-relax` 必须记录 final `CELL_PARAMETERS`，并检查 cell 自由度是否符合 `cell_dofree` 和维度边界。
7. Convergence message：区分正常收敛、达到步数上限、优化器失败、电子 SCF 异常和普通 `JOB DONE`。`JOB DONE` 只说明程序正常结束，不等于结构可用。
8. 下游准入：写出 PASS / WARN / BLOCK，明确允许进入哪些 workflow，以及是否必须先做更严格 relax、`vc-relax` 或 static SCF。

## 可验证输出

- `pw.relax.<system>.in/out` 或 `pw.vc-relax.<system>.in/out` 的阅读笔记。
- final structure 的来源说明。
- final static SCF 的准入判断。
- force / stress / pressure 摘要和 PASS / WARN / BLOCK 记录。
- 低维体系或受限晶胞的 cell constraint 说明。

## Output review 最小清单

- 计算类型是 `relax` 还是 `vc-relax`。
- final `ATOMIC_POSITIONS` 已从 output 末尾提取，坐标单位明确。
- final forces 和 total force 已记录，并与阈值对照。
- `vc-relax` 的 final `CELL_PARAMETERS`、stress tensor、pressure 和 `cell_dofree` 已记录。
- convergence message 已分类；没有把 `JOB DONE` 单独当作充分完成证据。
- final static SCF 已计划或完成，并写明是否允许进入 bands、DOS、phonon 或后处理。
- 低维体系只记录晶胞自由度边界和真空方向约束，不在本页展开结构构建。

## 低维体系边界

低维体系的关键风险是把非物理方向也交给 `vc-relax`。例如含真空方向的模型，如果无约束放开所有 cell 自由度，优化器可能改变真空层或破坏原本的维度设定。

本页只要求掌握 workflow 边界：

- 先判断哪些方向是物理晶格自由度，哪些方向只是建模边界或真空。
- 如果使用 `vc-relax`，必须记录 `cell_dofree` 或等价的 cell constraint。
- 结构构建、slab、真空层、超胞、对称性和低维模型细节不在本仓库展开，放到独立 Structure-Learning 项目。

## 推荐阅读

- [workflows/ground-state/relax.md](../workflows/ground-state/relax.md)
- [workflows/ground-state/vc-relax.md](../workflows/ground-state/vc-relax.md)
- [theory-minimum/force-stress-relaxation.md](../theory-minimum/force-stress-relaxation.md)
- [standards/output-review-checklist.md](../standards/output-review-checklist.md)

## 后续进入哪个 workflow

完成后进入 final static SCF；static SCF 通过 output review 后，再进入 bands、DOS/PDOS、phonon 或后处理 workflow。

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- [workflows/ground-state/relax.md](../workflows/ground-state/relax.md)
- [workflows/ground-state/vc-relax.md](../workflows/ground-state/vc-relax.md)
- [theory-minimum/force-stress-relaxation.md](../theory-minimum/force-stress-relaxation.md)
- [standards/output-review-checklist.md](../standards/output-review-checklist.md)
