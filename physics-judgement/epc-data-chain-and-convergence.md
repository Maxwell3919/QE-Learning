# EPC Data Chain and Convergence

## 本页解决什么问题

本页用于判断 electron-phonon coupling (EPC) 结果是否具备可复查的数据链。EPC 不是只看最终 `lambda`、linewidth、mobility 或 `Tc` 数字；它把 ground-state SCF、phonon/DFPT、电子态、Wannier interpolation、dense k/q sampling、smearing/broadening 和目标 observable 连接在一起。

EPC 的风险在于误差会沿整条链传播。任一上游环节不可信，最终 EPC 数值都可能只是程序输出，而不是可写入科研结论的物理结果。积分后的 `lambda` 或 `Tc` 还会隐藏 q 点、声子 branch、Fermi-surface patch 和 broadening 的局部问题；因此 EPC 审阅应同时看汇总指标和 mode-/q-resolved 证据。

## 典型 output 现象

| 现象 | 可能提示 | 不能直接说明什么 |
|---|---|---|
| EPC 指标对 k/q mesh、smearing 或 broadening 强烈变化 | Fermi surface、double-delta integration 或 interpolation 未收敛 | 不能给量化 EPC 结论 |
| phonon linewidth 异常或 alpha2F 出现不稳定峰 | phonon、occupation、Wannier 或 q-grid 问题 | 不能只解释为真实强耦合 |
| phonon dispersion 存在未解释 imaginary modes | dynamical stability 链条未通过 | 不能进入稳定相 EPC 解释 |
| Wannier bands 未回对 QE bands | interpolation 基础不可信 | 不能使用 EPW 下游量 |
| EPW 正常结束但 provenance 不完整 | 程序完成不等于数据链可信 | 不能只引用最终表格 |

## 可能原因

- 数值误差：SCF/NSCF、k mesh、q-grid、smearing、broadening、phonon threshold 或 interpolation mesh 未收敛。
- 模型误差：functional、DFT+U、SOC、vdW 或 pseudopotential 改变电子态和 phonon，却未重新审阅。
- 赝势误差：PP 影响 band structure、phonon、Born charge 和 electron-phonon matrix elements。
- 边界条件误差：金属 Fermi surface、polar correction、low-dimensional electrostatics 或 finite-size 设置未处理。
- 后处理误差：只看 `lambda`、`Tc` 或 plot，没有检查 linewidth、alpha2F、Wannier validation 和 q/k convergence。
- workflow 传播误差：SCF、DFPT、Wannier、EPW 读取不同结构、不同 `prefix/outdir` 或不同模型。
- 真实物理效应：Fermi surface nesting、Kohn anomaly、polar coupling 或 soft phonon 真实影响 EPC，但需要与数值因素区分。

## 必查 QE input/output

| 对象 | 程序或文件 | 检查意义 |
|---|---|---|
| relaxed structure / final SCF | `pw.x` | EPC 上游电子基态和 occupation 的质量 |
| NSCF / electronic bands | `pw.x` | band manifold、Fermi surface、`nbnd` 和 k sampling |
| `ph.x` q-grid | PHonon | phonon 和 perturbation convergence |
| `fildvscf` | `ph.x` | 保存 response potential，供后续 EPC 使用 |
| `electron_phonon` 相关设置 | `ph.x` | QE 内 EPC 模式入口，版本语义需核验 |
| Wannier projection / window / validation | Wannier90 / EPW | dense interpolation 的前提 |
| EPW coarse/dense k/q grids | EPW input/output | 决定插值和最终 observable 的采样质量 |
| `lambda`、alpha2F、linewidth、mobility、`Tc` | EPW / post-processing | 目标量本身，不是上游审阅替代品 |
| mode-/q-resolved linewidth 或 coupling | EPW / post-processing | 判断强耦合来自局部模式、Kohn anomaly、polar coupling 还是数值噪声 |
| warnings、units、broadening | output | 判断结果是否可比较 |

## 判断规则

