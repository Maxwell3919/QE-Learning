# QE Physics Judgement Conclusion Reference Book (v1)

本文件是 `physics-judgement/` 的结论内核。它把 DFT/QE 计算判断中最稳定、最容易影响结论可信度的规则压缩成一份可复查的参考册，用于指导后续页面维护、workflow 审阅和专题分工。

它不是论文综述，也不是新的 workflow 教程。使用时应先定位 output 现象，再回查对应的 `physics-judgement/`、`theory-minimum/`、`workflows/`、`standards/` 与 `references/` 页面。

## 1. 核心判断总则

1. DFT/QE 的基础对象是 ground-state density、total energy 以及由能量导数得到的 force/stress；Kohn-Sham bands、DOS、PDOS 是辅助单粒子图像，不能自动等同于 quasiparticle spectrum 或 optical spectrum。
2. 程序正常结束只说明运行完成，不等于 `PASS`。计算结论必须经过 output review、数值收敛、模型边界、来源追踪和下游准入判断。
3. numerical convergence 和 model correctness 是两类问题。提高 `ecutwfc`、`ecutrho`、k mesh 或 q-grid 可以降低离散误差，但不能修复不合适的 functional、pseudopotential、Hubbard U、SOC、vdW、slab boundary 或 excited-state boundary。
4. functional、pseudopotential、DFT+U、SOC、vdW、hybrid、slab electrostatics 等设置是物理模型选择；它们必须随目标 observable 一起记录。
5. phonon/DFPT 是 response workflow。它继承并放大上游 SCF、结构优化、赝势、cutoff、k mesh、smearing、symmetry 和 q-grid 的问题。
6. Wannierization、EPC、GW/BSE、TDDFT、XSpectra、Yambo 等高级流程不是普通后处理；它们引入新的数据链、收敛维度和解释边界。
7. 每个可发表或可复查的 DFT/QE 结论都应带有 uncertainty statement，说明数值误差、模型误差、边界条件、workflow provenance 和已知限制。

## 2. 结论强度标签

| 标签 | 含义 | 写入页面时的要求 |
|---|---|---|
| `Strong` | 由经典理论、方法论文或官方文档直接支持 | 可写成明确判断，但仍要说明适用范围 |
| `Moderate` | 由综述、摘要级调研或多个来源共同支持 | 可作为稳定提醒，细节需要回查正文或官方文档 |
| `Boundary` | 用于限制解释范围，不作为定量结论 | 应进入 `WARN/BLOCK` 条件或 workflow 边界 |
| `Version-sensitive` | 依赖 QE、PHonon、EPW、Wannier90、Yambo 等版本 | 必须以当前官方文档核验字段和程序行为 |

## 3. PASS / WARN / BLOCK 总表

| 状态 | 条件 | 下游动作 |
|---|---|---|
| `PASS` | input、output、上游依赖、数值收敛、模型选择、来源和 known limitations 均可追踪；目标 observable 与所用方法匹配 | 可以进入下游 workflow 或写入正式结论 |
| `WARN` | 程序结束且主要输出可读，但存在模型边界、收敛敏感性、解释边界或来源不足；风险已记录 | 只能进入探索性下游或写条件结论 |
| `BLOCK` | 关键 input/output 缺失、scratch 混用、来源不可追踪、目标 observable 超出方法边界，或 output 与物理任务矛盾 | 停止下游；先重算、补审阅或更换方法 |

## 4. 误差分类

| 误差来源 | 可否主要通过收敛测试降低 | 需要记录什么 | 对结论的影响 |
|---|---|---|---|
| basis / cutoff error | 可以 | `ecutwfc`、`ecutrho`、pseudo 类型、目标量收敛表 | 影响 energy、force、stress、phonon、EPC |
| k/q sampling error | 可以 | SCF mesh、DOS mesh、bands path、q-grid、shift、irreducible points | 影响金属、DOS、phonon、EPC、Fermi surface |
| SCF / response convergence error | 可以 | `conv_thr`、`tr2_ph`、iteration、estimated accuracy、warnings | 影响 charge density、force、phonon frequency、Born charge |
| functional error | 不能只靠收敛测试修复 | `input_dft`、functional family、目标 observable | 影响 lattice、gap、magnetism、phonon、surface、EPC |
| pseudopotential error | 不能只靠 cutoff 证明完全可靠 | UPF 来源、functional 一致性、valence、relativistic 类型 | 影响结构、力、应力、SOC、phonon、EPC |
| Hubbard / SOC / vdW model error | 不能 | `HUBBARD`、U 来源、projector/manifold、`noncolin/lspinorb`、`vdw_corr` | 改变局域态、磁性、能带、结构、phonon |
| finite-size / boundary error | 部分可以 | cell、vacuum、dipole、charged state、dimensionality、image interaction | 影响表面、work function、缺陷、低维体系 |
| workflow propagation error | 不能由单步检查替代 | `prefix/outdir`、上游 PASS/WARN/BLOCK、final static SCF、版本 | 下游结果继承上游问题 |
| interpretation boundary | 不能 | 目标 observable、方法边界、是否需要 GW/BSE/TDDFT/EPW | 决定能否做定量物理解释 |

