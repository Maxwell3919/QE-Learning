# Energy vs Free Energy

## 本页解决什么问题

本页用于判断 DFT total energy、energy difference、enthalpy、phonon free energy、finite-temperature stability、MD 和 NEB 之间的边界。SCF/relax/vc-relax 给出的能量属于特定模型和结构状态下的电子能量面信息；它不能自动代表所有温度、压力和动力学条件下的 thermodynamic stability。

本页不承担热力学教程功能；它帮助用户决定当前 QE output 能支持 0 K electronic energy statement，还是需要 phonon/QHA/MD/free-energy 方法补充。

## 典型 output 现象

| 现象 | 可能提示 | 不能直接说明什么 |
|---|---|---|
| 两个结构 total energy 差异接近收敛误差 | 数值不确定性主导 | 不能给强相稳定结论 |
| vc-relax 后 pressure/stress 未稳定 | enthalpy 或 pressure boundary 不清 | 不能比较压力相关稳定性 |
| phonon DOS 存在未解释 imaginary modes | harmonic free energy 前提失败 | 不能直接做 harmonic thermodynamics |
| NEB barrier 输出正常 | MEP 完成 | 不能直接写成 free-energy barrier |
| 短 MD trajectory 看似稳定 | 采样有限 | 不能证明有限温全局稳定 |

## 可能原因

- 数值误差：SCF、cutoff、k mesh、force/stress、phonon q-grid 或 MD timestep/sampling 未收敛。
- 模型误差：functional、PP、U、SOC、vdW 或 magnetic state 选择改变 energy hierarchy。
- 赝势误差：PP 影响 total energy、force、stress、phonon 和 thermal properties。
- 边界条件误差：finite-size、cell constraint、pressure boundary、vacuum 或 dimensionality 改变能量比较。
- 后处理误差：用不可信 phonon DOS 做 free energy，或把 smoothed DOS 当作 thermal evidence。
- workflow 传播误差：energy、phonon、QHA、MD 或 NEB 来自不同结构/模型/`prefix/outdir`。
- 真实物理效应：zero-point energy、vibrational entropy、anharmonicity、metastability 或 dynamically stabilized phases 改变有限温判断。

## 必查 QE input/output

| 对象 | 程序或文件 | 检查意义 |
|---|---|---|
| total energy and estimated accuracy | `pw.x` output | 判断 0 K electronic energy 的数值噪声 |
| force/stress/pressure | relax / vc-relax output | 判断结构和 enthalpy boundary |
| `input_dft`、PP、U/SOC/vdW setting | input / record | 确认 energy comparison 属于同一模型 |
| phonon frequencies and DOS | `ph.x`、`q2r.x`、`matdyn.x` | 判断 harmonic free energy 前提 |
| imaginary modes | phonon output | 判断 harmonic/QHA 是否进入 BLOCK |
| pressure and cell state | vc-relax / record | 区分 total energy 与 enthalpy |
| MD settings and trajectory | `pw.x` MD / CP | 判断 finite-temperature sampling 边界 |
| NEB images and convergence | `neb.x` output | 判断 minimum energy path，不是 free-energy path |

## 判断规则

1. 0 K DFT total energy difference 只能支持选定模型下的 static-lattice energy statement。
2. 需要压力边界时，应区分 total energy、enthalpy 和 stress/pressure convergence。
3. Harmonic vibrational free energy 需要可信 phonon dispersion/DOS；未解释 imaginary modes 会破坏 harmonic 前提。
4. QHA、anharmonic methods、AIMD 或 thermodynamic integration 属于更高层级，不能由单点 total energy 替代。
5. NEB 默认提供 potential-energy surface 上的 minimum energy path；free-energy path 需要额外采样或模型说明。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | 明确结论层级为 0 K electronic energy / static enthalpy / harmonic free energy / finite-temperature method，并完成对应 convergence 与 output review | 可报告对应层级稳定性或路径结论 |
| WARN | 用 0 K energy 或 harmonic phonon 做有限温趋势，边界已说明但证据不完整 | 只允许写趋势或假设下结论 |
| BLOCK | 用单点 total energy 直接写 finite-temperature thermodynamics；未解释 imaginary modes 却做 harmonic free energy；NEB barrier 写成 free-energy barrier | 停止热力学或动力学强结论 |

## 不能做出的过度结论

- 不能把 DFT total energy 直接等同有限温自由能。
- 不能忽略 vibrational contribution 后写高温相稳定结论。
- 不能用不可信 phonon DOS 做热力学。
- 不能把短 MD 稳定性写成充分采样。
- 不能把 NEB minimum energy barrier 写成 free-energy barrier，除非额外方法支持。

## 下游影响

- `workflows/ground-state/scf.md`、`workflows/ground-state/relax.md`、`workflows/ground-state/vc-relax.md`：决定 energy/force/stress 证据层级。
- `workflows/phonon/phonon-dos.md` 和 `workflows/phonon/phonon-debugging.md`：phonon stability 是 harmonic thermodynamics 的前置。
- `workflows/advanced/molecular-dynamics.md`：finite-temperature sampling 需要独立边界。
- `workflows/advanced/neb-reaction-path.md`：NEB 结论应限定为 minimum energy path。
- `physics-judgement/uncertainty-statement-template.md`：energy/free-energy statement 必须写出层级和限制。

## 与 theory-minimum / workflows / standards 的关系

- 最低理论回查：[theory-minimum/force-stress-relaxation.md](../theory-minimum/force-stress-relaxation.md)、[theory-minimum/dfpt-phonons.md](../theory-minimum/dfpt-phonons.md)。
- workflow 回查：[workflows/phonon/phonon-dos.md](../workflows/phonon/phonon-dos.md)、[workflows/advanced/molecular-dynamics.md](../workflows/advanced/molecular-dynamics.md)、[workflows/advanced/neb-reaction-path.md](../workflows/advanced/neb-reaction-path.md)。
- 记录规范：[standards/output-review-checklist.md](../standards/output-review-checklist.md)、[standards/calculation-record-template.md](../standards/calculation-record-template.md)。

## 来源与结论强度

| 结论 | 强度 | 来源边界 |
|---|---|---|
| DFT total energy 是选定 ground-state 模型下的能量面信息 | Strong | HK/KS DFT 与 QE `pw.x` output |
| harmonic free energy 依赖可信 phonon frequencies / DOS | Strong | DFPT phonon theory 与 PHonon workflow |
| NEB 给出 minimum energy path，不自动给出 free-energy path | Boundary | NEB 方法边界 |
| MD/thermostat/free-energy 方法输入和限制需查当前 QE 文档 | Version-sensitive | QE package docs |

## 资料来源

- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)：SCF、relax、vc-relax、MD 相关输入边界。
- [QE INPUT_PH](https://www.quantum-espresso.org/Doc/INPUT_PH.html)：phonon response 与 frequency 数据来源。
- Baroni et al., DFPT phonon review：harmonic phonon / response framework。
- [workflows/phonon/phonon-dos.md](../workflows/phonon/phonon-dos.md)：本仓库 phonon DOS workflow。
- [references/canonical-literature.md](../references/canonical-literature.md)：canonical literature 分级。
