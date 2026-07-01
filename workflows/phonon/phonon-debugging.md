# Phonon debugging and imaginary frequency triage

## 本页解决什么问题

Phonon debugging 用于审阅 `ph.x` 不收敛、Gamma acoustic modes 异常、negative frequency、q-grid 插值异常、Born/dielectric 分支异常和文件链混用等问题。Imaginary frequency triage 是 workflow 审查，不是自动判定真实不稳定。本页目标是把症状归入数值误差、结构问题、边界条件问题、文件链问题或可能的真实动力学不稳定，并决定 PASS / WARN / BLOCK。

## 上游依赖

- 原始 SCF、relax/vc-relax、ph.x、q2r.x、matdyn.x、dynmat.x 的 input/output 保留完整。
- 结构残余力、应力、cell、SCF `conv_thr`、cutoff、k mesh、smearing、q-grid、ASR 和 `prefix/outdir` 均可回查。
- Debugging rerun 使用新的记录和明确的受控变量；不要覆盖原始 output。
- 对 2D、极性、金属、小带隙或磁性体系，应先记录特殊物理边界。

## 计算图

```text
suspicious phonon output
  -> classify symptom
  -> check upstream SCF / structure / q-grid / file chain
  -> one-variable controlled rerun
  -> compare frequencies, modes, warnings and convergence
  -> PASS / WARN / BLOCK decision
```

## Triage 顺序

Phonon debugging 应按证据链从上游到下游推进。若先改 ASR、平滑参数或绘图脚本，容易把症状隐藏起来，却无法判断根因。

1. 结构和 static SCF：确认 phonon 使用的结构、`prefix/outdir`、赝势、functional、cutoff、k mesh、smearing、force/stress 和 final static SCF 都可复查。
2. `ph.x` response：确认每个 q point、irrep 或 perturbation 都收敛，warning 已记录，`tr2_ph` 与目标精度相称。
3. q-grid 和文件链：确认 `fildyn`、dyn 文件集合、`q2r.x` 输入、`flfrc`、`matdyn.x` 输入输出来自同一结构、同一模型和同一 q-grid。
4. ASR 前后对比：记录 ASR 前后的 Gamma acoustic、低频 branch 和可疑 mode 变化。ASR 可用于诊断平移不变性问题，不可作为单独的可信性证明。
5. direct q-point 与 interpolation：若虚频只出现在 `matdyn.x` path 上，应复查附近 direct `ph.x` q point 或 IFC/q-grid，而不是只根据插值图下结论。
6. eigenvector / displacement pattern：定位虚频 mode 的原子位移图像，判断它是否对应合理的集体畸变、边界伪影或数值噪声。
7. controlled rerun：每次只改变一个变量，例如 cutoff、k mesh、smearing、q-grid、ASR 设置或结构阈值，并比较同一 q point、同一 branch、同一单位下的频率和 eigenvector。

## 关键 QE 输入对象

```fortran
&INPUTPH
  prefix = '<debug_system>',
  outdir = '<fresh_scratch_dir>',
  tr2_ph = <controlled_phonon_threshold>,
  ldisp = <same_or_controlled_change>,
  nq1 = <same_or_controlled_change>,
  nq2 = <same_or_controlled_change>,
  nq3 = <same_or_controlled_change>,
  fildyn = '<debug_system>.dyn',
  recover = .true.,
/
```

| 字段 | 程序 | 判断意义 | 联动对象 | output 中如何验证 |
|---|---|---|---|---|
| `conv_thr` | `pw.x` | 上游 SCF 误差 | `tr2_ph`、phonon frequency | SCF estimated accuracy |
| `ecutwfc` / `ecutrho` | `pw.x` | basis 收敛 | force constants、stress、phonon | SCF output 和 convergence record |
| `K_POINTS` / smearing | `pw.x` | 金属和小带隙 phonon 敏感性 | Fermi surface、EPC 下游 | SCF output 中 k points / occupation |
| `forc_conv_thr` / `press_conv_thr` | `pw.x` relax/vc-relax | 结构残余力/应力 | acoustic/soft modes | relax/vc-relax final forces/stress |
| `tr2_ph` | `ph.x` | perturbation 收敛 | q point / irrep | `ph.x` convergence 段落 |
| `nq1/nq2/nq3` | `ph.x` | q-grid 完整性和插值质量 | `q2r.x` IFC | dyn 文件列表和 `q2r.x` output |
| `asr` / `zasr` | `dynmat.x` / `q2r.x` / `matdyn.x` | ASR 诊断 | acoustic modes | ASR 前后频率对比 |
| `epsil` | `ph.x` | polar response 分支 | LO-TO、non-analytic | dielectric/Born output |

## 命令与文件边界

Debugging 不应覆盖原始计算。每次复查只改变一个变量，并使用可区分的 `prefix/outdir`、`fildyn`、`flfrc` 和输出文件名。需要比较原始输出与 controlled rerun 的同一 q point、同一 branch、同一 ASR 设置和同一单位。

## Output review