## 5. 稳定结论与页面路由

### 5.1 Ground-state boundary

- `Strong`：DFT 的核心可靠对象优先是 ground-state density、total energy、force 和 stress；Kohn-Sham eigenvalues 是辅助体系输出。
- `Boundary`：当目标是 quasiparticle gap、optical absorption、exciton、XAS、EELS 或 excited-state spectrum，普通 DFT bands/DOS 只能作为前置图像。
- 回查页面：
  - [dft-ground-state-boundary.md](dft-ground-state-boundary.md)
  - [kohn-sham-eigenvalue-boundary.md](kohn-sham-eigenvalue-boundary.md)
  - [ground-state-vs-excited-state.md](ground-state-vs-excited-state.md)
  - [workflows/electronic/bands.md](../workflows/electronic/bands.md)
  - [workflows/electronic/dos.md](../workflows/electronic/dos.md)

### 5.2 Functional, band gap and delocalization

- `Strong`：functional choice 是模型选择，不是普通数值参数。
- `Strong`：GGA/LDA band gap problem 不能通过单纯加密 k mesh 或提高 cutoff 消除。
- `Moderate`：self-interaction / delocalization error 常表现为 gap 偏小、局域态过度扩展、磁矩偏小或电荷转移解释不稳。
- 回查页面：
  - [functional-choice-and-sensitivity.md](functional-choice-and-sensitivity.md)
  - [band-gap-problem-and-delocalization.md](band-gap-problem-and-delocalization.md)
  - [02-exchange-correlation-ladder.md](02-exchange-correlation-ladder.md)
  - [03-self-interaction-and-delocalization-error.md](03-self-interaction-and-delocalization-error.md)
  - [04-band-gap-problem-and-derivative-discontinuity.md](04-band-gap-problem-and-derivative-discontinuity.md)

### 5.3 Numerical convergence and pseudopotential consistency

- `Strong`：换 pseudopotential 等价于更换 electron-ion interaction model，必须重审 cutoff、k mesh 和目标 observable。
- `Strong`：bands path 是能带绘图路径，不是 Brillouin-zone integration mesh；不能替代 DOS/PDOS 的 dense uniform mesh。
- `Boundary`：pseudo 与 `input_dft`、SOC、DFT+U、phonon、EPC 的一致性需要按目标 workflow 审阅。
- 回查页面：
  - [numerical-vs-model-error.md](numerical-vs-model-error.md)
  - [pseudopotential-transferability-and-functional-consistency.md](pseudopotential-transferability-and-functional-consistency.md)
  - [kmesh-smearing-sensitivity.md](kmesh-smearing-sensitivity.md)
  - [theory-minimum/pseudopotentials.md](../theory-minimum/pseudopotentials.md)
  - [workflows/ground-state/cutoff-convergence.md](../workflows/ground-state/cutoff-convergence.md)
  - [workflows/ground-state/kpoint-convergence.md](../workflows/ground-state/kpoint-convergence.md)

### 5.4 Relaxation, static SCF and workflow propagation

- `Strong`：relax/vc-relax 输出的是优化过程和最终结构，不是最终电子结构结论。
- `Strong`：进入 bands、DOS、PDOS 或 phonon 前，应使用最终结构重做 static SCF，并重新审阅 k density、symmetry、stress 和 warnings。
- `Boundary`：cell 改变后继续沿用旧 k mesh 可能改变实际采样密度。
- 回查页面：
  - [relax-is-not-final-electronic-structure.md](relax-is-not-final-electronic-structure.md)
  - [workflows/ground-state/relax.md](../workflows/ground-state/relax.md)
  - [workflows/ground-state/vc-relax.md](../workflows/ground-state/vc-relax.md)
  - [theory-minimum/force-stress-relaxation.md](../theory-minimum/force-stress-relaxation.md)

