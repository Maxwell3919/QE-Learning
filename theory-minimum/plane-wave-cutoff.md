# Plane-wave cutoff

## QE 中对应的问题

平面波 cutoff 回答的是：在当前结构、赝势、交换关联泛函、k mesh、smearing 和目标 workflow 下，平面波基组与电荷密度网格是否已经足够，使输出量可以进入下游判断。

在 QE 的 `pw.x` 中，cutoff 至少分成两层：

- `ecutwfc` 控制波函数平面波基组大小。它直接影响 total energy、能带、本征态、力、应力、DFPT 线性响应以及几乎所有依赖波函数的后处理。
- `ecutrho` 控制电荷密度和势的平面波 cutoff，也决定相关 FFT 网格。它对力、应力、USPP/PAW 增强电荷、GGA/vacuum 体系和 phonon 可能比 total energy 更敏感。

最低判断应关注目标量对 `ecutwfc` / `ecutrho` 的变化是否已经小到足以支撑当前 workflow 的结论，而不能只看某个 cutoff 数值是否常见。因此 cutoff policy 必须绑定到：

- 赝势文件和赝势类型；
- 目标性质，例如 total energy、force、stress、band gap、phonon frequency；
- 后续 workflow，例如 SCF、relax、vc-relax、bands、DOS、phonon/DFPT；
- 记录中的 PASS / WARN / BLOCK 准入结论。

换赝势、换 valence 配置、换泛函、换结构维度或把任务从 SCF 推进到 relax/phonon 时，旧 cutoff policy 不能自动继承，应重新做与目标量匹配的收敛检查。

## 输入字段表

| 字段 | 所属位置 | 作用 | 需要成组固定/记录的对象 | output 中如何复核 | 常见误用 |
|---|---|---|---|---|---|
| `ecutwfc` | `&SYSTEM` / `pw.x` | 波函数 kinetic-energy cutoff，单位 Ry | `ATOMIC_SPECIES`、`K_POINTS`、`occupations`、`conv_thr`、结构、泛函 | `kinetic-energy cutoff`；每个 scan 点的 SCF 收敛状态 | 只跑单点；只看教程值；换赝势后沿用旧值 |
| `ecutrho` | `&SYSTEM` / `pw.x` | 电荷密度和势的 kinetic-energy cutoff，单位 Ry | `ecutwfc`、赝势类型、FFT grid、是否输出 force/stress、是否进入 DFPT | `charge density cutoff`；FFT grid；force/stress/phonon 随 scan 的变化 | 认为省略即一定可靠；USPP/PAW 下不单独检查 |
| `ATOMIC_SPECIES` | card / `pw.x` | 指定元素标签、质量和 UPF 赝势文件；赝势决定 cutoff 需求和 valence 空间 | pseudo 文件名、来源、版本、XC、NC/USPP/PAW 类型 | output 中 loaded pseudopotential、pseudo type、valence 信息和 warning | 混用不同来源赝势；替换 pseudo 后不重扫 cutoff |
| `K_POINTS` | card / `pw.x` | cutoff scan 中必须保持不变的 Brillouin zone 采样 | k mesh、shift、对称性设置、金属 smearing | output 中 k 点数量和 symmetry 信息 | 同时改 cutoff 和 k mesh，导致无法归因 |
| `occupations` / `smearing` / `degauss` | `&SYSTEM` / `pw.x` | 决定金属/绝缘体系的占据处理；会影响 total energy 和力的可比性 | cutoff scan 全程固定 | output 中 occupations、smearing、Fermi energy 等 | cutoff 未收敛和 smearing 未收敛混在一起 |
| `conv_thr` | `&ELECTRONS` / `pw.x` | SCF 电子收敛阈值；cutoff scan 的每个点都必须先电子收敛 | diagonalization、mixing、起始电荷策略 | `convergence has been achieved` 或未收敛信息 | 用未收敛 SCF 的目标量判断 cutoff |
| `tprnfor` | `&CONTROL` / `pw.x` | 要在 SCF 输出中审阅力时需要打开；relax 类计算会自动涉及力 | 是否以 force 作为 cutoff 目标量 | `Forces acting on atoms` | 声称 force 收敛但 output 未打印力 |
| `tstress` | `&CONTROL` / `pw.x` | 要在 SCF 输出中审阅应力时需要打开；vc-relax 类任务需要应力 | 是否以 stress / pressure 作为 cutoff 目标量 | `total stress`、pressure | vc-relax 或弹性相关任务只用 total energy 收敛 |
| `nr1/nr2/nr3`、`nr1s/nr2s/nr3s` | `&SYSTEM` / `pw.x` | 手动指定 FFT 网格；通常由 cutoff 自动决定 | 只有需要复现实验或诊断网格问题时才显式记录 | output 中 FFT grid / smooth grid | 手动网格与 cutoff policy 不一致，造成不可复现 |

## output review

### 基本一致性

- output 中报告的 `kinetic-energy cutoff` 与 `charge density cutoff` 是否与 input 或预期默认行为一致。
- output 中加载的 UPF 文件、元素标签、valence 信息、pseudo type 和 warning 是否与记录一致。
- 每个 scan 点是否 SCF 收敛；未收敛点不能用于 cutoff PASS。
- cutoff scan 是否只改变 `ecutwfc` / `ecutrho` 相关变量，结构、k mesh、smearing、泛函、赝势和电子收敛设置是否保持一致。

