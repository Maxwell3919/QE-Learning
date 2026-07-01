# Phonon DOS workflow

## 本页解决什么问题

Phonon DOS 描述声子频率在 q-space 中的态密度分布，用于审阅振动谱整体频率范围、低频区域、热力学输入数据和与 dispersion 的一致性。它来自 q-grid dynamical matrices 经 `q2r.x` 得到的 real-space interatomic force constants，再由 `matdyn.x` 在 DOS q mesh 上采样或积分。Phonon DOS 的对象是 q-space frequency sampling / interpolation，不是单条 high-symmetry q-path；一条 band path 上的频率文件只能说明路径上的分支形状，不能替代 DOS mesh 对整个采样空间的统计。

本页的重点是把 `ph.x -> q2r.x -> matdyn.x` 的文件链、程序边界和 output review 固定下来。DOS 图是否平滑只是绘图和数值处理现象，不是可靠性证明；所有稳定性、热学或谱密度结论都必须回到上游 q-grid、IFC、低频/负频、ASR、单位和 warning 记录。

## 上游依赖

- 已通过 [phonon dispersion DFPT workflow](phonon-dispersion-dfpt.md) 中的 q-grid、dyn 文件、`q2r.x` 和 IFC 审阅。
- `flfrc` 来自当前结构、当前 SCF 和当前 q-grid，不混用旧 force constants。
- 低频、虚频、ASR 和 acoustic branches 已在 dispersion 或 debugging 页面中记录。
- 如果用于热力学或自由能下游，q-grid 和 DOS mesh 需要按目标 observable 做进一步收敛审阅。

## 计算图

```text
final static SCF
  -> ph.x ldisp=.true. on <q_grid>
  -> dynamical matrices: <system>.dyn0, <system>.dyn1, ...
  -> q2r.x -> real-space IFC: <system>.fc
  -> matdyn.x with dos=.true. on <dos_q_mesh>
  -> <system>.freq + phonon DOS file
  -> output review
```

这个计算图中有两个不同的 q-space 层次。`ph.x` 的 q-grid 决定 DFPT dynamical matrices 和 IFC 的来源；`matdyn.x` 的 DOS q mesh 决定在已有 IFC 上如何采样 phonon frequencies 并生成 DOS。两者都应在记录中出现，不能只记录最终 DOS 文件名。

## 关键 QE 输入对象

```fortran
&INPUT
  flfrc = '<system>.fc',
  asr = '<asr_scheme>',
  dos = .true.,
  nk1 = <dos_nk1>,
  nk2 = <dos_nk2>,
  nk3 = <dos_nk3>,
  deltaE = <frequency_step_cm_minus_1>,
  fldos = 'phonon-dos.<system>.dat',
  flfrq = 'phonon-dos.<system>.freq',
/
```

| 字段 | 程序 | 判断意义 | 联动对象 | output 中如何验证 |
|---|---|---|---|---|
| `flfrc` | `matdyn.x` | 读取 real-space IFC | `q2r.x` 输出 | `matdyn.x` header 中读取的文件 |
| `dos` | `matdyn.x` | 启用 phonon DOS 模式 | `nk1/nk2/nk3` | output 是否进入 DOS 计算 |
| `nk1/nk2/nk3` | `matdyn.x` | DOS q mesh | 频率采样、热学下游 | output 中 q mesh 和 DOS 文件 |
| `deltaE` | `matdyn.x` | DOS 频率步长，单位按 QE 文档记录 | `fldos` 曲线分辨率 | DOS 文件频率间隔 |
| `degauss` | `matdyn.x` | DOS broadening，单位为 `cm^-1` | tetrahedra / smoothing 选择 | output 和 DOS 曲线处理记录 |
| `fldos` | `matdyn.x` | phonon DOS 输出文件 | 记录和绘图 | 输出文件路径和列定义 |
| `flfrq` | `matdyn.x` | DOS mesh 上的频率输出 | `fldos`、低频/负频审阅 | 频率文件名和是否覆盖旧文件 |
| `asr` | `matdyn.x` | DOS 计算中的 ASR 处理 | 低频 DOS、acoustic modes | ASR 设置和低频区域变化 |

## 命令与文件边界

```bash
matdyn.x -in matdyn.dos.<system>.in > matdyn.dos.<system>.out
```

`matdyn.x` 的 DOS 模式读取 `flfrc`，不直接重新计算 DFPT perturbations。若 `flfrc` 源自不完整 q-grid、不可信 `q2r.x` 或混用结构，phonon DOS 不能用于下游。建议将 dispersion line-mode 和 DOS-mode 的 `flfrq`、`fldos` 文件名分开，避免旧文件覆盖。

文件链审阅应从上游向下游进行：先确认 `ph.x` 的 `fildyn` 系列和 `dyn0` 属于当前结构与当前 final static SCF；再确认 `q2r.x` 读取了同一组 dynamical matrices 并写出当前 `flfrc`；最后确认 `matdyn.x` DOS run 读取的是这个 `flfrc`，且 `dos=.true.`、DOS q mesh、`deltaE` 或 `degauss`、`fldos` 和 `flfrq` 都在 output 或输入记录中可追踪。只保存最终图片而不保存这些文件名和 output，不足以支持复查。

## Output review

