# DFT Approximation Map

## 1. 这个主题解决什么判断问题？

本页建立 DFT/QE 误差来源总图。判断一次计算是否可信时，需要区分数值误差、模型误差、物理近似误差和 workflow 传播误差。不同误差的处理方式不同：有些可以通过 cutoff、k mesh 或 q-grid 收敛测试降低；有些来自 exchange-correlation functional、赝势、U、SOC、vdW、边界条件或温度模型，不能靠单纯加大数值参数修复。

本页的目标是让学习者在看到异常 output 时先判断“问题属于哪一类”，再决定是重做收敛测试、换模型、检查上游依赖，还是把结果标记为 `WARN / BLOCK`。

## 2. 物理图像

DFT/QE 的结果来自一串近似。常见层次包括：

- Born-Oppenheimer 图像：电子和原子核运动分离，许多 QE workflow 默认在给定原子结构上求电子基态。
- Kohn-Sham 映射：把相互作用电子问题映射为辅助单粒子方程；Kohn-Sham eigenvalues 可用于 bands/DOS 审阅，但并不自动等同于所有实验激发能。
- Exchange-correlation approximation：LDA、GGA、meta-GGA、hybrid 等泛函决定交换关联能如何近似。
- Pseudopotential approximation：用 UPF 文件定义价电子问题和离子芯相互作用。
- Plane-wave basis 与 cutoff：用有限平面波基组和 FFT 网格表示波函数、密度和势。
- Finite k/q sampling：用有限 k 点和 q 点近似 Brillouin zone 积分与响应函数。
- Periodic boundary condition：用周期重复表示 bulk、slab、低维或带电体系时会引入边界模型。
- Post-processing approximation：bands、DOS、PDOS、potential、ELF、Wannier、EPC、光谱等后处理会继续传播上游误差。

这些近似共同决定 output 的可信度。一次 `JOB DONE` 只说明程序结束，不说明这些近似已经适合目标物理问题。

## 3. DFT/QE 中的近似来源

DFT/QE 的误差可以按处理方式分层：

- 可系统收敛的数值误差：`ecutwfc`、`ecutrho`、FFT grid、k mesh、q-grid、DOS energy grid、SCF/DFPT threshold。
- 需要模型选择的物理误差：exchange-correlation functional、DFT+U、hybrid、SOC、vdW、spin state、finite-temperature model。
- 需要来源审阅的输入误差：pseudopotential 来源、UPF 元信息、结构来源、cell convention、symmetry setting。
- 边界条件误差：supercell size、vacuum、charged background、dipole correction、isolated-boundary treatment。
- workflow 传播误差：上游 SCF、relax、NSCF、DFPT 或 Wannier 数据未通过 output review，却被下游继续读取。

普通 SCF 的总能收敛不能自动放行 phonon、EPC、work function、Born effective charge 或强关联磁性判断。越靠后的 workflow，越需要回查上游近似。

## 4. 相关 QE 输入字段

| 字段 | 程序 | 判断意义 | 常见误用 |
|---|---|---|---|
| `input_dft` | `pw.x` | 指定 exchange-correlation functional，是物理模型选择 | 把泛函当作数值收敛参数；换泛函后不重审结构和下游 |
| `pseudo_dir` / `ATOMIC_SPECIES` | `pw.x` | 固定赝势来源、价电子空间和相对论能力 | 只记录本地文件名，不记录来源、XC、valence 和 UPF 元信息 |
| `ecutwfc` / `ecutrho` | `pw.x` | 控制平面波基组和密度网格误差 | 只按总能收敛，不检查 force、stress、phonon 或 response |
| `K_POINTS` | `pw.x` | 控制 Brillouin zone 采样 | 用 bands path 替代 SCF/DOS/phonon 所需 uniform mesh |
| `occupations` / `smearing` / `degauss` | `pw.x` / `dos.x` | 控制占据模型和费米面附近积分 | 金属用 fixed occupation；用过大 smearing 解释带隙或 DOS 峰 |
| `nspin` / `noncolin` / `lspinorb` | `pw.x` | 定义磁性和相对论模型 | 未审阅磁性构型或无相对论赝势时解释 SOC |
| `HUBBARD` / `Hubbard_U` | `pw.x` | 定义 DFT+U 模型和局域子空间 | 不记录 U 来源；把 U 当作拟合带隙的自由旋钮 |
| `vdw_corr` | `pw.x` | 定义 vdW/dispersion 修正模型 | relax 与后处理使用不同模型但未记录 |
| `tr2_ph` / `ldisp` / `nq1/nq2/nq3` | `ph.x` | 定义 DFPT 响应精度和 q-grid | 普通 SCF 精度直接进入 phonon/EPC |
| `prefix` / `outdir` | 多程序 | 连接上游 scratch 数据与下游程序 | 混用旧 scratch，导致 output 与 input 不对应 |

