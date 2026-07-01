# Force, stress and relaxation

## QE 中对应的问题

`relax` 和 `vc-relax` 都是在回答同一个准入问题：当前结构是否已经足够接近平衡，能够作为 static SCF、bands、DOS、phonon 或其他响应性质 workflow 的上游结构。

在 QE 中这个判断不能只看 `JOB DONE`。最低限度需要同时确认：

- 离子优化是否真正收敛，残余 force 是否满足 `forc_conv_thr` 及下游目标。
- 如果晶胞也被优化，stress tensor、pressure 和 `press_conv_thr` 是否支持 final cell 可用。
- output 末尾的 final coordinates 和 final cell 是否已被提取到下游输入，而不是继续使用初始结构。
- 最后一次电子 SCF 是否收敛；离子步完成不等于电子态数据已经可直接用于性质计算。

固定晶胞 `relax` 主要审阅 force 和 final coordinates。变晶胞 `vc-relax` 需要同时审阅 force、stress、pressure、final coordinates 和 final cell。低维结构只在这里保留边界提醒：不要无约束放开非物理真空方向；具体 slab、wire、layer 构建规则属于结构构建主题。

## 输入字段

| 字段 | 所属位置 | 用在哪类计算 | 判断作用 | 常见误用 | Output 中如何核对 |
|---|---|---|---|---|---|
| `calculation = 'relax'` | `pw.x` / `&CONTROL` | 固定晶胞结构优化 | 只优化原子坐标，不优化晶胞 | 需要优化晶胞时误用 `relax`，或用 `scf` 代替结构优化 | output 应显示 ionic minimization，并在末尾给出 updated atomic positions |
| `calculation = 'vc-relax'` | `pw.x` / `&CONTROL` | 变晶胞结构优化 | 同时优化原子坐标和晶胞 | 不需要变晶胞却放开 cell；低维结构未限制真空方向 | output 应显示 variable-cell optimization，并给出 final cell 与 final coordinates |
| `forc_conv_thr` | `pw.x` / `&CONTROL` | `relax`、`vc-relax` | 离子优化的力收敛阈值 | 阈值只按默认值或粗略测试设置，未匹配 phonon/响应性质等下游要求 | 检查 final forces、total force、convergence message 是否支持结构准入 |
| `tprnfor` | `pw.x` / `&CONTROL` | 需要审阅 force 的计算 | 控制是否打印 forces | 未打印 force，导致无法复查结构是否可用 | output 中应能看到每个原子的 force 和 total force |
| `tstress` | `pw.x` / `&CONTROL` | 需要审阅 stress 的计算，尤其 `vc-relax` | 控制是否计算并打印 stress | `vc-relax` 或 stress 判断缺少 stress tensor | output 中应能看到 stress tensor 和 pressure |
| `ion_dynamics` | `pw.x` / `&IONS` | `relax`、`vc-relax` | 选择离子优化算法 | 优化失败后只机械续算，不看步长、力和能量历史 | output 中检查 optimizer 状态、收敛或失败信息 |
| `cell_dynamics` | `pw.x` / `&CELL` | `vc-relax` | 选择晶胞优化算法 | 只设置离子优化，忽略 cell optimizer；或失败后不分析 cell 变化 | output 中检查 cell update、final cell 和 variable-cell 收敛信息 |
| `press` | `pw.x` / `&CELL` | `vc-relax` | 目标压力 | 未记录目标压力，导致 pressure 偏离时无法判断是否符合任务 | output 中比较 reported pressure 与目标压力 |
| `press_conv_thr` | `pw.x` / `&CELL` | `vc-relax` | 压力收敛阈值 | 只看力收敛，不看 pressure/stress 是否达到晶胞准入要求 | 检查 final pressure、stress tensor 和 cell convergence message |
| `cell_dofree` | `pw.x` / `&CELL` | `vc-relax` | 限制允许变化的晶胞自由度 | 低维结构把真空方向也放开；需要保持对称/形状时未约束 | 对照 final cell 与初始 cell，确认变化方向符合物理问题 |
| `ATOMIC_POSITIONS` | card | `relax`、`vc-relax` | 输入初始坐标；output 末尾给出 final coordinates | 下游仍使用 input 坐标 | 从 output 末尾复制 updated/final coordinates 到 static SCF 输入 |
| `CELL_PARAMETERS` | card | `vc-relax` 必查；`ibrav = 0` 时显式给出 | 输入初始晶胞；`vc-relax` output 给出 final cell | 只更新坐标不更新晶胞；cell 改变后不复查 k 点密度 | 从 output 末尾复制 final cell，并重新检查 static SCF 的 k mesh 与 cutoff policy |

## output review

### `relax` 必查输出

| 输出对象 | 判断问题 | PASS 倾向 | WARN / BLOCK 倾向 |
|---|---|---|---|
| 计算类型 | 是否真的运行了固定晶胞优化 | output 明确是 `relax` / ionic minimization | 实际是 `scf`、中途退出或 input/output 不匹配 |
| 电子 SCF | 每个离子步，尤其最后一步是否收敛 | 最后一次 electronic convergence 正常 | 最后 SCF 未收敛、达到迭代上限或出现数值异常 |
| Forces / total force | 残余力是否满足结构目标 | final forces 和 total force 支持 `forc_conv_thr` 判断 | 只看到能量降低但 force 未收敛；缺少 force 输出 |
| Optimizer message | 离子优化是否收敛 | 明确出现 ionic convergence 或 equivalent completion message | optimizer failure、步长异常、达到最大步数但未收敛 |
| Final coordinates | 下游是否使用最终结构 | output 末尾 final/updated `ATOMIC_POSITIONS` 已记录 | 仍使用 input 坐标进入 static SCF |

