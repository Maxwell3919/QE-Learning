# vdW and Low-Dimensional Boundary

## 本页解决什么问题

本页用于判断含弱相互作用、层状、表面、界面、吸附和低维周期模型的 DFT/QE 结果是否具备可解释性。核心审阅点不在于是否打开 vdW；需要确认所选 exchange-correlation 模型、vdW 修正、边界条件、结构优化和下游后处理是否属于同一条可复查的数据链。

vdW 与低维问题容易把模型误差、边界条件误差和数值收敛误差混在一起。层间距、吸附能、表面电势、work function、phonon soft mode 和能带变化都可能来自物理弱相互作用，也可能来自真空、偶极、cell 约束或下游文件链不一致。vdW 修正改变的是能量面和结构响应；低维边界改变的是周期镜像、长程静电和电势参考。二者常耦合，但不能互相替代审阅。

## 典型 output 现象

| 现象 | 可能提示 | 不能直接说明什么 |
|---|---|---|
| relax 后层间距、吸附高度或界面距离对模型选择敏感 | vdW / functional choice 主导几何 | 不能说明某个 vdW 模型在所有 observable 上更可靠 |
| force 已收敛但 stress 或 cell 参数随 cutoff/k mesh 变化 | 数值收敛与弱相互作用能量尺度接近 | 不能只用 `JOB DONE` 证明结构可用于 phonon 或 work function |
| work function 或 electrostatic potential 随真空区域变化 | slab electrostatics 或偶极边界未稳 | 不能从单张 potential 图给定量结论 |
| phonon soft branch 对 vdW 模型或 cell 参数敏感 | 能量面由弱相互作用和结构共同控制 | 不能只用 ASR 或平滑图像排除不稳定 |
| bands/DOS 与 relax 所用 functional 或 vdW 设置不一致 | workflow 传播误差 | 不能把混合模型结果作为同一物理模型结论 |

## 可能原因

- 数值误差：`ecutwfc/ecutrho`、k mesh、force/stress 收敛不足，导致弱相互作用能量差被数值噪声淹没。
- 模型误差：semilocal functional 对长程 dispersion 的描述有限；`vdw_corr`、nonlocal functional 或 semi-empirical correction 改变能量面。
- 赝势误差：pseudopotential 与 `input_dft`、vdW 模型、relativistic treatment 或 cutoff 要求不一致。
- 边界条件误差：低维模型的真空区、周期镜像、偶极场、cell 约束或孤立边界处理不充分。
- 后处理误差：只看可视化结构、电荷差分图或 potential 曲线，没有回到 SCF、relax 和 output review。
- workflow 传播误差：relax 使用一种模型，下游 bands/phonon/work function 使用另一种模型，或复用了旧 `prefix/outdir`。
- 真实物理效应：弱结合、层间滑移、表面极化、界面偶极或吸附诱导电荷重排确实改变结构和电子性质。

## 必查 QE input/output

| 对象 | 程序或文件 | 检查意义 |
|---|---|---|
| `input_dft` | `pw.x` | 确认 exchange-correlation 模型，与 pseudopotential 来源一致 |
| `vdw_corr` 及相关 vdW 字段 | `pw.x` | 确认 vdW 模型和版本支持范围，不能只写“加了 vdW” |
| `ATOMIC_SPECIES` / `.UPF` | `pw.x` input/output | 确认 PP provenance、functional、relativistic treatment 和 cutoff 建议 |
| `CELL_PARAMETERS`、结构约束、vacuum 方向 | `pw.x` input/output | 检查低维边界和 cell 自由度是否匹配目标问题 |
| `tefield`、`dipfield`、`assume_isolated`、ESM 相关设置 | `pw.x` | 表面/低维 electrostatics 的版本敏感入口 |
| forces、stress、pressure、final coordinates | `pw.x` output | 判断结构优化是否足以进入下游 |
| `prefix/outdir` | 全链条 | 确认 downstream bands/phonon/work function 读取同一模型数据 |
| warning 与 symmetry 信息 | output | 检查边界、对称性或文件链风险 |

## 判断规则

