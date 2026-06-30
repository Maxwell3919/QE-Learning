# Physics Judgement

`physics-judgement/` 用于补充 DFT/QE 计算判断所需的物理理论背景。它连接 DFT 近似、QE 输入字段、output review、workflow 依赖和 `PASS / WARN / BLOCK` 判断，帮助学习者解释为什么某些计算结果可能不可信，以及哪些物理效应会改变结论。

本目录与 `theory-minimum/` 的区别在于：`theory-minimum/` 回答正确使用 QE 的最低理论；本目录回答如何形成更可靠的 DFT 物理判断。它也不替代 `workflows/`：具体怎么运行仍回到 workflow 页面，本目录只解释判断背后的物理原因、近似来源和失败模式。

## 阅读方式

如果 output 中出现虚频、带隙异常、磁矩异常、DOS 与 bands 不一致、SCF 振荡、work function 不稳定、Born charge 不合理，不应只调参数。先确认 workflow 记录完整，再回到本目录查可能的物理原因：泛函、赝势、边界条件、维度、smearing、SOC、DFT+U、vdW、DFPT 响应或后处理近似。

建议按现象回查：

- 带隙、DOS、能带解释异常：读 `02`、`03`、`04`、`08`。
- 磁矩、SOC、DFT+U、强关联异常：读 `05`、`06`。
- 层状、表面、二维或真空相关问题：读 `07`、`11`、`12`。
- phonon、Born charge、EPC 或 soft mode 问题：读 `09`、`10`、`14`。
- 激发态、光谱和 GW/BSE 边界：读 `15`。
- 热力学、MD 和不确定性：读 `16`、`17`、`18`。

本目录不写具体材料算例。页面中只使用金属体系、极性绝缘体、小带隙体系、强关联体系、二维体系、磁性体系、重元素体系、软模体系、表面/界面体系等通用物理情境。

## 相关 QE 输入字段与使用锚点

本目录讨论的物理判断必须回到 QE 使用证据。阅读时至少要能把物理风险映射到这些对象：

- `input_dft`、`HUBBARD`、`vdw_corr`、`noncolin/lspinorb`：模型选择是否改变结论。
- `ecutwfc/ecutrho`、`K_POINTS`、`nq1/nq2/nq3`、`tr2_ph`：数值精度是否足以支持目标物理量。
- `occupations/smearing/degauss`：金属、费米面、DOS、phonon 和 EPC 的占据边界。
- `prefix/outdir`、QE version、pseudopotential source：workflow 数据链是否可追溯。
- output 中的 warning、symmetry、Fermi energy、magnetization、force/stress、phonon frequency、Born charge、potential platform：物理判断是否有证据。

对应的最低理论入口是 [theory-minimum/README.md](../theory-minimum/README.md)，具体 workflow 入口是 [workflows/README.md](../workflows/README.md)。

## 对 workflow 的影响

物理判断会改变 workflow 的准入和解释边界：

- ground-state workflow：决定 SCF、收敛性、occupation、赝势和泛函选择是否足以支撑后续计算。
- electronic workflow：决定 bands、DOS、PDOS、Fermi surface、work function 等结果能否做定量解释。
- phonon/DFPT workflow：决定虚频、Born effective charge、dielectric tensor、LO-TO splitting 和 EPC 是否具备物理意义。
- advanced workflow：决定 DFT+U、SOC、vdW、hybrid、Wannier、MD、NEB、GW/BSE 等方法是否只是入口概念，还是已经满足可审阅的数据链。

## 常见误区

- 把数值收敛问题误判为物理效应。
- 把泛函、DFT+U、SOC、vdW 或 hybrid 当作普适精度提升。
- 把 Kohn-Sham eigenvalue、DFT band gap 或可视化图像直接当作实验观测量。
- 把 ASR、smearing、scissor 或后处理平滑当作替代上游收敛的修正。
- output 只有 `JOB DONE` 就进入下游，而没有记录 warning、数据来源和 `PASS / WARN / BLOCK`。

## 最低掌握深度

读完本目录后，学习者应能：

- 把异常 output 归类为数值误差、模型误差、边界误差、后处理误差或 workflow 传播误差。
- 说明哪些问题可以通过收敛测试降低，哪些问题需要换物理模型或补充对照。
- 将 band gap、磁性、SOC、DFT+U、vdW、phonon、Born charge、work function、EPC、激发态和有限温度结论写成带边界的判断。
- 把物理风险落实到 `PASS / WARN / BLOCK` 和 calculation record。


