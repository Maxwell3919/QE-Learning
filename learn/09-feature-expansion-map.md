# Feature Expansion Map

本页不是学习顺序，也不是功能优先级排名。它只用于说明 QE 功能边界：哪些条目属于本仓库核心学习闭环，哪些条目只需要知道入口、前置依赖和适用边界。

阅读本页时不要按条目先后判断重要性。核心条目用于补齐 SCF、收敛、relax、电子结构、声子和后处理主线；高级条目只保留地图入口，不在本页扩写成完整教程。

每个条目固定回答五件事：

- QE 程序 / 外部工具；
- 核心或高级；
- 当前仓库页面；
- 最低学习目标；
- 推荐来源。

## NSCF

- QE 程序 / 外部工具：`pw.x`
- 核心或高级：核心
- 当前仓库页面：`workflows/ground-state/nscf.md`
- 最低学习目标：理解 NSCF 与 SCF 的数据关系，知道它如何服务 DOS、PDOS 和 bands。
- 推荐来源：QE INPUT_PW；Pranab Das QE tutorial。

## DOS

- QE 程序 / 外部工具：`dos.x`
- 核心或高级：核心
- 当前仓库页面：`workflows/electronic/dos.md`
- 最低学习目标：能复述 SCF -> NSCF -> DOS 的数据链，并知道 DOS 输出依赖能量网格、k mesh 和展宽设置。
- 推荐来源：QE documentation；Pranab Das QE tutorial。

## PDOS / projwfc.x

- QE 程序 / 外部工具：`projwfc.x`
- 核心或高级：核心
- 当前仓库页面：`workflows/electronic/pdos.md`
- 最低学习目标：理解投影态密度、原子/轨道标签和总 DOS 的关系。
- 推荐来源：QE documentation；Pranab Das QE tutorial。

## charge density / pp.x

- QE 程序 / 外部工具：`pp.x`
- 核心或高级：核心
- 当前仓库页面：`workflows/electronic/charge-density.md`
- 最低学习目标：知道如何从已收敛 SCF 状态提取电荷密度，并能说明输出数据来自哪里。
- 推荐来源：QE INPUT_PP；QE PostProc documentation。

## electrostatic potential

- QE 程序 / 外部工具：`pp.x`, `average.x`
- 核心或高级：高级入口
- 当前仓库页面：`workflows/electronic/electrostatic-potential.md`
- 最低学习目标：理解电势输出、平面平均和结构模型前提；不在本页展开 slab 或界面案例。
- 推荐来源：QE INPUT_PP；QE PostProc documentation。

## ELF

- QE 程序 / 外部工具：`pp.x`
- 核心或高级：高级入口
- 当前仓库页面：`workflows/electronic/elf.md`
- 最低学习目标：知道 ELF 是从已收敛电子结构后处理得到的空间场，不把可视化图像本身当作结论。
- 推荐来源：QE INPUT_PP；QE PostProc documentation。

## work function

- QE 程序 / 外部工具：`pp.x`, `average.x`
- 核心或高级：高级入口
- 当前仓库页面：`workflows/electronic/work-function.md`
- 最低学习目标：理解功函数依赖 slab 模型、电势平台和费米能级；本页只标出入口，不展开材料案例。
- 推荐来源：QE INPUT_PP；QE PostProc documentation。

## Fermi surface / fs.x

- QE 程序 / 外部工具：`fs.x`, XCrySDen
- 核心或高级：高级入口
- 当前仓库页面：`workflows/electronic/fermi-surface.md`
- 最低学习目标：知道 BXSF 文件、稠密 k mesh 和可视化工具之间的数据关系。
- 推荐来源：QE PostProc documentation；XCrySDen documentation。

## cutoff convergence

- QE 程序 / 外部工具：`pw.x`
- 核心或高级：核心
- 当前仓库页面：`workflows/ground-state/cutoff-convergence.md`
- 最低学习目标：比较 cutoff 对目标量的影响，并能说明收敛判断基于哪个输出量。
- 推荐来源：QE INPUT_PW；Pranab Das QE tutorial。