### `vc-relax` 必查输出

| 输出对象 | 判断问题 | PASS 倾向 | WARN / BLOCK 倾向 |
|---|---|---|---|
| 计算类型 | 是否真的运行了变晶胞优化 | output 明确是 `vc-relax` / variable-cell optimization | 实际没有放开 cell，或 cell 设置与任务不一致 |
| Forces | 原子位置是否收敛 | final force 满足 `forc_conv_thr` 和下游目标 | 力未收敛却只因为 pressure 接近目标而接受结构 |
| Stress tensor | 晶胞形状和体积的残余应力是否可接受 | stress tensor 与任务目标一致，无明显异常分量 | stress 大、方向异常，或未打印 stress |
| Pressure | 目标压力是否达到 | final pressure 与 `press` / `press_conv_thr` 判断一致 | pressure 未收敛，或目标压力未记录 |
| Cell update history | 晶胞变化是否物理合理 | cell 变化方向与 `cell_dofree` 和体系维度一致 | 低维结构真空方向被压缩/拉伸；cell 振荡或突变 |
| Final cell + coordinates | 下游 static SCF 是否可复现 | final `CELL_PARAMETERS` 和 final `ATOMIC_POSITIONS` 都已提取 | 只更新坐标、不更新 cell；或 cell 改变后未复查 k mesh |

### Static SCF 准入判断

`relax` / `vc-relax` 的 output 不是 bands、DOS、phonon 的最终电子结构边界。结构优化通过后，应使用 final coordinates；对 `vc-relax` 还要使用 final cell，重新运行 static SCF。最低准入判断是：

- `relax`：final coordinates 已提取；force 审阅通过；最后 SCF 收敛；固定 cell 的使用符合任务目标。
- `vc-relax`：final coordinates 与 final cell 都已提取；force、stress、pressure 审阅通过；cell 自由度设置合理；cell 改变后 k mesh 和 cutoff policy 已复查。
- 若只达到“可以继续优化”的程度，应标记为中间结构，不能写成 phonon、bands、DOS 的最终上游结构。

## 常见错误

| 错误 | 为什么危险 | 最低修正 |
|---|---|---|
| 看到 `JOB DONE` 就接受结构 | `JOB DONE` 只说明程序正常结束，不等于 force/stress 达到目标 | 回到 output 检查 convergence message、final forces、stress/pressure 和 final structure |
| 用 relax 输出直接做性质解释 | 优化过程中的电子态不一定是清晰的最终 static SCF 数据边界 | 用 final structure 重跑 static SCF |
| 只保存 input 坐标 | 下游实际使用的是未优化结构 | 从 output 末尾提取 final `ATOMIC_POSITIONS` |
| `vc-relax` 后只更新坐标 | final cell 被丢失，结构与 output 判断不一致 | 同时提取 final `CELL_PARAMETERS` 和 final `ATOMIC_POSITIONS` |
| 只看 force，不看 stress/pressure | 对变晶胞结构，晶胞可能仍未收敛 | `vc-relax` 同时审阅 force、stress tensor、pressure 和 cell convergence |
| 低维结构无约束 `vc-relax` | 真空方向可能被非物理压缩或拉伸 | 明确 `cell_dofree` 边界；低维构型细节转入结构构建规范 |
| 优化失败后直接续算 | 可能延续错误优化路径或数值不稳定 | 先读优化历史、最大力、cell 变化、SCF 异常和 warning，再决定重启策略 |
| force/stress 阈值与下游不匹配 | 粗糙结构会污染 phonon 和响应性质 | 按下游 workflow 重新设置 `forc_conv_thr`、`press_conv_thr` 与 SCF 收敛策略 |

## 最低掌握深度

读完本页后，最低应能做到：

- 看到 `relax` output，能说明它是否只是可继续优化，还是可作为 static SCF 的输入结构。
- 看到 `vc-relax` output，能同时检查 final coordinates、final cell、force、stress、pressure 和 cell 自由度。
- 能解释 `forc_conv_thr` 与 `press_conv_thr` 分别控制什么，为什么二者不能互相替代。
- 能知道 `cell_dynamics` 只属于变晶胞优化问题，不能把离子优化收敛等同于晶胞收敛。
- 能把 final coordinates/cell 写入新的 static SCF 输入，并在记录中说明 PASS / WARN / BLOCK 理由。
- 能拒绝把未充分优化、缺少 final structure、缺少 force/stress 证据的结果送入 phonon 或响应性质 workflow。

## 对应 workflow

- [workflows/ground-state/relax.md](../workflows/ground-state/relax.md)
- [workflows/ground-state/vc-relax.md](../workflows/ground-state/vc-relax.md)
- [workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- [Relax workflow](../workflows/ground-state/relax.md)
- [VC-relax workflow](../workflows/ground-state/vc-relax.md)
