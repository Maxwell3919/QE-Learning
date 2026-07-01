# Pseudopotential Transferability and Functional Consistency

## 本页解决什么问题

本页用于判断 pseudopotential 是否足以支撑当前 DFT/QE 结论。PP 不只是文件附件；它定义了 electron-ion interaction、valence space、relativistic treatment、projector 和 cutoff 边界。一次 SCF 正常结束不能证明 PP transferability，也不能证明不同 PP 结果可以直接混写。

## 典型 output 现象

| output 现象 | 可能含义 | 回查动作 |
|---|---|---|
| loaded PP 与记录文件名或来源不一致 | `pseudo_dir` 或库版本混用 | 核对 `ATOMIC_SPECIES`、output header 和 record |
| PP/XC mismatch warning | `input_dft` 与 UPF functional 不一致 | 进入模型边界审查，不能直接忽略 |
| cutoff 改变后 force/stress/phonon 仍敏感 | PP 类型或 target observable 对 basis 敏感 | 重做目标量收敛，记录 `ecutwfc/ecutrho` |
| SOC 计算中 PP relativistic 信息不清 | SOC 前提不足 | 回查 UPF、QE PP 页面和 output warning |
| DFT+U / PDOS projector 解释异常 | PP valence/projector 与目标 manifold 不匹配 | 回查 UPF、Hubbard card、projwfc output |

## 可能原因

- 数值误差：`ecutwfc/ecutrho`、FFT grid、kmesh 或 response convergence 对目标量不足。
- 模型误差：PP 的 valence、semicore、core-valence separation 或 relativistic treatment 改变 Hamiltonian。
- 赝势误差：PP transferability 对目标结构、压力、磁性、phonon、Born charge 或 SOC 不足。
- 边界条件误差：surface、low-dimensional、charged 或 high-pressure 情境可能放大 PP 差异。
- 后处理误差：PDOS、Wannier、DFT+U projector 或 charge density 解释忽略 PP 定义。
- workflow 传播误差：SCF 用 PP A，relax/bands/phonon 或图件使用 PP B。
- 真实物理效应：semicore、relativistic 或 valence choice 的物理差异可能真实影响结论，但需要对照记录。

## 必查 QE input/output

| 对象 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `ATOMIC_SPECIES` | `pw.x` | species label、mass、UPF 文件名 | 只记录元素，不记录实际 UPF |
| `pseudo_dir` | `&CONTROL` | PP 文件来源目录 | 不同机器或 run 读取不同库 |
| UPF metadata | PP 文件 / output | XC、valence、PP type、relativistic 信息 | 不查 valence / SOC capability |
| `input_dft` | `&SYSTEM` | 与 PP functional consistency | 用不一致 functional 且无说明 |
| `ecutwfc/ecutrho` | `&SYSTEM` | PP 类型和 density cutoff 边界 | 沿用旧 PP cutoff |
| `noncolin/lspinorb` | `pw.x` | SOC 与 fully relativistic PP 关系 | scalar-relativistic PP 支撑 SOC 解释 |
| `HUBBARD` / `projwfc.x` | `pw.x` / `projwfc.x` | localized manifold / projector 解释 | PP 换了但 U 或 PDOS 解释沿用 |

## 判断规则

- PP source、library/version、UPF 文件名、XC、valence、PP type、relativistic capability 和 recommended cutoff 都应进入 calculation record。
- recommended cutoff 只能作为起点；能否进入下游取决于目标 observable 的 convergence，而不是 PP 文件建议值。
- PP/functional consistency 是模型边界。若 `input_dft` 与 UPF functional 不一致，必须说明目的、重新审阅收敛，并在结论中保留边界。
- 换 PP 后至少要重审 SCF、cutoff、kmesh、force/stress 和目标 observable；高风险目标包括 phonon、Born charge、magnetism、SOC、work function、PDOS、DFT+U 和 Wannier。
- PP transferability 不能由 `JOB DONE`、单次 SCF convergence 或单个 total energy 数字证明。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | PP source、functional consistency、valence、type、relativistic capability、cutoff convergence 和 target observable 边界清楚；output loaded PP 与 record 一致 | 允许进入对应 workflow 和带边界的物理解释 |
| WARN | PP 可追踪，但 transferability 对照或高风险目标量收敛不足 | 只允许探索性结论或趋势判断 |
| BLOCK | PP 来源不明；PP/functional 不一致无说明；SOC 不兼容；不同 PP 结果混用；换 PP 后未重做关键收敛 | 不允许进入下游性质结论 |

## 不能做出的过度结论

- 不能把 PP 当作可随意替换的技术附件。
- 不能把 PP recommended cutoff 写成收敛结论。
- 不能把同一元素的不同 UPF 视为同一模型。
- 不能把 SCF convergence 当成 transferability 证明。
- 不能在 PP 不支持目标 relativistic 或 projector 边界时解释 SOC、DFT+U、PDOS 或 Wannier 结论。

## 下游影响

- `SCF/cutoff`：PP 决定 valence space、basis demand 和 density cutoff。
- `relax/vc-relax`：force/stress 和 cell response 对 PP 与 cutoff 敏感。
- `bands/DOS/PDOS`：gap、band ordering、projection labels 和 local states 依赖 PP。
- `phonon/DFPT`：force constants、Born charge、dielectric tensor 和 LO-TO splitting 对 PP 质量敏感。
- `SOC/DFT+U/Wannier/work function`：relativistic capability、projector 和 valence treatment 直接影响解释边界。

## 与 theory-minimum / workflows / standards 的关系

- 理论回查：[Pseudopotentials](../theory-minimum/pseudopotentials.md)、[Plane-wave cutoff](../theory-minimum/plane-wave-cutoff.md)、[Magnetism, SOC and DFT+U](../theory-minimum/magnetism-soc-dftu.md)
- Workflow 回查：[SCF](../workflows/ground-state/scf.md)、[cutoff convergence](../workflows/ground-state/cutoff-convergence.md)、[PDOS](../workflows/electronic/pdos.md)、[phonon dispersion](../workflows/phonon/phonon-dispersion-dfpt.md)
- 相关判断：[Functional choice and sensitivity](functional-choice-and-sensitivity.md)、[SOC and symmetry boundary](soc-and-symmetry-boundary.md)、[U value provenance](u-value-provenance-and-boundary.md)
- 记录规范：[calculation record template](../standards/calculation-record-template.md)、[output review checklist](../standards/output-review-checklist.md)

## 来源与结论强度

| 结论 | 强度 | 来源边界 |
|---|---|---|
| PP 定义 electron-ion interaction model 和 valence boundary | Strong | pseudopotential canonical literature |
| PP/functional consistency 与 transferability 需要记录 | Strong | QE docs、reproducibility literature |
| recommended cutoff 不是收敛结论 | Boundary | workflow convergence requirement |
| UPF metadata、SOC compatibility 和字段行为 | Version-sensitive | 当前 QE PP / `INPUT_PW` 文档 |

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE pseudopotentials page: <https://www.quantum-espresso.org/pseudopotentials/>
- QE UPF format notes: <https://pseudopotentials.quantum-espresso.org/home/unified-pseudopotential-format>
- Vanderbilt, Ultrasoft Pseudopotentials.
- Blochl, Projector Augmented-Wave Method.
- Lejaeghere et al., Reproducibility in density functional theory calculations of solids.
- 本仓库：[canonical literature](../references/canonical-literature.md)、[source index](../references/source-index.md)