## ecutrho

- QE 程序 / 外部工具：`pw.x`
- 核心或高级：核心
- 当前仓库页面：`workflows/ground-state/cutoff-convergence.md`
- 最低学习目标：说明 `ecutrho` 与赝势类型、charge density cutoff 和收敛测试的关系。
- 推荐来源：QE INPUT_PW；QE pseudopotential documentation。

## k-point convergence

- QE 程序 / 外部工具：`pw.x`
- 核心或高级：核心
- 当前仓库页面：`workflows/ground-state/kpoint-convergence.md`
- 最低学习目标：比较 k mesh 对能量、力、应力或电子结构输出的影响。
- 推荐来源：QE INPUT_PW；Pranab Das QE tutorial。

## smearing / occupations

- QE 程序 / 外部工具：`pw.x`
- 核心或高级：核心
- 当前仓库页面：`workflows/ground-state/smearing-and-occupations.md`
- 最低学习目标：解释金属占据、smearing 类型和展宽参数对 SCF 与 DOS 的影响。
- 推荐来源：QE INPUT_PW；theory-minimum/smearing-metals.md。

## mixing

- QE 程序 / 外部工具：`pw.x`
- 核心或高级：核心
- 当前仓库页面：`workflows/ground-state/scf.md`
- 最低学习目标：能从 output 判断 SCF 是否振荡或不收敛，并知道 mixing 参数属于诊断入口。
- 推荐来源：QE INPUT_PW。

## force / stress convergence

- QE 程序 / 外部工具：`pw.x`
- 核心或高级：核心
- 当前仓库页面：`workflows/ground-state/relax.md`
- 最低学习目标：读懂 force、stress 和最终结构输出，并判断是否允许进入下游计算。
- 推荐来源：QE INPUT_PW；theory-minimum/force-stress-relaxation.md。

## phonon convergence

- QE 程序 / 外部工具：`ph.x`
- 核心或高级：核心
- 当前仓库页面：`workflows/phonon/phonon-debugging.md`
- 最低学习目标：理解 phonon 对结构、SCF 收敛、q-grid 和数值阈值的敏感性。
- 推荐来源：QE INPUT_PH；QE PHonon user guide；Kyoto QE phonon DokuWiki。

## relax

- QE 程序 / 外部工具：`pw.x`
- 核心或高级：核心
- 当前仓库页面：`workflows/ground-state/relax.md`
- 最低学习目标：完成原子位置优化的 output review，说明力阈值和最终结构是否可信。
- 推荐来源：QE INPUT_PW；theory-minimum/force-stress-relaxation.md。

## vc-relax

- QE 程序 / 外部工具：`pw.x`
- 核心或高级：核心
- 当前仓库页面：`workflows/ground-state/vc-relax.md`
- 最低学习目标：完成晶胞优化的 output review，区分变胞优化、最终结构和后续 static SCF。
- 推荐来源：QE INPUT_PW；theory-minimum/force-stress-relaxation.md。

## molecular dynamics

- QE 程序 / 外部工具：`pw.x`, `cp.x`
- 核心或高级：高级入口
- 当前仓库页面：`workflows/advanced/molecular-dynamics.md`
- 最低学习目标：知道 QE 中 MD 与静态 SCF/relax 的输入输出边界；本页不展开采样教程。
- 推荐来源：QE PW documentation；QE CP documentation。

## Car-Parrinello / cp.x

- QE 程序 / 外部工具：`cp.x`
- 核心或高级：高级入口
- 当前仓库页面：`workflows/advanced/molecular-dynamics.md`
- 最低学习目标：知道 CP 与 PWscf 主线不同，入口放在 MD 高级主题下。
- 推荐来源：QE CP documentation。

## NEB / neb.x