### 目标量判断

- total energy：适合做第一层 sanity check，但不能单独证明 force、stress 或 phonon 已可用。
- force：relax、NEB、phonon 前结构准备等任务必须检查；如果 total energy 平滑但最大力或原子受力方向明显漂移，应继续收敛或调整策略。
- stress：vc-relax、晶格常数、压力、弹性相关任务必须检查；`ecutrho` 和 FFT grid 对应力噪声尤其需要关注。
- phonon / DFPT：phonon frequency、声学支、虚频判断和 Raman/高阶响应等下游任务对 cutoff 更敏感；不能只继承普通 SCF 的 total-energy cutoff。
- 后处理：bands、DOS、charge density、Bader-like 分析、Wannier 初始数据等需要使用与上游 SCF policy 一致的 cutoff 和 pseudo 记录。

### PASS / WARN / BLOCK

- PASS：所有 scan 点可复核，SCF 已收敛，目标量变化满足本 workflow 自定准则，且准则写入记录。
- WARN：total energy 看似稳定，但 force、stress、phonon 或目标后处理尚未检查；可用于探索，不应用于定稿或进入严格下游。
- BLOCK：SCF 未收敛、输出字段缺失、赝势或 k mesh 在 scan 中改变、目标量仍有系统漂移，或换赝势后没有重做 cutoff scan。

## 常见错误

| 错误 | 为什么危险 | 应如何修正 |
|---|---|---|
| 直接照搬教程或 pseudo 文件推荐值 | 推荐值只能作为起点，不能替代目标 workflow 的收敛证据 | 用固定结构、固定 k mesh、固定 pseudo 的 scan 表建立本任务 policy |
| 只看 total energy | total energy 可能比 force、stress、phonon 更早表现平滑 | 根据下游目标把 force、stress 或 phonon frequency 加入 review |
| 忽略 `ecutrho` | 电荷密度、势、FFT grid 和增强电荷会影响力、应力和响应性质 | 对 `ecutwfc` 与 `ecutrho` 分开记录，并说明是否单独 scan 过 |
| 把 NC、USPP、PAW 当成同一种 cutoff 逻辑 | 不同 pseudo 类型的密度/增强电荷需求不同 | 在记录中明确 pseudo type；USPP/PAW 特别复查 `ecutrho` 和 stress/phonon |
| 换赝势后沿用旧 cutoff | pseudo 文件改变了 valence、投影子、augmentation charge 或推荐范围 | 把换 pseudo 视为新数值体系，从 cutoff workflow 重新开始 |
| 同时改变 cutoff 和 k mesh | 目标量变化无法归因 | cutoff scan 中固定 k mesh；k convergence 另做 workflow |
| 用未收敛 SCF 输出做 cutoff 表 | 电子收敛误差会伪装成 cutoff 误差 | 每个点先确认 SCF convergence，再记录目标量 |
| relax 过程中边变结构边判断 cutoff | 结构改变会改变力、应力和能量，无法隔离 cutoff 效应 | 先在固定结构上做 cutoff policy，再进入 relax/vc-relax |
| phonon 出现虚频后只调 displacement 或 q mesh | cutoff、`ecutrho`、结构残余力和 SCF 阈值都可能造成假信号 | 回到 SCF/relax/cutoff 记录，复查 phonon 前置条件 |

## 最低掌握深度

读完本页后，最低应能完成以下判断：

- 能在 `pw.x` input 中指出 `ecutwfc`、`ecutrho`、`ATOMIC_SPECIES`、`K_POINTS`、`conv_thr`、`tprnfor`、`tstress` 的作用和联动关系。
- 能解释 NC、USPP、PAW 对 `ecutrho` 和增强电荷/FFT 网格敏感性的差异，而不是把所有 pseudo 套同一经验比例。
- 能从 output 中确认实际使用的 wavefunction cutoff、charge density cutoff、pseudo type、FFT grid、SCF 收敛和 warning。
- 能根据目标 workflow 选择 review 量：普通 SCF 至少看 total energy；relax 看 force；vc-relax 和压力相关任务看 stress；phonon/DFPT 看频率和响应量。
- 能给出一张 cutoff scan 表，并用目标量变化写出 PASS / WARN / BLOCK，而不是只写“看起来差不多”。
- 能说明换 pseudo、换泛函、换 valence 设置或换下游目标后，为什么必须复查 cutoff policy。

## 对应 workflow

- [Cutoff convergence workflow](../workflows/ground-state/cutoff-convergence.md)
- [SCF workflow](../workflows/ground-state/scf.md)
- [Relax workflow](../workflows/ground-state/relax.md)
- [Phonon dispersion DFPT workflow](../workflows/phonon/phonon-dispersion-dfpt.md)
- [physics-judgement/numerical-vs-model-error.md](../physics-judgement/numerical-vs-model-error.md)
- [physics-judgement/pseudopotential-transferability-and-functional-consistency.md](../physics-judgement/pseudopotential-transferability-and-functional-consistency.md)

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE pseudopotentials page: <https://www.quantum-espresso.org/pseudopotentials/>