## 5. output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| SCF 已结束但 force、stress 或 phonon 不稳定 | 总能收敛不足以支撑下游目标量 | cutoff、k mesh、smearing、relax 状态 |
| bands 与 DOS 对金属性判断不一致 | k-path、NSCF mesh、energy reference 或 smearing 不一致 | `workflows/electronic/bands.md` 与 `dos.md` |
| small gap 随 SOC、U 或泛函改变而翻转 | 模型选择影响物理结论 | exchange-correlation、SOC、DFT+U 页面 |
| Gamma acoustic modes 明显偏离零 | 结构、SCF、ASR 或 q-grid 问题 | relax、phonon debugging、DFPT 页面 |
| Born charge / dielectric tensor 缺失或异常 | 电场响应条件不满足，或体系不适合该解释 | `epsil`、metal/insulator 条件、PHonon output |
| work function 随真空厚度漂移 | slab 边界或电势平台未收敛 | electrostatic potential、vacuum、dipole correction |
| EPC、Wannier 或 excited-state 结果高度依赖上游设置 | workflow 传播误差或高级模型边界 | Wannier/EPC/GW-BSE 边界页面 |

## 6. 对 workflow 的影响

- [workflows/ground-state/scf.md](../workflows/ground-state/scf.md)
- [workflows/ground-state/cutoff-convergence.md](../workflows/ground-state/cutoff-convergence.md)
- [workflows/ground-state/kpoint-convergence.md](../workflows/ground-state/kpoint-convergence.md)
- [workflows/electronic/bands.md](../workflows/electronic/bands.md)
- [workflows/electronic/dos.md](../workflows/electronic/dos.md)
- [workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)
- [workflows/advanced/electron-phonon-overview.md](../workflows/advanced/electron-phonon-overview.md)
- [workflows/electronic/work-function.md](../workflows/electronic/work-function.md)

## 7. PASS / WARN / BLOCK 判断

| 状态 | 条件 | 允许进入的下游 |
|---|---|---|
| PASS | 数值收敛、模型选择、赝势、边界条件和 workflow 数据链均与目标性质一致 | 进入目标下游 workflow 或写入结论 |
| WARN | 数值证据可复查，但模型误差、边界条件或上游传播风险仍可能影响解释 | 只进入带限制说明的趋势判断或排错记录 |
| BLOCK | 关键误差类型未识别，或用数值参数试图修复模型选择问题；上游数据链不可追踪 | 停止定量下游，回到模型、来源或 workflow 重新设计 |

## 8. 常见误区

- 把 cutoff 加大当作解决所有误差的方法。
- 换泛函、赝势、U、SOC 或 vdW 后沿用旧结构和旧收敛记录。
- 只看总能收敛，不检查 force、stress、phonon 或 response。
- 把 Kohn-Sham gap 当实验 gap。
- 把 ASR 当作 phonon 收敛测试替代品。
- 把 work function 写成公式后不检查真空电势平台。
- 用 AiiDA、脚本或后处理图像替代 output review。
- 保存图而不保存 input、command、QE version、pseudopotential source 和 `prefix/outdir` 证据。

## 9. 与前沿常识的关系

当前 DFT 使用常识强调误差分层。GGA 带隙低估、self-interaction/delocalization error、DFT+U 参数依赖、SOC 对重元素和拓扑判断的影响、vdW 对结构的影响、phonon/EPC 对上游收敛的敏感性，都是物理判断的一部分。更高级的方法并不自动更可靠；它们需要更严格的输入来源、output review 和下游边界说明。

## 10. 最低掌握深度

- 能把一次异常结果归类为数值误差、模型误差、边界误差、后处理误差或 workflow 传播误差。
- 能说明哪些误差可以通过 convergence scan 降低，哪些需要换模型或补物理假设。
- 能从 output 中找到支持 `PASS / WARN / BLOCK` 的证据。
- 能在 calculation record 中写明当前结论的主要近似和限制。
- 能判断某个高级 workflow 是否需要先回到 SCF、relax、NSCF、DFPT 或 Wannier 质量审阅。

## 11. 对应仓库页面

- [theory-minimum/dft-ks-scf.md](../theory-minimum/dft-ks-scf.md)
- [theory-minimum/plane-wave-cutoff.md](../theory-minimum/plane-wave-cutoff.md)
- [theory-minimum/pseudopotentials.md](../theory-minimum/pseudopotentials.md)
- [workflows/ground-state/scf.md](../workflows/ground-state/scf.md)
- [workflows/phonon/phonon-debugging.md](../workflows/phonon/phonon-debugging.md)
- [standards/output-review-checklist.md](../standards/output-review-checklist.md)
- [standards/pass-warn-block.md](../standards/pass-warn-block.md)
- [references/source-index.md](../references/source-index.md)


## 结论强度

- Strong：本页中由 DFT/DFPT 基础理论、经典方法论文或 QE 官方文档直接支撑的判断。
- Moderate：本页中由综述、方法摘要或多个来源共同支持，但细节需要回到正文或官方文档核验的判断。
- Boundary：本页中用于限制解释范围的判断，不作为定量结论。
- Version-sensitive：涉及 QE、PHonon、Wannier90、EPW、Yambo 或其他工具字段和行为的判断，必须按当前官方文档复查。

## 12. 资料来源

- [Hohenberg and Kohn, Phys. Rev. 136, B864 (1964)](https://doi.org/10.1103/PhysRev.136.B864)
- [Kohn and Sham, Phys. Rev. 140, A1133 (1965)](https://doi.org/10.1103/PhysRev.140.A1133)
- [Perdew and Schmidt, AIP Conf. Proc. 577, 1 (2001)](https://doi.org/10.1063/1.1390175)
- [Quantum ESPRESSO documentation](https://www.quantum-espresso.org/documentation/)
- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
