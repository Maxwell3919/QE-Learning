# Physics Judgement Checklist

本页把 `physics-judgement/` 压缩成一张审阅清单。它不替代各主题页；当某一组出现 WARN 或 BLOCK 信号时，应回查对应页面，再回到 `workflows/` 和 `standards/` 写入记录。

## 核心判断结论

- 先把异常 output 归类为 numerical、model、boundary、post-processing 或 workflow propagation 问题。
- 再决定是继续收敛测试、切换物理模型、回到上游 workflow，还是停止下游解释。
- `PASS / WARN / BLOCK` 必须写明证据、允许进入的下游和 uncertainty statement。

## PASS / WARN / BLOCK 判断

- PASS：必问问题已有 output 证据支撑，相关 input/source/workflow 记录完整，允许进入对应下游。
- WARN：证据可复查但存在模型、数值、边界或后处理限制，只能写带条件的趋势或排错结论。
- BLOCK：关键证据缺失、物理条件不满足或上游数据链不可追踪，停止定量解释。

## 最低掌握深度

读者应能用本清单定位 WARN/BLOCK 来源，回查对应 physics-judgement 页面，并把结论写入 calculation record。

## 相关 QE 输入字段

这张清单不替代各主题页的字段表。跨主题审阅时，应优先回查以下字段是否被记录并和物理判断一致：

| 字段 | 程序 | 判断意义 |
|---|---|---|
| `input_dft` | `pw.x` | 泛函模型是否与目标性质匹配 |
| `ecutwfc/ecutrho` | `pw.x` | basis 收敛是否足以支撑能量、力、应力和响应量 |
| `K_POINTS` | `pw.x` | ground state、DOS、Fermi surface、phonon/EPC 的采样是否充分 |
| `occupations/smearing/degauss` | `pw.x` | 金属、小带隙和有限电子温度处理是否合理 |
| `HUBBARD` | `pw.x` | DFT+U 模型、投影和 U 来源是否可追踪 |
| `noncolin/lspinorb` | `pw.x` | noncollinear/SOC 判断是否有相对论赝势和 symmetry 审阅 |
| `vdw_corr` | `pw.x` | vdW 模型是否与结构优化和后处理一致 |
| `tr2_ph/epsil/ldisp/nq1/nq2/nq3` | `ph.x` | DFPT response、q-grid 和 polar branch 是否满足物理条件 |

## output review 中的迹象

| output 迹象 | 可能含义 | 需要回查 |
|---|---|---|
| SCF 振荡或 warning 未解释 | ground state 可能不是可用上游 | SCF、smearing、mixing、赝势 |
| band gap、磁矩或 DOS 与预期冲突 | 模型选择或占据可能不合适 | functional、DFT+U、SOC、k mesh |
| 虚频或 acoustic branch 漂移 | 可能是数值噪声、结构问题或真实不稳定 | phonon、ASR、结构优化、q-grid |
| Born charge/dielectric 异常 | polar response 条件或收敛可能失败 | `epsil`、metal/insulator 条件、non-analytic correction |
| work function 漂移 | 电势平台、真空、偶极修正或 slab 边界不足 | electrostatic potential、surface/electrostatics |
| EPC 或 Wannier 结果强烈依赖 mesh/window | 插值和上游数据链不稳定 | Wannier、phonon、Fermi surface sampling |

## 对 workflow 的影响

- PASS：允许进入对应下游，但仍应在记录中写清模型边界。
- WARN：只能进入探索性下游或趋势判断，不能直接写定量结论。
- BLOCK：停止下游解释，回到对应 workflow 或 physics-judgement 页面复查。

## 常见误区

- 把 `JOB DONE` 当作物理 PASS。
- 只看一个 output 数字，不回查输入模型和上游依赖。
- 用后处理平滑、ASR 或更大的 smearing 掩盖上游问题。
- 对不同泛函、U、SOC 或 vdW 模型的结果做直接总能比较，却不说明模型改变。
- 对 phonon、EPC、work function、excited-state 结论忽略边界条件和数据链。