- QE 程序 / 外部工具：`neb.x`
- 核心或高级：高级入口
- 当前仓库页面：`workflows/advanced/neb-reaction-path.md`
- 最低学习目标：理解路径计算依赖端点结构、images 和收敛判断；本页不展开反应案例。
- 推荐来源：QE PWneb documentation。

## spin polarization

- QE 程序 / 外部工具：`pw.x`
- 核心或高级：高级入口
- 当前仓库页面：`workflows/advanced/spin-polarization.md`
- 最低学习目标：理解自旋极化输入、初始磁矩和输出磁矩的基本边界。
- 推荐来源：QE INPUT_PW；theory-minimum/magnetism-soc-dftu.md。

## noncollinear magnetism

- QE 程序 / 外部工具：`pw.x`
- 核心或高级：高级入口
- 当前仓库页面：`workflows/advanced/noncollinear-soc.md`
- 最低学习目标：知道非共线磁性需要独立输入设置和更严格的赝势/对称性检查。
- 推荐来源：QE INPUT_PW；theory-minimum/magnetism-soc-dftu.md。

## SOC

- QE 程序 / 外部工具：`pw.x`
- 核心或高级：高级入口
- 当前仓库页面：`workflows/advanced/noncollinear-soc.md`
- 最低学习目标：理解 SOC 与赝势、非共线设置和能带解释的关系。
- 推荐来源：QE INPUT_PW；theory-minimum/magnetism-soc-dftu.md。

## DFT+U

- QE 程序 / 外部工具：`pw.x`
- 核心或高级：高级入口
- 当前仓库页面：`workflows/advanced/dft-plus-u.md`
- 最低学习目标：理解 Hubbard 参数必须记录来源、版本和作用轨道；本页不提供材料参数建议。
- 推荐来源：QE INPUT_PW；theory-minimum/magnetism-soc-dftu.md。

## DFT+U+V

- QE 程序 / 外部工具：`pw.x`, `hp.x`
- 核心或高级：高级入口
- 当前仓库页面：`workflows/advanced/dft-plus-u.md`
- 最低学习目标：知道扩展 Hubbard workflow 和 `hp.x` 入口，不把它并入基础 DFT+U 教程。
- 推荐来源：QE documentation；QE INPUT_PW。

## vdW corrections

- QE 程序 / 外部工具：`pw.x`
- 核心或高级：高级入口
- 当前仓库页面：`workflows/advanced/vdW-corrections.md`
- 最低学习目标：理解色散修正的适用边界和参数记录要求。
- 推荐来源：QE INPUT_PW。

## hybrid functional overview

- QE 程序 / 外部工具：`pw.x`
- 核心或高级：高级入口
- 当前仓库页面：`workflows/advanced/hybrid-functional-overview.md`
- 最低学习目标：理解 hybrid functional 的计算成本、输入边界和与常规 SCF 的差异。
- 推荐来源：QE INPUT_PW。

## Wannier90 interface

- QE 程序 / 外部工具：`pw2wannier90.x`, Wannier90
- 核心或高级：高级入口
- 当前仓库页面：`workflows/advanced/wannier90-overview.md`
- 最低学习目标：理解 QE 到 Wannier90 的数据交接、投影设置和后续插值入口。
- 推荐来源：Wannier90 documentation；QE INPUT_pw2wannier90；references/tools/wannier90.md。

## Gamma phonon

- QE 程序 / 外部工具：`ph.x`, `dynmat.x`
- 核心或高级：核心
- 当前仓库页面：`workflows/phonon/gamma-phonon.md`
- 最低学习目标：区分 Gamma 点响应、频率输出、声学模检查和 q-grid dispersion。
- 推荐来源：QE INPUT_PH；QE PHonon user guide；Kyoto QE phonon DokuWiki。

## q-grid phonon