1. vdW 设置属于模型选择。它不能通过单纯加密 cutoff 或 k mesh 被“收敛”出来。
2. 对层间距、吸附、界面结合或低维 work function 的定量判断，必须同时说明 functional、vdW 模型、PP、结构状态、边界条件和下游复算链。
3. 若改变 vdW 模型导致 relaxed structure 明显变化，下游 bands、DOS、phonon、potential 和 work function 应基于同一最终结构与同一模型重新计算。
4. 低维体系的 electrostatic boundary 是独立审阅项。真空区、电势平台、偶极修正、isolated-boundary 设置和周期镜像不能由 total energy alone 代替。
5. 弱相互作用结论应写成模型内判断，除非完成了足够的模型敏感性、数值收敛和实验/高阶方法对照。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | `input_dft`、`vdw_corr`、PP、结构优化、vacuum/boundary、force/stress、下游文件链均可追溯，并在结论中写明模型边界 | 可进入同一模型下的 bands、phonon、work function 或趋势讨论 |
| WARN | 数据链可追踪，但 vdW/functional sensitivity、低维边界或 stress 收敛不完整 | 只允许写探索性趋势，不写强定量比较 |
| BLOCK | 未说明 vdW 模型却给出层间距/吸附/界面定量结论；vdW 与非 vdW 下游混用；低维 electrostatics 未审阅；旧 scratch 与新模型混用 | 停止正式结论，回到结构、边界和文件链审查 |

## 不能做出的过度结论

- 不能把某个 vdW correction 写成对所有性质都更可靠的模型。
- 不能只凭单点 total energy 差异给出弱相互作用定量结论。
- 不能忽略 vacuum、dipole 和 surface termination 对 work function 与 potential 的影响。
- 不能把 relax 已收敛等同于 phonon 或 electronic workflow 已可信。
- 不能跨不同 functional、vdW 模型、PP 或结构状态混合比较 total energy、band gap 或 phonon。
- 不能把低维 electrostatics 问题归因于 vdW 模型，或把 vdW 修正当作真空/偶极边界处理。

## 下游影响

- `workflows/advanced/vdW-corrections.md`：决定 vdW 设置是否只是边界页入口，还是已经足以进入结构/能量判断。
- `workflows/ground-state/relax.md` 与 `workflows/ground-state/vc-relax.md`：影响 force、stress、cell 和 final static SCF 的准入。
- `workflows/electronic/electrostatic-potential.md` 与 `workflows/electronic/work-function.md`：低维 electrostatics 会直接改变 vacuum level 和 work function。
- `workflows/phonon/phonon-dispersion-dfpt.md`：结构和能量面变化会传递到 soft modes 和 dispersion。
- publication record：需要把模型选择写成方法边界，而不是只列最终数值。

## 与 theory-minimum / workflows / standards 的关系

- 最低理论回查：[theory-minimum/pseudopotentials.md](../theory-minimum/pseudopotentials.md)、[theory-minimum/force-stress-relaxation.md](../theory-minimum/force-stress-relaxation.md)、[theory-minimum/kpoints-symmetry-kpath.md](../theory-minimum/kpoints-symmetry-kpath.md)。
- workflow 回查：[workflows/advanced/vdW-corrections.md](../workflows/advanced/vdW-corrections.md)、[workflows/electronic/work-function.md](../workflows/electronic/work-function.md)、[workflows/electronic/electrostatic-potential.md](../workflows/electronic/electrostatic-potential.md)。
- 记录规范：[standards/output-review-checklist.md](../standards/output-review-checklist.md)、[standards/calculation-record-template.md](../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../standards/pass-warn-block.md)。

## 来源与结论强度

| 结论 | 强度 | 来源边界 |
|---|---|---|
| semilocal functional 对长程 dispersion 存在模型限制 | Strong | vdW-DF 与 dispersion correction 方法文献 |
| `input_dft`、`vdw_corr`、电场/孤立边界字段必须按 QE 官方文档核验 | Version-sensitive | QE `INPUT_PW` |
| vdW 模型改变结构会传播到 bands、phonon、work function | Strong | DFT workflow 依赖关系与本仓库 workflow 审阅规范 |
| 低维 vacuum/dipole/electrostatics 是 work function 和 potential 的独立边界 | Boundary | 表面/界面 electrostatics 与本仓库 potential/work function 页面 |

## 资料来源

- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)：`input_dft`、`vdw_corr`、`tefield/dipfield`、`assume_isolated`、ESM 等字段边界。
- Dion et al., vdW-DF foundational work：nonlocal correlation / dispersion 模型边界。
- Tkatchenko and Scheffler dispersion correction literature：density-dependent dispersion correction 边界。
- [references/canonical-literature.md](../references/canonical-literature.md)：canonical literature 分级。
- [workflows/electronic/work-function.md](../workflows/electronic/work-function.md)：work function workflow 审阅入口。