| 检查项 | 从哪里看 | 能证明什么 | 不能证明什么 |
|---|---|---|---|
| IFC 来源 | `matdyn.x` output 中 `flfrc` | DOS 使用了指定 force constants | IFC 来自可信上游 |
| DOS q mesh | `nk1/nk2/nk3` 和 output | DOS 采样网格被记录 | mesh 对目标性质已收敛 |
| frequency range | `fldos` 与 `flfrq` | DOS 覆盖的频率范围 | dispersion 中所有软模已解释 |
| negative frequencies | DOS 文件、频率文件、dispersion review | 低频/负频风险是否进入记录 | 虚频来源已被判定 |
| broadening / step | `deltaE`、绘图记录 | 曲线处理可复查 | 平滑程度代表物理可信 |
| warnings | `matdyn.x` output | 程序边界或输入异常 | 上游没有 warning |

操作审阅建议按固定顺序写入 calculation record。第一步核对 `flfrc` 的来源和生成时间，避免旧 IFC 混入。第二步核对 DOS q mesh 与 `ph.x` q-grid 是两个不同设置，并分别记录。第三步读取 `fldos` 的频率单位、DOS 单位和归一化说明，确认绘图脚本没有静默裁掉负频或低频区域。第四步把 DOS 低频端与 dispersion 的 Gamma 附近、soft mode 位置和 acoustic branch 记录交叉检查。第五步记录 warning、ASR、broadening / `deltaE` 以及任何人工平滑；这些处理只能说明展示方式，不能把 WARN 或 BLOCK 数据提升为 PASS。

## 收敛与可靠性

- Phonon DOS 依赖 `ph.x` q-grid、IFC 截断、`matdyn.x` DOS q mesh、`deltaE` / `degauss` 和积分方式。
- DOS 曲线平滑不说明 phonon dispersion 可信；必须回查 acoustic branches、关键 q-path 和虚频。
- `matdyn.x` 的 DOS mesh 不是 `ph.x` DFPT q-grid；二者都需要记录。
- `matdyn.x` 官方 DOS 模式使用 uniform q-point grid；若体系维度、对称性或材料类别让 tetrahedra / DOS 解释存在边界，应在记录中降级为 WARN，直到有额外证据。
- 若用于热力学自由能、零点能或谱密度比较，低频区域和 imaginary modes 的处理必须明确。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | `flfrc` 来自 PASS 的 q-grid phonon；`matdyn.x` 明确以 `dos=.true.` 读取当前 IFC；DOS q mesh、`deltaE` / `degauss`、ASR、`fldos`、`flfrq`、frequency range、单位和 warning 审阅已记录；无未解释 imaginary modes | 可用于 phonon DOS 图件、初步热学输入和与 dispersion 的交叉审阅 |
| WARN | DOS mesh 或 q-grid 尚未完成目标收敛；低频区域存在可追踪小异常；broadening/step 只适合探索；非立方、低维或特殊对称性场景下 DOS 积分边界尚未说明 | 可用于诊断和敏感性测试，不应给定量热学结论 |
| BLOCK | IFC 不可信或混用；imaginary modes 未解释；`matdyn.x` 读取错误 `flfrc`；把 band q-path 当 DOS；用平滑、裁剪或 broadening 掩盖 dispersion / low-frequency 问题 | 不允许进入热学、稳定性声明、EPC 或发表图件 |

## 常见误区

- 用 band q-path 的频率文件当作 phonon DOS。
- 只看 DOS 曲线平滑，不审阅 `ph.x -> q2r.x -> matdyn.x` 文件链。
- 把 `degauss` 或绘图 smoothing 当成 convergence evidence。
- 负频区域被绘图处理隐藏后仍写成稳定体系。
- 未记录 `deltaE`、DOS q mesh 或 ASR。
- 用未收敛 q-grid 支持热力学或自由能结论。

## 下游影响

Phonon DOS 可作为热力学、自由能、谱密度和低频振动分析的输入。下游最敏感的是低频权重、虚频处理、单位和归一化，因此 WARN 状态的 DOS 只能作为诊断图件，不能支撑定量热学或稳定性声明。它不能替代 [phonon dispersion workflow](phonon-dispersion-dfpt.md) 和 [phonon debugging workflow](phonon-debugging.md)。如果下游是 EPC 或超导，phonon DOS 只能作为辅助检查，不能替代电子-声子矩阵元和 dense k/q 数据链。

## 来源与边界

- Stable: `matdyn.x` 的 `dos`、`nk1/nk2/nk3`、`deltaE`、`degauss`、`fldos`、`flfrq`、`asr` 以 QE `INPUT_MATDYN` 为准；`q2r.x` 的 `fildyn/flfrc/zasr` 以 QE `INPUT_Q2R` 为准。
- Boundary: 本页不规定固定 DOS q mesh；是否足够取决于目标 observable 和低频区域稳定性。
- Internal standard: phonon DOS review 应写入 [output-review-checklist.md](../../standards/output-review-checklist.md) 和 calculation record。

## 资料来源

- QE INPUT_MATDYN reference: <https://www.quantum-espresso.org/Doc/INPUT_MATDYN.html>
- QE INPUT_Q2R reference: <https://www.quantum-espresso.org/Doc/INPUT_Q2R.html>
- QE PHonon user guide: <https://www.quantum-espresso.org/Doc/ph_user_guide/>
- 本仓库：[theory-minimum/dfpt-phonons.md](../../theory-minimum/dfpt-phonons.md)、[physics-judgement/09-phonons-soft-modes-and-dynamical-stability.md](../../physics-judgement/09-phonons-soft-modes-and-dynamical-stability.md)
