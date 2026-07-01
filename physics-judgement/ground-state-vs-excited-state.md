# Ground-State vs Excited-State Boundary

## 本页解决什么问题

本页用于判断普通 QE 基态计算能支持哪些电子结构表述，不能支持哪些激发态、quasiparticle 或 optical spectrum 结论。Kohn-Sham bands 和 DOS 是理解电子结构的重要对象，但它们不直接对应实验激发谱。

本页不承担完整 GW/BSE/TDDFT 教程功能；它用于防止把 ground-state workflow 输出越界解释为 quasiparticle gap、optical gap、exciton、absorption spectrum、XAS 或 EELS 结论。

## 典型 output 现象

| 现象 | 可能提示 | 不能直接说明什么 |
|---|---|---|
| bands 显示某个 Kohn-Sham gap | DFT-level electronic structure | 不能直接写成 experimental gap、quasiparticle gap 或 optical gap |
| DOS peak 出现在某能区 | state density 或 broadening 结果 | 不能直接写成 optical transition |
| hybrid / scissor 后 gap 改变 | 模型或后处理层级改变 | 不能替代完整 GW/BSE 边界说明 |
| absorption-like spectrum 对空带、window 或 broadening 敏感 | excited-state workflow 未收敛 | 不能只用一次输出给定量光谱 |
| Yambo/GW/BSE/TDDFT 正常结束 | 高级程序完成 | 不能免除空带、介电矩阵、k mesh、broadening 等审阅 |

## 可能原因

- 数值误差：k mesh、`nbnd`、empty states、energy window、broadening 或 dielectric cutoff 未收敛。
- 模型误差：ground-state functional、hybrid starting point、scissor 或 many-body approximation 改变解释层级。
- 赝势误差：PP、relativistic treatment 和 semicore choices 影响 bands、matrix elements 和 spectra。
- 边界条件误差：低维体系、charged state、surface/slab 或 finite-size effects 影响 excitation interpretation。
- 后处理误差：把 DOS/bands plot、broadened spectrum 或 scissor result 当作完整 excited-state method。
- workflow 传播误差：GW/BSE/TDDFT 输入读取旧 bands、旧 wavefunctions、旧 structure 或不一致的 `prefix/outdir`。
- 真实物理效应：quasiparticle correction、excitonic binding、spin-orbit splitting 或 local-field effects 确实改变谱学结论。

## 必查 QE input/output

| 对象 | 程序或文件 | 检查意义 |
|---|---|---|
| ground-state SCF | `pw.x` | 起点 density、functional、PP、SOC/U/vdW setting |
| bands / DOS / PDOS | `pw.x`、`bands.x`、`dos.x`、`projwfc.x` | 明确这是 KS-level electronic structure |
| `nbnd` / empty bands | `pw.x` input/output | excited-state workflow 常需更高能空态覆盖 |
| k mesh / k path | `K_POINTS` | 区分 integration、band path 和 optical/GW/BSE sampling |
| energy zero / Fermi level / VBM-CBM alignment | output / record | 防止不同数据链混用能量参考 |
| GW/BSE/TDDFT/Yambo 等输入 | external or QE ecosystem | 检查是否真正进入 excited-state 方法 |
| broadening / energy window / convergence | spectra output | 判断峰位和强度是否可解释 |

## 判断规则

1. Ground-state DFT 输出可以支持 DFT-level density、total energy、forces、stress 和 KS-level electronic structure 讨论，但不自动支持 quasiparticle 或 optical spectrum。
2. Kohn-Sham eigenvalue dispersion 可用于能带图和定性电子结构分析；定量 excitation claim 需要额外方法或明确模型边界。
3. Optical gap、quasiparticle gap、exciton 和 absorption spectrum 是不同对象，不能用一个 KS gap 替代。
4. Scissor correction、hybrid functional 或 broadened DOS 是边界化处理，不等同于完整 GW/BSE。
5. 若使用 Yambo、GW、BSE、TDDFT、XSpectra 或 EELS 工具，必须单独审阅其输入、收敛和版本文档。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | 结论明确限定为 DFT-level / KS-level，或已使用匹配目标的 excited-state 方法并完成独立审阅 | 可写对应层级结论 |
| WARN | 仅用 bands/DOS 作为趋势、初筛或物理图像 | 可做定性提示，不写定量谱学结论 |
| BLOCK | 用 ground-state bands/DOS 直接给 quasiparticle、optical、exciton、device-level excited-state 结论；混用能量参考或旧上游数据 | 停止越界解释，回到方法选择 |

## 不能做出的过度结论

- 不能把 Kohn-Sham gap 写成 optical gap。
- 不能把 DOS peak 写成 optical transition。
- 不能把 scissor correction 当作完整 BSE/GW。
- 不能把 ordinary DFT bands 当作 experimental spectrum。
- 不能在未说明 SOC、U、functional 和 PP 边界时做高风险 gap 或 topology 结论。

## 下游影响

- `workflows/electronic/bands.md`、`workflows/electronic/dos.md`、`workflows/electronic/pdos.md`：必须写明 KS-level 解释边界。
- `workflows/advanced/hybrid-functional-overview.md`：hybrid 是模型升级入口，不是所有 excited-state 问题的替代。
- `learn/09-feature-expansion-map.md`：GW/BSE/TDDFT/Yambo 应作为高级边界入口。
- `physics-judgement/uncertainty-statement-template.md`：gap、spectrum 和 optical statement 必须带方法层级。

## 与 theory-minimum / workflows / standards 的关系

- 最低理论回查：[theory-minimum/dft-ks-scf.md](../theory-minimum/dft-ks-scf.md)、[theory-minimum/kpoints-symmetry-kpath.md](../theory-minimum/kpoints-symmetry-kpath.md)。
- workflow 回查：[workflows/electronic/bands.md](../workflows/electronic/bands.md)、[workflows/electronic/dos.md](../workflows/electronic/dos.md)、[workflows/advanced/hybrid-functional-overview.md](../workflows/advanced/hybrid-functional-overview.md)。
- 记录规范：[standards/output-review-checklist.md](../standards/output-review-checklist.md)、[standards/calculation-record-template.md](../standards/calculation-record-template.md)。

## 来源与结论强度

| 结论 | 强度 | 来源边界 |
|---|---|---|
| HK/KS DFT 是 ground-state framework，KS 系统是辅助体系 | Strong | HK、KS、Kohn Nobel lecture |
| 准粒子或光学谱需要 GW/BSE/TDDFT 等额外方法 | Strong | Onida-Reining-Rubio review 与工具文档 |
| QE bands/DOS 可作为 KS-level electronic structure，不自动等同谱学实验 | Boundary | DFT theory boundary 与本仓库 electronic workflow |
| Yambo/GW/BSE/TDDFT 输入和限制需查当前官方文档 | Version-sensitive | Yambo / QE ecosystem docs |

## 资料来源

- Hohenberg and Kohn, *Inhomogeneous Electron Gas*：ground-state density 边界。
- Kohn and Sham, *Self-Consistent Equations Including Exchange and Correlation Effects*：KS 辅助体系。
- Kohn Nobel lecture：density-functional 方法边界。
- Onida, Reining and Rubio, electronic excitations review：GW/BSE 与 excited-state 边界。
- [Yambo educational wiki](https://wiki.yambo-code.eu/wiki/index.php?title=Main_Page)：Yambo 学习与工具入口。
- [workflows/electronic/bands.md](../workflows/electronic/bands.md)：本仓库 bands workflow。