### 5.5 Phonon, DFPT and imaginary frequency

- `Strong`：phonon 是 response calculation，依赖并放大上游 SCF、结构、cutoff、k mesh、smearing、symmetry 和 q-grid 的问题。
- `Strong`：ASR 只能处理平移不变性残差，不能替代收敛测试或结构优化。
- `Boundary`：imaginary frequency 至少需要按数值误差、结构未充分优化、真实动力学不稳定三类排查。
- 回查页面：
  - [imaginary-frequency-triage.md](imaginary-frequency-triage.md)
  - [09-phonons-soft-modes-and-dynamical-stability.md](09-phonons-soft-modes-and-dynamical-stability.md)
  - [10-dfpt-response-and-polar-materials.md](10-dfpt-response-and-polar-materials.md)
  - [workflows/phonon/phonon-debugging.md](../workflows/phonon/phonon-debugging.md)
  - [workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)

### 5.6 DFT+U, SOC and vdW

- `Strong`：U 不是 universal element parameter；必须记录 U 来源、Hubbard manifold、projector、functional、pseudopotential 和是否 self-consistent。
- `Version-sensitive`：QE 的 `HUBBARD` card、DFT+U+V、hp.x 等字段和行为需要按当前官方文档核验。
- `Strong`：SOC/noncollinear 需要同时审阅 fully relativistic pseudopotential、`noncolin/lspinorb`、初始磁构型、symmetry 和 band degeneracy。
- `Moderate`：vdW correction 是模型选择，会改变结构，并把影响传递到 bands、phonon、surface 和 interface 结论。
- 回查页面：
  - [u-value-provenance-and-boundary.md](u-value-provenance-and-boundary.md)
  - [soc-and-symmetry-boundary.md](soc-and-symmetry-boundary.md)
  - [vdw-and-low-dimensional-boundary.md](vdw-and-low-dimensional-boundary.md)
  - [workflows/advanced/dft-plus-u.md](../workflows/advanced/dft-plus-u.md)
  - [workflows/advanced/noncollinear-soc.md](../workflows/advanced/noncollinear-soc.md)
  - [workflows/advanced/vdW-corrections.md](../workflows/advanced/vdW-corrections.md)

### 5.7 Surface electrostatics and work function

- `Strong`：work function 的表达式只在 slab model、vacuum level、电势平台、Fermi level、dipole correction 和 boundary condition 已审阅时才有物理意义。
- `Boundary`：charged slab、charged defect、极性表面、二维体系和界面体系需要额外审阅 periodic image interaction 与 electrostatic correction。
- 回查页面：
  - [work-function-and-electrostatic-boundary.md](work-function-and-electrostatic-boundary.md)
  - [11-surfaces-interfaces-and-electrostatics.md](11-surfaces-interfaces-and-electrostatics.md)
  - [12-finite-size-boundary-and-dimensionality.md](12-finite-size-boundary-and-dimensionality.md)
  - [workflows/electronic/work-function.md](../workflows/electronic/work-function.md)
  - [workflows/electronic/electrostatic-potential.md](../workflows/electronic/electrostatic-potential.md)

### 5.8 Wannierization and EPC

- `Strong`：Wannier-interpolated bands 必须在目标能区回对 QE bands；projection、frozen window、outer window 和 disentanglement 必须记录。
- `Boundary`：spread 是审阅指标之一，不能单独证明 Wannier model 成功。
- `Strong`：EPC 可信度依赖完整数据链：relaxed structure -> converged SCF -> DFPT phonons -> Wannier validation -> dense k/q interpolation -> observable-specific convergence。
- `Boundary`：EPC 结果不能只凭单个 summary scalar 判断，需要审阅 phonon linewidth、Eliashberg function、mobility、Tc 或目标 observable 对应的收敛链。
- 回查页面：
  - [wannier-validation-and-window-choice.md](wannier-validation-and-window-choice.md)
  - [epc-data-chain-and-convergence.md](epc-data-chain-and-convergence.md)
  - [13-wannierization-and-effective-model-thinking.md](13-wannierization-and-effective-model-thinking.md)
  - [14-electron-phonon-coupling-physical-judgement.md](14-electron-phonon-coupling-physical-judgement.md)
  - [workflows/advanced/wannier90-overview.md](../workflows/advanced/wannier90-overview.md)
  - [workflows/advanced/electron-phonon-overview.md](../workflows/advanced/electron-phonon-overview.md)