1. EPC 结论必须先通过 SCF、phonon、Wannier 三个上游 gate，再审阅 dense k/q 和 broadening。
2. EPC 的 convergence target 应由目标 observable 决定。不同目标量对 mesh、smearing 和 interpolation 的敏感性不同。
3. 金属体系中 Fermi surface sampling 是核心风险；small-gap、polar 或 SOC 相关体系还需要额外边界说明。
4. `lambda`、alpha2F、linewidth、mobility 和 `Tc` 是不同层级的物理量，不能用一个数替代全链审阅。`Tc` 还依赖使用的理论公式、Coulomb 参数或 Eliashberg 设置，不能只由 `lambda` 自动推出。
5. 若 phonon 存在未解释 imaginary modes，稳定相 EPC 的定量解释应进入 BLOCK。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | SCF、phonon、Wannier validation、EPC k/q grids、smearing/broadening、target observable 和 provenance 均完成目标相关审阅 | 可报告对应 EPC 目标量，并写明模型和 convergence 边界 |
| WARN | 链条可追踪，但关键 grids、broadening 或 Wannier sensitivity 未完全审阅 | 只允许写趋势或 screening-level 判断 |
| BLOCK | phonon 不可信；Wannier 未验证；grid/broadening 未收敛；只给最终 EPC 指标而无数据链；EPC 下游结论跨模型混用 | 停止量化 EPC、transport 或 superconductivity 结论 |

## 不能做出的过度结论

- 不能只看一个最终 EPC 指标给出量化结论。
- 不能把 coarse-grid EPC 当作最终 mobility、resistivity 或 superconductivity 结果。
- 不能只报告积分后的 `lambda`，却不审阅 q/mode-resolved linewidth、alpha2F 特征和 broadening 敏感性。
- 不能忽略 phonon imaginary modes 对 EPC 解释的影响。
- 不能用未经验证的 Wannier interpolation 支撑 dense-grid EPC。
- 不能跨 functional、SOC、DFT+U、PP 或 structure 状态混用 EPC 数据。

## 下游影响

- `workflows/advanced/electron-phonon-overview.md`：决定 EPC 是否只停留在高级边界，还是具备完整数据链。
- `workflows/advanced/wannier90-overview.md`：Wannier validation 是 EPW 前置 gate。
- `workflows/phonon/phonon-dispersion-dfpt.md` 与 `workflows/phonon/phonon-debugging.md`：phonon 可信度直接限制 EPC。
- `workflows/electronic/fermi-surface.md`：Fermi surface sampling 影响 metallic EPC。
- `physics-judgement/uncertainty-statement-template.md`：EPC statement 必须写入全链边界。

## 与 theory-minimum / workflows / standards 的关系

- 最低理论回查：[theory-minimum/dfpt-phonons.md](../theory-minimum/dfpt-phonons.md)、[theory-minimum/smearing-metals.md](../theory-minimum/smearing-metals.md)、[theory-minimum/kpoints-symmetry-kpath.md](../theory-minimum/kpoints-symmetry-kpath.md)。
- workflow 回查：[workflows/advanced/electron-phonon-overview.md](../workflows/advanced/electron-phonon-overview.md)、[workflows/advanced/wannier90-overview.md](../workflows/advanced/wannier90-overview.md)、[workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)。
- 记录规范：[standards/output-review-checklist.md](../standards/output-review-checklist.md)、[standards/calculation-record-template.md](../standards/calculation-record-template.md)。

## 来源与结论强度

| 结论 | 强度 | 来源边界 |
|---|---|---|
| EPC matrix elements 依赖电子态、phonon 模式和 occupation/Fermi surface | Strong | EPC review 与 EPW 文档 |
| `fildvscf` 与 `electron_phonon` 等字段必须按 QE/EPW 版本核验 | Version-sensitive | QE `INPUT_PH` 与 EPW docs |
| Wannier validation 是 dense interpolation 前置条件 | Strong | Wannier90/EPW workflow |
| 单个 EPC 数值不能替代 k/q/smearing/broadening convergence | Boundary | 本仓库 output review 规范 |

## 资料来源

- [QE INPUT_PH](https://www.quantum-espresso.org/Doc/INPUT_PH.html)：`fildvscf`、`electron_phonon`、q-grid 和 PHonon 输入字段。
- [EPW documentation](https://docs.epw-code.org/)：EPW workflow、interpolation 和 output 边界。
- Giustino, electron-phonon interactions from first principles：EPC 物理与方法综述。
- [Wannier90 documentation](https://wannier90.readthedocs.io/en/latest/)：Wannier interpolation 前置文档。
- [references/canonical-literature.md](../references/canonical-literature.md)：canonical literature 分级。