## 结论型补写页面

这些页面来自 authoritative literature loop 的稳定判断结论。它们不替代编号理论页，而是用于从 output 现象快速回查物理判断边界。

- [DFT ground-state boundary](dft-ground-state-boundary.md)
- [Kohn-Sham eigenvalue boundary](kohn-sham-eigenvalue-boundary.md)
- [Functional choice and sensitivity](functional-choice-and-sensitivity.md)
- [Numerical vs model error](numerical-vs-model-error.md)
- [Pseudopotential transferability and functional consistency](pseudopotential-transferability-and-functional-consistency.md)
- [Band gap problem and delocalization](band-gap-problem-and-delocalization.md)
- [K-mesh and smearing sensitivity](kmesh-smearing-sensitivity.md)
- [Relax is not final electronic structure](relax-is-not-final-electronic-structure.md)
- [Imaginary frequency triage](imaginary-frequency-triage.md)
- [U value provenance and boundary](u-value-provenance-and-boundary.md)
- [SOC and symmetry boundary](soc-and-symmetry-boundary.md)
- [vdW and low-dimensional boundary](vdw-and-low-dimensional-boundary.md)
- [Work function and electrostatic boundary](work-function-and-electrostatic-boundary.md)
- [Wannier validation and window choice](wannier-validation-and-window-choice.md)
- [EPC data chain and convergence](epc-data-chain-and-convergence.md)
- [Ground-state vs excited-state boundary](ground-state-vs-excited-state.md)
- [Energy vs free energy](energy-vs-free-energy.md)
- [Uncertainty statement template](uncertainty-statement-template.md)

## 结论强度

- Strong：经典理论、方法论文或官方文档直接支持。
- Moderate：摘要、综述或多个方法来源共同支持，但细节需正文或官方文档核验。
- Boundary：用于限制解释范围，不作为定量结论。
- Version-sensitive：依赖 QE / PHonon / EPW / Wannier90 / Yambo 等版本，必须查官方文档。

## 页面索引

- [DFT 近似地图](01-dft-approximation-map.md)
- [交换关联泛函阶梯](02-exchange-correlation-ladder.md)
- [自相互作用与离域误差](03-self-interaction-and-delocalization-error.md)
- [带隙问题](04-band-gap-problem-and-derivative-discontinuity.md)
- [强关联与 DFT+U](05-strong-correlation-and-dft-plus-u.md)
- [磁性、自旋与 SOC](06-magnetism-spin-and-soc.md)
- [vdW 与非局域相互作用](07-van-der-waals-and-nonlocal-interactions.md)
- [金属、费米面与 smearing](08-metals-fermi-surface-and-smearing.md)
- [声子软模与动力学稳定性](09-phonons-soft-modes-and-dynamical-stability.md)
- [DFPT 响应与极性材料](10-dfpt-response-and-polar-materials.md)
- [表面、界面与静电势](11-surfaces-interfaces-and-electrostatics.md)
- [有限尺寸、边界条件与维度](12-finite-size-boundary-and-dimensionality.md)
- [Wannier 化与有效模型](13-wannierization-and-effective-model-thinking.md)
- [EPC 物理判断](14-electron-phonon-coupling-physical-judgement.md)
- [光谱、GW/BSE 与激发态边界](15-spectroscopy-gw-bse-and-excited-state-boundaries.md)
- [热力学、自由能与 MD 边界](16-thermodynamics-free-energy-and-md-boundaries.md)
- [不确定性、误差条与可复现性](17-uncertainty-error-bars-and-reproducibility.md)
- [物理判断清单](18-physics-judgement-checklist.md)

## 与现有目录的关系

- `learn/`：决定何时进入更深层物理判断。
- `workflows/`：提供具体 QE 运行链和 output review。
- `theory-minimum/`：提供最低理论定义。
- `references/`：追踪公开来源。
- `standards/`：把物理判断写成可复查记录。

## 资料来源

- [references/source-index.md](../references/source-index.md)
- [standards/output-review-checklist.md](../standards/output-review-checklist.md)
- [standards/pass-warn-block.md](../standards/pass-warn-block.md)
