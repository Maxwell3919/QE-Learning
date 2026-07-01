# Dielectric tensor and Born effective charge

## 本页解决什么问题

本页解释 dielectric tensor 和 Born effective charge 在 QE/DFPT 中是什么，以及为什么它们只能在满足物理和文件链条件时用于 LO-TO splitting、non-analytic correction、IR/Raman 和极性声子解释。它帮助用户读懂 `ph.x epsil=.true.` 的 output，而不是把张量数值当作普通 ground-state scalar。

## 最低掌握深度

最低需要知道：

- Dielectric tensor 描述系统对宏观电场的线性响应。
- Born effective charge 描述宏观极化对原子位移的动态响应，不是静态离子价。
- 它们是 response quantities，对 functional、PP、gap、structure、k mesh、cutoff 和 response convergence 敏感。
- QE 中 `epsil=.true.` 是 Gamma、非金属电场响应分支；输出后仍需审阅 tensor symmetry、warning 和文件链。

## 物理图像

极性材料中的晶格振动和宏观电场会相互耦合。某些原子位移会改变晶体极化，极化变化又会产生长程库仑场；这个电场反过来改变长波极限下的 dynamical matrix。因此，在极性绝缘体中，只看短程 force constants 不能完整描述 Gamma 附近的纵向光学模式。

Dielectric tensor 描述电子和离子系统对宏观电场的线性响应。Born effective charge 描述某个原子沿某方向位移时，宏观极化如何变化。它是动态响应量，包含成键和电子重排信息，因此不能解释为静态离子价。一个形式电荷相同的原子，在不同结构、泛函、赝势或局域环境下可以有不同的 Born effective charge。

LO-TO splitting 来自长程电场的 non-analytic contribution：同样接近 Gamma 点，纵向和横向扰动的电场边界不同，频率可以分裂。QE 中 `epsil=.true.`、Born charge、dielectric tensor、`dynmat.x` 或 `matdyn.x` 的 non-analytic 设置共同决定这个解释是否成立。

## 最低数学结构

可以把 Born effective charge 看作极化对原子位移的导数，把 dielectric tensor 看作极化对电场的导数：

```text
Z*  ~  dP / du
epsilon  ~  dP / dE
```

这些导数来自线性响应。它们进入长波极限 dynamical matrix 的 non-analytic 项，影响 polar phonon、LO-TO splitting 和 IR activity。因为响应量依赖 band gap、occupation、pseudopotential、functional 和 structure，output review 需要检查的不只是张量数值，还包括适用条件、收敛、对称性和文件链。

## QE 中的对应对象

| 对象 | QE 程序 | 判断意义 | output 证据 |
|---|---|---|---|
| `epsil=.true.` | `ph.x` | 请求 dielectric tensor 和 Born effective charge | dielectric/Born tensor 段落 |
| `trans` | `ph.x` | 声子 perturbations 与 response 计算 | perturbation convergence |
| `tr2_ph` | `ph.x` | response 自洽阈值 | electric-field / atomic perturbation 收敛 |
| `fildyn` | `ph.x` / `dynmat.x` | Gamma dynamical matrix 与 response 文件链 | 后处理读取记录 |
| `asr` / `zasr` | `dynmat.x` / `q2r.x` / `matdyn.x` | acoustic / charge neutrality 相关约束 | ASR 设置、frequency changes |
| non-analytic correction | `dynmat.x` / `matdyn.x` | LO-TO splitting 和方向相关 Gamma limit | direction、tensor 来源、frequency splitting |

相关 workflow：[dielectric/Born workflow](../workflows/phonon/dielectric-born-effective-charge.md)、[IR/Raman](../workflows/phonon/ir-raman.md)、[Gamma phonon](../workflows/phonon/gamma-phonon.md)。

## 核心概念

在极性绝缘体中，长波极限下原子位移会产生宏观电场，电场又反馈到 dynamical matrix。Dielectric tensor 和 Born effective charge 提供了这一 non-analytic contribution 的输入，因此会影响 LO-TO splitting 和 IR activity。由于它们来自线性响应，output 中张量完整性、对称性、收敛和适用条件比单个数值更重要。

## 对 input 的影响