## SCF/ground state

### 必问问题

- SCF 是否收敛到目标电子态？
- `prefix/outdir` 是否可追溯？

### WARN 信号

- SCF 振荡、estimated accuracy 不足、warning 未解释

### BLOCK 信号

- 收敛失败、旧 scratch 混用、赝势来源不明

### 回查页面

- [01-dft-approximation-map.md](01-dft-approximation-map.md)
- [../workflows/ground-state/scf.md](../workflows/ground-state/scf.md)

## functional

### 必问问题

- 泛函是否与物理问题匹配？
- 换泛函后是否重审结构和下游？

### WARN 信号

- 不同泛函趋势差异大

### BLOCK 信号

- 把泛函当数值参数调节

### 回查页面

- [02-exchange-correlation-ladder.md](02-exchange-correlation-ladder.md)
- [../workflows/advanced/hybrid-functional-overview.md](../workflows/advanced/hybrid-functional-overview.md)

## pseudopotential

### 必问问题

- UPF 来源、XC、valence、relativistic 信息是否记录？

### WARN 信号

- cutoff 或 valence 不清

### BLOCK 信号

- 换赝势不重做收敛

### 回查页面

- [01-dft-approximation-map.md](01-dft-approximation-map.md)
- [../theory-minimum/pseudopotentials.md](../theory-minimum/pseudopotentials.md)

## k-point/smearing

### 必问问题

- mesh、shift、occupation 是否适合目标？

### WARN 信号

- Fermi energy/DOS 对 k mesh 敏感

### BLOCK 信号

- 金属 fixed occupations 或 degauss 过大

### 回查页面

- [08-metals-fermi-surface-and-smearing.md](08-metals-fermi-surface-and-smearing.md)
- [../workflows/ground-state/smearing-and-occupations.md](../workflows/ground-state/smearing-and-occupations.md)

## magnetism/SOC

### 必问问题

- 磁性构型、SOC 赝势和 symmetry 是否审阅？

### WARN 信号

- 磁矩依赖初态

### BLOCK 信号

- 无相对论赝势解释 SOC

### 回查页面

- [06-magnetism-spin-and-soc.md](06-magnetism-spin-and-soc.md)
- [../workflows/advanced/noncollinear-soc.md](../workflows/advanced/noncollinear-soc.md)

## DFT+U/strong correlation

### 必问问题

- U 来源、投影子空间和 occupation matrix 是否记录？

### WARN 信号

- 不同 U 改变趋势

### BLOCK 信号

- 随意调 U 匹配单一观测量

### 回查页面

- [05-strong-correlation-and-dft-plus-u.md](05-strong-correlation-and-dft-plus-u.md)
- [../workflows/advanced/dft-plus-u.md](../workflows/advanced/dft-plus-u.md)

## vdW

### 必问问题

- 结构优化是否使用同一 vdW 模型？

### WARN 信号

- 层间距或吸附高度敏感

### BLOCK 信号

- relax 和后处理模型不一致

### 回查页面

- [07-van-der-waals-and-nonlocal-interactions.md](07-van-der-waals-and-nonlocal-interactions.md)
- [../workflows/advanced/vdW-corrections.md](../workflows/advanced/vdW-corrections.md)

## bands/DOS

### 必问问题

- Kohn-Sham gap 是否按边界解释？

### WARN 信号

- bands 与 DOS 能量零点不一致

### BLOCK 信号

- 把 DFT gap 当实验 gap

### 回查页面

- [04-band-gap-problem-and-derivative-discontinuity.md](04-band-gap-problem-and-derivative-discontinuity.md)
- [../workflows/electronic/bands.md](../workflows/electronic/bands.md)

## phonon

### 必问问题

- 虚频是否通过结构、SCF、cutoff、k/q、ASR 复查？

### WARN 信号