### 5.9 Excited states, free energy and uncertainty

- `Strong`：ground-state DFT bands/DOS 不能替代 quasiparticle、optical excitation、exciton、XAS、EELS 等 excited-state 方法。
- `Boundary`：0 K total energy difference 不能直接代表实验温度下的 phase stability；可能需要 zero-point energy、phonon free energy、QHA、anharmonicity、AIMD 或 free-energy method。
- `Strong`：每个 DFT/QE 结论至少应说明 numerical convergence、model choices、pseudopotential、functional、finite-size/boundary、workflow provenance 和 known limitations。
- 回查页面：
  - [ground-state-vs-excited-state.md](ground-state-vs-excited-state.md)
  - [energy-vs-free-energy.md](energy-vs-free-energy.md)
  - [uncertainty-statement-template.md](uncertainty-statement-template.md)
  - [15-spectroscopy-gw-bse-and-excited-state-boundaries.md](15-spectroscopy-gw-bse-and-excited-state-boundaries.md)
  - [16-thermodynamics-free-energy-and-md-boundaries.md](16-thermodynamics-free-energy-and-md-boundaries.md)
  - [17-uncertainty-error-bars-and-reproducibility.md](17-uncertainty-error-bars-and-reproducibility.md)

## 6. 从 output 现象到判断页

| output 现象 | 优先分类 | 首先回查 | 下游决策 |
|---|---|---|---|
| SCF 收敛但目标 band gap 与物理预期冲突 | model / interpretation boundary | [band-gap-problem-and-delocalization.md](band-gap-problem-and-delocalization.md) | bands/DOS 可作趋势；定量 gap 进入 `WARN/BLOCK` |
| DOS 与 bands 对金属性判断不一致 | k mesh / smearing / energy reference | [kmesh-smearing-sensitivity.md](kmesh-smearing-sensitivity.md) | 停止解释 DOS，重审 NSCF mesh 与 Fermi level |
| relax 已结束但 final force/stress 边界不清 | workflow propagation | [relax-is-not-final-electronic-structure.md](relax-is-not-final-electronic-structure.md) | 进入下游前重做 final static SCF |
| phonon 出现 imaginary frequency | phonon triage | [imaginary-frequency-triage.md](imaginary-frequency-triage.md) | 先排查数值、结构和 q-grid，再判断软模 |
| Born charge 或 dielectric tensor 看似异常 | DFPT response boundary | [10-dfpt-response-and-polar-materials.md](10-dfpt-response-and-polar-materials.md) | 核验 insulator 条件、`epsil` 和 non-analytic correction |
| 磁矩或 band degeneracy 解释不稳 | magnetism / SOC / symmetry | [soc-and-symmetry-boundary.md](soc-and-symmetry-boundary.md) | 回查 pseudo、`noncolin/lspinorb`、symmetry 和磁构型 |
| work function 随 vacuum 或方向变化明显 | electrostatic boundary | [work-function-and-electrostatic-boundary.md](work-function-and-electrostatic-boundary.md) | 暂停报告数值，重审平台、偶极和 boundary |
| Wannier bands 不能复现 QE bands | effective model failure | [wannier-validation-and-window-choice.md](wannier-validation-and-window-choice.md) | 不进入 Berry / transport / EPC |
| EPC summary value 看起来异常 | data-chain failure | [epc-data-chain-and-convergence.md](epc-data-chain-and-convergence.md) | 回查 SCF、phonon、Wannier、dense k/q 和 observable |
| 只报告 0 K total energy difference | thermodynamics boundary | [energy-vs-free-energy.md](energy-vs-free-energy.md) | 只能写 0 K model conclusion，不能直接外推有限温 |

## 7. 最小 uncertainty statement 模板