| 症状 | 从哪里看 | 可能原因 | 回查动作 |
|---|---|---|---|
| `ph.x` 不收敛 | perturbation convergence、warning | SCF 太松、smearing/mixing、metal response、内存或 restart | 回查 SCF、occupation、`tr2_ph`、restart 文件 |
| Gamma acoustic modes 不为零 | `dynmat.x` 或 `matdyn.x` Gamma 频率 | 结构残余力、ASR、cutoff/k mesh、数值噪声 | 比较 ASR 前后，检查 final forces/stress |
| 单个小虚频 | q-position、branch、magnitude | 插值、q-grid、结构边界、近软模 | 复算关键 q point，检查 eigenvector |
| 整条 branch 虚频 | dispersion 文件 | 真实不稳定、错误结构、错误 q-grid 或极性修正缺失 | 回查结构、IFC、ASR、物理模型 |
| direct q-point 与 interpolation 不一致 | `ph.x` dyn、`matdyn.x` frequency | q-grid/IFC 截断、path 插值或文件链问题 | 复查 `q2r.x`、`flfrc`、邻近 q point |
| eigenvector 不连续或难解释 | mode output、visualized displacement | branch mixing、degeneracy、symmetry 或绘图处理问题 | 回查 mode ordering、symmetry、单位和坐标 |
| q2r dyn 缺失 | `q2r.x` output | q-grid 未完成、`fildyn` 前缀错 | 检查 dyn 文件列表和 q-grid 记录 |
| LO-TO 异常 | polar branch、frequency splitting | dielectric/Born 缺失或不适用 | 回查 `epsil` 输出和 non-analytic 设置 |
| 2D Gamma 附近异常 | 低 q 频率、边界设置 | 周期镜像、真空、长程相互作用 | 回查边界条件和低维处理策略 |

## 收敛与可靠性

- 数值噪声：通常表现为 acoustic modes 小偏离、参数敏感、ASR 前后变化明显，需要收敛复查。
- 结构未充分优化：常见于残余力/应力较大、relax 后未做 final static SCF 或 cell 与 q-path 不一致。
- 真实动力学不稳定：应表现为在更严格参数、关键 q point 复算、合理 ASR 和结构审阅后仍稳定存在，并有可解释 eigenvector。
- ASR 可以作为诊断和约束，不是把错误上游变成可信结果的工具。
- 对金属、极性和低维体系，smearing、k mesh、non-analytic correction 和边界条件会放大 phonon 误差。
- 对接近简并的模式，mode ordering 可能随 q 点或后处理改变；此时应比较 mode subspace 和 displacement pattern，而不是只比较表格中的第几个 branch。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | 症状已定位；复查结果显示文件链一致、上游可信、perturbation 收敛；虚频或 acoustic 异常有明确处理和边界 | 可进入对应 phonon workflow 或有限边界的结论 |
| WARN | 可能是数值伪影或边界问题，但仍缺少一个以上敏感性检查；结果只适合诊断 | 可继续 controlled rerun，不应进入最终稳定性、热学或谱学结论 |
| BLOCK | 未定位虚频来源；关键 q point 未收敛；结构/SCF 为 BLOCK；文件链混用；用 ASR 直接消除问题后宣称可信 | 不允许进入图件归档、稳定性声明、EPC、热学或发表结论 |

## 常见误区

- 看到小虚频后跳过 triage，写成相变或真实不稳定。
- 用 ASR 消掉 acoustic 异常后不复查结构和 SCF。
- 对 q-grid 不完整的 dispersion 进行物理解释。
- 不检查 eigenvector 或 displacement pattern。
- 只看 `matdyn.x` 插值曲线，不回查 direct `ph.x` q point 或 IFC 文件链。
- 在金属中忽略 smearing/k mesh 对 phonon 的影响。
- 将 2D 或极性体系的长程相互作用问题当作普通数值噪声。
- Debugging 时同时改变多个变量，导致无法判断原因。

## 下游影响

Phonon debugging 影响 [Gamma phonon workflow](gamma-phonon.md)、[phonon dispersion DFPT workflow](phonon-dispersion-dfpt.md)、[phonon DOS workflow](phonon-dos.md)、[dielectric/Born workflow](dielectric-born-effective-charge.md) 和 IR/Raman。EPC、热学、自由能和结构稳定性声明都不能绕过本页的 WARN/BLOCK 结论。

## 来源与边界

- Stable: `ph.x` q-grid、`tr2_ph`、restart 与 `q2r.x`/`matdyn.x` 文件语义以 QE 官方 input references 为准。
- Boundary: 本页给出 triage 框架，不替代体系特定的物理建模、有限温非谐分析或相变理论。
- Internal standard: 复查记录必须写入 [calculation-record-template.md](../../standards/calculation-record-template.md) 和 [output-review-checklist.md](../../standards/output-review-checklist.md)。

## 资料来源

- QE INPUT_PH reference: <https://www.quantum-espresso.org/Doc/INPUT_PH.html>
- QE INPUT_Q2R reference: <https://www.quantum-espresso.org/Doc/INPUT_Q2R.html>
- QE INPUT_MATDYN reference: <https://www.quantum-espresso.org/Doc/INPUT_MATDYN.html>
- QE PHonon user guide: <https://www.quantum-espresso.org/Doc/ph_user_guide/>
- Baroni et al., phonons and related crystal properties from DFPT, Reviews of Modern Physics.
- 本仓库：[physics-judgement/09-phonons-soft-modes-and-dynamical-stability.md](../../physics-judgement/09-phonons-soft-modes-and-dynamical-stability.md)、[physics-judgement/10-dfpt-response-and-polar-materials.md](../../physics-judgement/10-dfpt-response-and-polar-materials.md)