- `epsil=.true.` 应只在 Gamma response 目标明确且体系条件可解释时开启。
- 上游 occupation、smearing、band gap 边界必须记录；金属或部分占据体系不应按普通绝缘体解释。
- `fildyn`、`flfrc`、response tensor 和 non-analytic correction 必须来自同一结构/SCF/PP/XC 数据链。
- 如果用于 IR/Raman 或 LO-TO splitting，应记录方向、ASR/zasr、tensor 来源和后处理程序。

## 对 output review 的影响

| output 证据 | 支持的判断 | 不能证明什么 |
|---|---|---|
| `epsil` 分支执行 | QE 请求并进入电场 response | 物理条件自动满足 |
| dielectric tensor | 介电响应张量输出完整 | 张量已对所有数值参数收敛 |
| Born charge tensors | 每个原子动态有效电荷输出 | 等于静态价态 |
| tensor symmetry | 与结构对称性是否一致 | 异常一定是物理效应 |
| perturbation convergence | response 方程完成 | functional/PP/model error 已消除 |
| warnings | 金属、symmetry、收敛或电场问题 | warning 可忽略 |
| non-analytic chain | LO-TO 文件链可追踪 | splitting 可无边界解释 |

## 常见误区

- 把 Born charge 当作静态离子价。
- tensor 输出后不检查对称性。
- response 未收敛就做 IR/LO-TO 解释。
- polar correction 文件链不清。
- 在金属或 occupation 边界不清时解释 dielectric/Born 张量。
- q-grid dispersion 中误以为每个 q point 都有 `epsil` response。
- LO-TO splitting 不记录方向和 ASR 设置。

## PASS / WARN / BLOCK 关联

| 状态 | 理论依据 | 下游准入 |
|---|---|---|
| PASS | 绝缘体条件明确；`epsil` 分支完整；electric-field/atomic perturbations 收敛；tensor symmetry 和文件链可复查 | 可进入 LO-TO、IR/Raman 和 polar phonon 分析 |
| WARN | 小带隙、tensor 敏感、symmetry 或 ASR 记录不足 | 可探索或做敏感性测试，不应给定量响应结论 |
| BLOCK | 金属/部分占据条件下解释张量；response 未收敛；张量缺失；non-analytic 文件链混用 | 不允许进入 IR/LO-TO/Born charge 结论 |

## 下游影响

- `Gamma phonon`：`epsil` 分支与 Gamma modes 一起审阅。
- `phonon dispersion`：polar correction 依赖张量来源和文件链一致。
- `IR/Raman`：IR activity 和 LO-TO 解释依赖 Born charge/dielectric tensor。
- `workflows advanced`：EPC、spectroscopy 或 polar materials 分析会继承这些 response 边界。

## 与 physics-judgement 的边界

本页只说明 response tensor 的最低 QE 使用。以下问题转到：

- [DFPT response and polar materials](../physics-judgement/10-dfpt-response-and-polar-materials.md)
- [phonons, soft modes and dynamical stability](../physics-judgement/09-phonons-soft-modes-and-dynamical-stability.md)
- [kmesh/smearing sensitivity](../physics-judgement/kmesh-smearing-sensitivity.md)

## 来源与边界

- Stable: `epsil`、`trans`、`tr2_ph`、`fildyn` 以 QE `INPUT_PH` 为准；`asr/zasr` 和 non-analytic 后处理以对应 QE `INPUT_*` 为准。
- Stable: Born charge、dielectric tensor 与 IFC 的 DFPT 理论边界来自 Gonze and Lee 以及 DFPT canonical literature。
- Boundary: 本页不把 response tensor 与实验介电常数或谱强度无条件等同。

## 资料来源

- QE INPUT_PH reference: <https://www.quantum-espresso.org/Doc/INPUT_PH.html>
- QE INPUT_DYNMAT reference: <https://www.quantum-espresso.org/Doc/INPUT_DYNMAT.html>
- QE INPUT_Q2R reference: <https://www.quantum-espresso.org/Doc/INPUT_Q2R.html>
- QE INPUT_MATDYN reference: <https://www.quantum-espresso.org/Doc/INPUT_MATDYN.html>
- QE PHonon user guide: <https://www.quantum-espresso.org/Doc/ph_user_guide/>
- Gonze and Lee, Dynamical matrices, Born effective charges, dielectric permittivity tensors and interatomic force constants.
- 本仓库：[dielectric/Born workflow](../workflows/phonon/dielectric-born-effective-charge.md)