```markdown
## Uncertainty Statement

- Numerical convergence:
  - cutoff:
  - k/q sampling:
  - SCF / response threshold:
  - target observable used for convergence:
- Model choices:
  - exchange-correlation functional:
  - pseudopotential source and consistency:
  - Hubbard / SOC / vdW / hybrid settings:
- Boundary conditions:
  - cell / dimensionality / vacuum / charged state:
  - finite-size or image-interaction risk:
- Workflow provenance:
  - upstream calculations:
  - `prefix/outdir`:
  - QE / PHonon / EPW / Wannier90 / Yambo version:
- Known limitations:
  - ground-state vs excited-state boundary:
  - finite-temperature / free-energy boundary:
  - interpretation strength:
- PASS / WARN / BLOCK:
  - status:
  - reason:
  - allowed downstream workflows:
```

## 8. 结论来源

| 结论 | 来源类型 | 支撑文献/文档 | 是否需要全文核验 |
|---|---|---|---|
| DFT 是 ground-state theory，Kohn-Sham bands 有解释边界 | A0/A1 | Hohenberg-Kohn；Kohn-Sham；Kohn Nobel Lecture；Onida-Reining-Rubio RMP | 引用边界需要核验 |
| 收敛测试不能替代模型判断 | A2 | Kim-Sim-Burke；Lejaeghere et al.；QE official documentation | 具体字段需核验 |
| band gap problem 不是 k mesh 问题 | A2 | Mori-Sanchez-Cohen-Yang；derivative discontinuity 相关综述 | 表述边界需核验 |
| pseudopotential 与 functional consistency 会影响下游 | A1 | Vanderbilt USPP；Blochl PAW；QE code papers；QE INPUT_PW | QE 字段需核验 |
| phonon 是 response calculation，ASR 不能替代收敛 | A0/A1 | Baroni et al. DFPT RMP；Gonze-Lee；QE INPUT_PH；PHonon guide | PH 字段需核验 |
| U 不是 universal element parameter | A1/A2 | Cococcioni-de Gironcoli；Timrov et al.；QE Hubbard documentation | QE 版本字段需核验 |
| SOC 需要 pseudo、noncollinear 和 symmetry 联动审阅 | A1/B1 | QE INPUT_PW；noncollinear DFT 文献；topological band theory review | QE 版本字段需核验 |
| vdW 是模型选择，会改变结构和下游性质 | A1/B1 | Dion vdW-DF；Tkatchenko-Scheffler；QE INPUT_PW | 具体模型字段需核验 |
| Wannierization 必须回对 QE bands | A1 | Marzari-Vanderbilt；Souza-Marzari-Vanderbilt；Wannier90 documentation | 工具行为需核验 |
| EPC 需要 DFPT、Wannier 和 dense k/q 完整数据链 | A1/B1 | Giustino RMP；EPW documentation；EPW code papers | EPW 版本行为需核验 |
| 0 K total energy 不能直接代表有限温自由能 | A1/B1 | Car-Parrinello；phonon/free-energy and anharmonicity reviews | 方法边界需核验 |
| uncertainty statement 是 DFT 结论的一部分 | A2 | Lejaeghere et al.；FAIR/provenance literature；QE-Learning standards | 记录模板需维护 |

## 9. 资料来源

- [references/canonical-literature.md](../references/canonical-literature.md)
- [references/reading-maintenance-policy.md](../references/reading-maintenance-policy.md)
- [references/source-index.md](../references/source-index.md)
- [Quantum ESPRESSO official documentation](https://www.quantum-espresso.org/documentation/)
- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
- [QE INPUT_PH](https://www.quantum-espresso.org/Doc/INPUT_PH.html)
- [EPW documentation](https://docs.epw-code.org/)
- [Wannier90 documentation](https://wannier.org/)
- [Yambo educational wiki](https://wiki.yambo-code.eu/wiki/index.php?title=Main_Page)
- [Yambo learn](https://www.yambo-code.eu/learn/)

## 10. 维护入口

- [physics-judgement/README.md](README.md)
- [physics-judgement/18-physics-judgement-checklist.md](18-physics-judgement-checklist.md)
- [references/canonical-literature.md](../references/canonical-literature.md)
- [references/reading-maintenance-policy.md](../references/reading-maintenance-policy.md)
- [references/source-index.md](../references/source-index.md)
- [standards/output-review-checklist.md](../standards/output-review-checklist.md)
- [standards/calculation-record-template.md](../standards/calculation-record-template.md)
- [standards/citation-and-source-policy.md](../standards/citation-and-source-policy.md)