- QE 程序 / 外部工具：`ph.x`, `q2r.x`, `matdyn.x`
- 核心或高级：核心
- 当前仓库页面：`workflows/phonon/phonon-dispersion-dfpt.md`
- 最低学习目标：完成 DFPT dispersion 的数据链：dynamical matrices -> IFC -> dispersion。
- 推荐来源：QE INPUT_PH；QE INPUT_Q2R；QE INPUT_MATDYN；Pranab Das QE tutorial；Kyoto QE phonon DokuWiki。

## phonon DOS

- QE 程序 / 外部工具：`matdyn.x`
- 核心或高级：核心
- 当前仓库页面：`workflows/phonon/phonon-dos.md`
- 最低学习目标：从 IFC 生成 phonon DOS，并说明 q mesh 与 DOS 平滑的关系。
- 推荐来源：QE INPUT_MATDYN；Kyoto QE phonon DokuWiki。

## dielectric tensor

- QE 程序 / 外部工具：`ph.x`
- 核心或高级：高级入口
- 当前仓库页面：`workflows/phonon/dielectric-born-effective-charge.md`
- 最低学习目标：理解 `epsil` 相关条件和绝缘体前提；本页不展开材料解释。
- 推荐来源：QE INPUT_PH；theory-minimum/dielectric-born-charge.md。

## Born effective charge

- QE 程序 / 外部工具：`ph.x`
- 核心或高级：高级入口
- 当前仓库页面：`workflows/phonon/dielectric-born-effective-charge.md`
- 最低学习目标：理解 Born effective charge 的响应量含义、绝缘体前提和 output 位置。
- 推荐来源：QE INPUT_PH；Kyoto QE phonon DokuWiki；theory-minimum/dielectric-born-charge.md。

## LO-TO splitting

- QE 程序 / 外部工具：`ph.x`, `matdyn.x`
- 核心或高级：高级入口
- 当前仓库页面：`workflows/phonon/dielectric-born-effective-charge.md`
- 最低学习目标：知道非解析项、介电张量和 Born effective charge 是 LO-TO splitting 的前提。
- 推荐来源：QE PHonon user guide；QE INPUT_PH；QE INPUT_MATDYN。

## IR / Raman

- QE 程序 / 外部工具：`ph.x`, `dynmat.x`
- 核心或高级：高级入口
- 当前仓库页面：`workflows/phonon/ir-raman.md`
- 最低学习目标：理解 IR/Raman 是响应性质入口，需要先确认结构、SCF 和 phonon 输出可信。
- 推荐来源：QE PHonon documentation；QE INPUT_PH。

## electron-phonon overview

- QE 程序 / 外部工具：`ph.x`, EPW
- 核心或高级：高级入口
- 当前仓库页面：`workflows/advanced/electron-phonon-overview.md`
- 最低学习目标：知道 EPC 依赖可靠电子结构、声子和 k/q 网格；本页不展开公式或材料案例。
- 推荐来源：EPW documentation；references/tools/epw.md。

## EPW overview

- QE 程序 / 外部工具：EPW, Wannier90
- 核心或高级：高级入口
- 当前仓库页面：`workflows/advanced/electron-phonon-overview.md`
- 最低学习目标：理解 EPW 与 Wannier interpolation、k/q 网格和 EPC 输出的关系。
- 推荐来源：EPW documentation；Wannier90 documentation；references/tools/epw.md。

## XSpectra

- QE 程序 / 外部工具：`xspectra.x`
- 核心或高级：高级入口
- 当前仓库页面：无专页，仅保留本地图入口。
- 最低学习目标：知道 XSpectra 属于专用谱学包，不纳入基础 workflow。
- 推荐来源：QE XSPECTRA documentation。

## TurboTDDFT

- QE 程序 / 外部工具：`turbo_*`
- 核心或高级：高级入口
- 当前仓库页面：无专页，仅保留本地图入口。
- 最低学习目标：知道 TurboTDDFT 属于 TDDFPT 分支，不纳入基础 workflow。
- 推荐来源：QE TDDFPT documentation。

## TurboEELS