- 小虚频或声学支漂移

### BLOCK 信号

- 直接把虚频等同真实不稳定

### 回查页面

- [09-phonons-soft-modes-and-dynamical-stability.md](09-phonons-soft-modes-and-dynamical-stability.md)
- [../workflows/phonon/phonon-debugging.md](../workflows/phonon/phonon-debugging.md)

## DFPT response

### 必问问题

- `epsil`、Born charge、dielectric 分支是否物理有效？

### WARN 信号

- Born charge 异常或 LO-TO 不稳定

### BLOCK 信号

- 在金属中解释 Born charge

### 回查页面

- [10-dfpt-response-and-polar-materials.md](10-dfpt-response-and-polar-materials.md)
- [../workflows/phonon/dielectric-born-effective-charge.md](../workflows/phonon/dielectric-born-effective-charge.md)

## surface/electrostatics

### 必问问题

- 真空平台、偶极修正和平均方向是否审阅？

### WARN 信号

- work function 随真空漂移

### BLOCK 信号

- 无平台仍解释 work function

### 回查页面

- [11-surfaces-interfaces-and-electrostatics.md](11-surfaces-interfaces-and-electrostatics.md)
- [../workflows/electronic/work-function.md](../workflows/electronic/work-function.md)

## Wannier/EPC

### 必问问题

- Wannier bands 是否复现 QE bands？EPC mesh 是否收敛？

### WARN 信号

- spread 异常、lambda 对 mesh 敏感

### BLOCK 信号

- phonon 未通过就解释 EPC

### 回查页面

- [13-wannierization-and-effective-model-thinking.md](13-wannierization-and-effective-model-thinking.md)
- [14-electron-phonon-coupling-physical-judgement.md](14-electron-phonon-coupling-physical-judgement.md)

## excited states

### 必问问题

- 是否区分 ground-state DFT 与激发态方法？

### WARN 信号

- 光谱峰对 nbnd/broadening 敏感

### BLOCK 信号

- 用 KS eigenvalues 直接解释所有光谱

### 回查页面

- [15-spectroscopy-gw-bse-and-excited-state-boundaries.md](15-spectroscopy-gw-bse-and-excited-state-boundaries.md)
- [../learn/09-feature-expansion-map.md](../learn/09-feature-expansion-map.md)

## finite temperature

### 必问问题

- total energy、free energy 和 MD 采样是否区分？

### WARN 信号

- 能量差与误差同量级

### BLOCK 信号

- 0 K 能量直接代表实验温度稳定性

### 回查页面

- [16-thermodynamics-free-energy-and-md-boundaries.md](16-thermodynamics-free-energy-and-md-boundaries.md)
- [../workflows/advanced/molecular-dynamics.md](../workflows/advanced/molecular-dynamics.md)

## uncertainty

### 必问问题

- 数值、模型、赝势、边界和 workflow 误差是否分开记录？

### WARN 信号

- 只有单个数值没有误差来源

### BLOCK 信号

- 来源不可追踪仍写定量结论

### 回查页面

- [17-uncertainty-error-bars-and-reproducibility.md](17-uncertainty-error-bars-and-reproducibility.md)
- [../standards/calculation-record-template.md](../standards/calculation-record-template.md)


## 结论强度

- Strong：本页中由 DFT/DFPT 基础理论、经典方法论文或 QE 官方文档直接支撑的判断。
- Moderate：本页中由综述、方法摘要或多个来源共同支持，但细节需要回到正文或官方文档核验的判断。
- Boundary：本页中用于限制解释范围的判断，不作为定量结论。
- Version-sensitive：涉及 QE、PHonon、Wannier90、EPW、Yambo 或其他工具字段和行为的判断，必须按当前官方文档复查。

## 资料来源

- [standards/output-review-checklist.md](../standards/output-review-checklist.md)
- [standards/pass-warn-block.md](../standards/pass-warn-block.md)
- [references/source-index.md](../references/source-index.md)
