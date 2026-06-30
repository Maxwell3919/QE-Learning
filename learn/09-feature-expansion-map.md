# Feature Expansion Map

本页说明 QE 支持但本仓库尚未详细展开的功能。高级包只做功能地图，不写成主线教程。

## NSCF

- QE 程序 / 外部工具：`pw.x`
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/ground-state/nscf.md`
- 推荐参考资料：QE INPUT_PW
- 最低学习目标：理解 NSCF 与 SCF 的数据关系

## DOS

- QE 程序 / 外部工具：`dos.x`
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/electronic/dos.md`
- 推荐参考资料：QE docs, Pranab tutorial
- 最低学习目标：完成 SCF -> NSCF -> DOS

## PDOS / projwfc.x

- QE 程序 / 外部工具：`projwfc.x`
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/electronic/pdos.md`
- 推荐参考资料：QE docs, Pranab tutorial
- 最低学习目标：理解投影态密度和轨道标签

## charge density / pp.x

- QE 程序 / 外部工具：`pp.x`
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/electronic/charge-density.md`
- 推荐参考资料：QE INPUT_PP
- 最低学习目标：从 SCF 状态提取电荷密度

## electrostatic potential

- QE 程序 / 外部工具：`pp.x`, `average.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/electronic/electrostatic-potential.md`
- 推荐参考资料：QE PostProc docs
- 最低学习目标：理解电势输出和平均方法

## ELF

- QE 程序 / 外部工具：`pp.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/electronic/elf.md`
- 推荐参考资料：QE INPUT_PP
- 最低学习目标：生成并解释 ELF 数据

## work function

- QE 程序 / 外部工具：`pp.x`, `average.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/electronic/work-function.md`
- 推荐参考资料：QE PostProc docs
- 最低学习目标：理解 slab 电势平台与功函数

## Fermi surface / fs.x

- QE 程序 / 外部工具：`fs.x`, XCrySDen
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/electronic/fermi-surface.md`
- 推荐参考资料：QE PostProc docs
- 最低学习目标：生成 BXSF 并理解费米面

## cutoff convergence

- QE 程序 / 外部工具：`pw.x`
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/ground-state/cutoff-convergence.md`
- 推荐参考资料：QE INPUT_PW
- 最低学习目标：比较 cutoff 对目标量的影响

## ecutrho

- QE 程序 / 外部工具：`pw.x`
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/ground-state/cutoff-convergence.md`
- 推荐参考资料：QE INPUT_PW
- 最低学习目标：说明 `ecutrho` 与赝势类型关系

## k-point convergence

- QE 程序 / 外部工具：`pw.x`
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/ground-state/kpoint-convergence.md`
- 推荐参考资料：QE INPUT_PW
- 最低学习目标：比较 k mesh 对输出的影响

## smearing / occupations

- QE 程序 / 外部工具：`pw.x`
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/ground-state/smearing-and-occupations.md`
- 推荐参考资料：QE INPUT_PW
- 最低学习目标：解释金属占据和 smearing

## mixing

- QE 程序 / 外部工具：`pw.x`
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/ground-state/scf.md`
- 推荐参考资料：QE INPUT_PW
- 最低学习目标：诊断 SCF 振荡和 mixing 参数

## force / stress convergence

- QE 程序 / 外部工具：`pw.x`
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/ground-state/relax.md`
- 推荐参考资料：QE INPUT_PW
- 最低学习目标：读 force/stress 并判断优化质量

## phonon convergence

- QE 程序 / 外部工具：`ph.x`
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/phonon/phonon-debugging.md`
- 推荐参考资料：QE PHonon guide
- 最低学习目标：理解 phonon 对 SCF/结构/q-grid 敏感性

## relax

- QE 程序 / 外部工具：`pw.x`
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/ground-state/relax.md`
- 推荐参考资料：QE INPUT_PW
- 最低学习目标：完成原子位置优化判断

## vc-relax

- QE 程序 / 外部工具：`pw.x`
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/ground-state/vc-relax.md`
- 推荐参考资料：QE INPUT_PW
- 最低学习目标：完成晶胞优化判断

## molecular dynamics