- QE 程序 / 外部工具：`turbo_eels.x`
- 核心或高级：高级入口
- 当前仓库页面：无专页，仅保留本地图入口。
- 最低学习目标：知道 TurboEELS 是 EELS 专用入口，不纳入基础 workflow。
- 推荐来源：QE TDDFPT documentation。

## GWL / GW overview

- QE 程序 / 外部工具：GWL, YAMBO
- 核心或高级：高级入口
- 当前仓库页面：无专页，仅保留本地图入口。
- 最低学习目标：知道 GW/BSE 属于高级准粒子/激发态方向，不属于基础 QE 主线。
- 推荐来源：QE documentation；YAMBO documentation。

## PWcond

- QE 程序 / 外部工具：`pwcond.x`
- 核心或高级：高级入口
- 当前仓库页面：无专页，仅保留本地图入口。
- 最低学习目标：知道 PWcond 是输运分支入口，不纳入基础 workflow。
- 推荐来源：QE PWcond documentation。

## GIPAW

- QE 程序 / 外部工具：GIPAW
- 核心或高级：高级入口
- 当前仓库页面：无专页，仅保留本地图入口。
- 最低学习目标：知道 GIPAW 属于磁响应/谱学分支，不纳入基础 workflow。
- 推荐来源：QE documentation。

## KCW / Koopmans-compliant workflows

- QE 程序 / 外部工具：KCW
- 核心或高级：高级入口
- 当前仓库页面：无专页，仅保留本地图入口。
- 最低学习目标：知道 KCW 是 Koopmans-compliant 扩展入口，不纳入基础 workflow。
- 推荐来源：QE documentation。

## Environ / ESM / RISM

- QE 程序 / 外部工具：Environ, ESM, RISM
- 核心或高级：高级入口
- 当前仓库页面：无专页，仅保留本地图入口。
- 最低学习目标：知道这些工具服务溶剂、电化学或边界条件扩展，不纳入基础 workflow。
- 推荐来源：QE documentation；Environ documentation。

## PLUMED

- QE 程序 / 外部工具：PLUMED
- 核心或高级：高级入口
- 当前仓库页面：无专页，仅保留本地图入口。
- 最低学习目标：知道 PLUMED 用于增强采样等高级 MD 场景，不纳入基础 workflow。
- 推荐来源：PLUMED documentation；QE documentation。

## pseudopotential selection

- QE 程序 / 外部工具：UPF, SSSP, PseudoDojo
- 核心或高级：核心
- 当前仓库页面：`theory-minimum/pseudopotentials.md`; `workflows/advanced/pseudopotential-generation-overview.md`
- 最低学习目标：能记录赝势来源、版本、推荐 cutoff 和选择理由。
- 推荐来源：QE pseudopotential documentation；SSSP；PseudoDojo。

## UPF 文件理解

- QE 程序 / 外部工具：UPF
- 核心或高级：核心
- 当前仓库页面：`theory-minimum/pseudopotentials.md`
- 最低学习目标：能读基本赝势元信息，并知道文件名不能替代 provenance 记录。
- 推荐来源：QE pseudopotential documentation。

## pseudopotential validation

- QE 程序 / 外部工具：SSSP-style tests
- 核心或高级：核心
- 当前仓库页面：`workflows/advanced/pseudopotential-generation-overview.md`
- 最低学习目标：理解赝势验证需要测试目标量和收敛行为，不只看文件名或来源标签。
- 推荐来源：SSSP；PseudoDojo；QE pseudopotential documentation。

## pseudopotential generation / atomic / ld1.x

- QE 程序 / 外部工具：`atomic`, `ld1.x`
- 核心或高级：高级入口
- 当前仓库页面：`workflows/advanced/pseudopotential-generation-overview.md`
- 最低学习目标：知道自制赝势属于高级主题，需要独立验证闭环；本页不展开生成教程。
- 推荐来源：QE atomic documentation；QE pseudopotential documentation。