- QE 程序 / 外部工具：`pw.x`, `cp.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/advanced/molecular-dynamics.md`
- 推荐参考资料：QE PW/CP docs
- 最低学习目标：理解 MD 输入和输出边界

## Car-Parrinello / cp.x

- QE 程序 / 外部工具：`cp.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：仅保留地图
- 推荐放置页面：`workflows/advanced/molecular-dynamics.md`
- 推荐参考资料：QE CP docs
- 最低学习目标：知道 CP 与 PWscf 主线不同

## NEB / neb.x

- QE 程序 / 外部工具：`neb.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：已有简页
- 推荐放置页面：`workflows/advanced/neb-reaction-path.md`
- 推荐参考资料：QE PWneb docs
- 最低学习目标：理解路径计算依赖端点结构

## spin polarization

- QE 程序 / 外部工具：`pw.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/advanced/spin-polarization.md`
- 推荐参考资料：QE INPUT_PW
- 最低学习目标：理解自旋极化输入

## noncollinear magnetism

- QE 程序 / 外部工具：`pw.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/advanced/noncollinear-soc.md`
- 推荐参考资料：QE INPUT_PW
- 最低学习目标：理解非共线输入边界

## SOC

- QE 程序 / 外部工具：`pw.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/advanced/noncollinear-soc.md`
- 推荐参考资料：QE INPUT_PW
- 最低学习目标：理解 SOC 与赝势要求

## DFT+U

- QE 程序 / 外部工具：`pw.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/advanced/dft-plus-u.md`
- 推荐参考资料：QE INPUT_PW
- 最低学习目标：理解 Hubbard 参数记录

## DFT+U+V

- QE 程序 / 外部工具：`pw.x`, `hp.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：仅保留地图
- 推荐放置页面：`workflows/advanced/dft-plus-u.md`
- 推荐参考资料：QE docs
- 最低学习目标：知道扩展 Hubbard workflow

## vdW corrections

- QE 程序 / 外部工具：`pw.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/advanced/vdW-corrections.md`
- 推荐参考资料：QE INPUT_PW
- 最低学习目标：理解色散修正适用场景

## hybrid functional overview

- QE 程序 / 外部工具：`pw.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/advanced/hybrid-functional-overview.md`
- 推荐参考资料：QE INPUT_PW
- 最低学习目标：理解 hybrid 成本和输入边界

## Wannier90 interface

- QE 程序 / 外部工具：`pw2wannier90.x`, Wannier90
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/advanced/wannier90-overview.md`
- 推荐参考资料：Wannier90 docs, QE INPUT_pw2wannier90
- 最低学习目标：理解 QE 到 Wannier 的数据交接

## Gamma phonon

- QE 程序 / 外部工具：`ph.x`, `dynmat.x`
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/phonon/gamma-phonon.md`
- 推荐参考资料：QE INPUT_PH, Kyoto DokuWiki
- 最低学习目标：区分 Gamma 响应和 q-grid

## q-grid phonon

- QE 程序 / 外部工具：`ph.x`, `q2r.x`, `matdyn.x`
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：已有
- 推荐放置页面：`workflows/phonon/phonon-dispersion-dfpt.md`
- 推荐参考资料：Pranab, Kyoto, QE PHonon
- 最低学习目标：完成 DFPT dispersion 数据链

## phonon DOS

- QE 程序 / 外部工具：`matdyn.x`
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/phonon/phonon-dos.md`
- 推荐参考资料：Kyoto DokuWiki
- 最低学习目标：从 IFC 生成 phonon DOS

## dielectric tensor

- QE 程序 / 外部工具：`ph.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/phonon/dielectric-born-effective-charge.md`
- 推荐参考资料：QE INPUT_PH
- 最低学习目标：理解 `epsil` 条件

## Born effective charge

- QE 程序 / 外部工具：`ph.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/phonon/dielectric-born-effective-charge.md`
- 推荐参考资料：QE INPUT_PH, Kyoto
- 最低学习目标：理解极性材料响应

## LO-TO splitting

- QE 程序 / 外部工具：`ph.x`, `matdyn.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/phonon/dielectric-born-effective-charge.md`
- 推荐参考资料：QE PHonon guide
- 最低学习目标：知道非解析项条件

## IR / Raman

- QE 程序 / 外部工具：`ph.x`, `dynmat.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/phonon/ir-raman.md`
- 推荐参考资料：QE PHonon docs
- 最低学习目标：理解响应量适用条件

## electron-phonon overview

- QE 程序 / 外部工具：`ph.x`, EPW
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/advanced/electron-phonon-overview.md`
- 推荐参考资料：EPW docs
- 最低学习目标：知道 EPC 前置条件

## EPW overview

- QE 程序 / 外部工具：EPW, Wannier90
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：仅保留地图
- 推荐放置页面：`workflows/advanced/electron-phonon-overview.md`
- 推荐参考资料：EPW docs
- 最低学习目标：理解 EPC/Wannier/q-grid 关系

## XSpectra

- QE 程序 / 外部工具：`xspectra.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：仅保留地图
- 推荐放置页面：无
- 推荐参考资料：QE XSPECTRA docs
- 最低学习目标：知道属于专用谱学包

## TurboTDDFT

- QE 程序 / 外部工具：`turbo_*`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：仅保留地图
- 推荐放置页面：无
- 推荐参考资料：QE TDDFPT docs
- 最低学习目标：知道属于 TDDFPT 分支

## TurboEELS

- QE 程序 / 外部工具：`turbo_eels.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：仅保留地图
- 推荐放置页面：无
- 推荐参考资料：QE TDDFPT docs
- 最低学习目标：知道 EELS 分支

## GWL / GW overview

- QE 程序 / 外部工具：GWL, YAMBO
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：仅保留地图
- 推荐放置页面：无
- 推荐参考资料：QE docs, YAMBO docs
- 最低学习目标：知道 GW/BSE 不属基础主线

## PWcond

- QE 程序 / 外部工具：`pwcond.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：仅保留地图
- 推荐放置页面：无
- 推荐参考资料：QE PWcond docs
- 最低学习目标：知道输运分支

## GIPAW

- QE 程序 / 外部工具：GIPAW
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：仅保留地图
- 推荐放置页面：无
- 推荐参考资料：QE docs
- 最低学习目标：知道磁响应/谱学分支

## KCW / Koopmans-compliant workflows

- QE 程序 / 外部工具：KCW
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：仅保留地图
- 推荐放置页面：无
- 推荐参考资料：QE docs
- 最低学习目标：知道 Koopmans 扩展

## Environ / ESM / RISM

- QE 程序 / 外部工具：Environ, ESM, RISM
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：仅保留地图
- 推荐放置页面：无
- 推荐参考资料：QE docs
- 最低学习目标：知道溶剂/电化学边界条件分支

## PLUMED

- QE 程序 / 外部工具：PLUMED
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：仅保留地图
- 推荐放置页面：无
- 推荐参考资料：PLUMED/QE docs
- 最低学习目标：知道增强采样分支

## pseudopotential selection

- QE 程序 / 外部工具：UPF, SSSP, PseudoDojo
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/advanced/pseudopotential-generation-overview.md`
- 推荐参考资料：QE pseudopotentials
- 最低学习目标：能记录赝势来源和选择理由

## UPF 文件理解

- QE 程序 / 外部工具：UPF
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：需要补
- 推荐放置页面：`theory-minimum/pseudopotentials.md`
- 推荐参考资料：QE pseudopotentials
- 最低学习目标：能读基本赝势元信息

## pseudopotential validation

- QE 程序 / 外部工具：SSSP-style tests
- 属于核心学习还是高级扩展：核心学习
- 当前仓库状态：需要补
- 推荐放置页面：`workflows/advanced/pseudopotential-generation-overview.md`
- 推荐参考资料：SSSP/PseudoDojo
- 最低学习目标：理解验证不是只看文件名

## pseudopotential generation / atomic / ld1.x

- QE 程序 / 外部工具：`atomic`, `ld1.x`
- 属于核心学习还是高级扩展：高级扩展
- 当前仓库状态：仅保留地图
- 推荐放置页面：`workflows/advanced/pseudopotential-generation-overview.md`
- 推荐参考资料：QE atomic docs
- 最低学习目标：知道自制赝势属于高级主题
