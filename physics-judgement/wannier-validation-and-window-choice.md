# Wannier Validation and Window Choice

## 本页解决什么问题

本页用于判断 Wannierization 是否足以支撑有效模型、插值、Berry quantity、transport、topology 或 EPC 下游结论。Wannier functions 是对子空间和 gauge 的选择，不是“程序收敛就可信”的自动结果。

核心审阅问题是：所选 projection、frozen window、outer window、disentanglement 和 spin/SOC 数据链，是否能在目标能区复现 QE ab initio bands，并满足下游 observable 的精度要求。

## 典型 output 现象

| 现象 | 可能提示 | 不能直接说明什么 |
|---|---|---|
| Wannier90 正常结束但 interpolated bands 与 QE bands 不匹配 | projection、window 或 disentanglement 失败 | 不能进入 transport、EPC 或 topology 结论 |
| total spread 较小但目标 bands 错位 | 局域性指标与目标子空间不一致 | 不能只凭 spread 判定模型可信 |
| frozen window 没覆盖目标能区 | 关键 bands 未被约束 | 不能解释该能区的有效哈密顿量 |
| disentanglement 对 window 敏感 | entangled bands 子空间选择不稳 | 不能给定量插值结论 |
| SOC/spinor 数据链混乱 | spin/SOC 文件或投影设置不一致 | 不能比较 SOC 与非 SOC Wannier 结果 |

## 可能原因

- 数值误差：NSCF k mesh、`nbnd`、能量窗口或收敛精度不足。
- 模型误差：functional、DFT+U、SOC、vdW 或 PP 改变目标 bands，但 Wannier 设置未更新。
- 赝势误差：projector 与 PP 轨道通道、semicore 或 relativistic treatment 不匹配。
- 边界条件误差：低维、磁性、SOC 或小带隙体系的 symmetry / degeneracy 没有重审。
- 后处理误差：只检查 `.wout` 收敛和 spread，没有回对 bands。
- workflow 传播误差：Wannier 输入来自旧 SCF/NSCF、不同 `prefix/outdir`、不同 `input_dft` 或不同 spin/SOC 设置。
- 真实物理效应：目标 bands 本身高度纠缠、强杂化或接近简并，需要更谨慎的子空间定义。

## 必查 QE input/output

| 对象 | 程序或文件 | 检查意义 |
|---|---|---|
| upstream SCF / NSCF | `pw.x` input/output | 确认结构、functional、PP、SOC、U、`nbnd` 和 k mesh |
| target band manifold | QE bands / NSCF | 明确 Wannierization 要复现哪个能区和哪些 bands |
| projections | Wannier90 input | 判断初始局域轨道是否有物理含义 |
| frozen window | Wannier90 input/output | 确认必须复现的能区被固定 |
| outer window / disentanglement | Wannier90 input/output | 判断 entangled bands 的子空间选择 |
| spreads and convergence | `.wout` | 只作为局域性和优化证据之一 |
| interpolated bands vs ab initio bands | plotting / compare output | 最关键 validation 证据 |
| `pw2wannier90.x` / interface files | QE-Wannier90 interface | 检查文件链、spinor、SOC、orbital projection 是否一致 |

## 判断规则

1. Wannier validation 的首要证据是目标能区内 interpolated bands 回对 QE bands，而不是 spread 数值本身。
2. isolated bands 和 entangled bands 的风险不同。entangled 情况必须记录 frozen window、outer window 和 disentanglement 判断。
3. 下游用途决定验证强度。只做可视化、有效模型、Berry curvature、transport、EPC 或 topology 时，所需 band/subspace 验证层级不同。
4. SOC、noncollinear spin、magnetism、DFT+U 或 functional 切换后，不能复用旧 Wannier 文件解释新模型。
5. Window 和 projection 是研究选择，必须在 calculation record 中记录，不能只保存最终 `.wout`。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | window、projection、spread、disentanglement、target subspace 和 interpolated-vs-QE bands 全部记录并通过目标能区验证 | 可进入相应插值、有效模型或下游 observable |
| WARN | 只在部分能区匹配，或 window/projection sensitivity 未完全审阅 | 可用于探索或可视化，不支撑定量下游 |
| BLOCK | 未验证 ab initio vs interpolated bands；window 来源不清；projection 与物理目标错位；直接进入 EPC/transport/topology 结论 | 停止下游解释，回到子空间选择 |

## 不能做出的过度结论

- 不能只凭 Wannier90 收敛或较小 spread 判断模型可信。
- 不能用未经验证的 Wannier interpolation 做定量 transport、EPC 或 topology 结论。
- 不能跨 SOC / non-SOC、不同 functional、不同 PP 或不同 `prefix/outdir` 混用 Wannier 文件。
- 不能把 projection 默认值当作物理证明。
- 不能解释 frozen window 外的 bands，除非重新说明外推边界。

## 下游影响

- `workflows/advanced/wannier90-overview.md`：决定 Wannierization 是否只是边界页入口，还是具备下游准入。
- `workflows/advanced/electron-phonon-overview.md`：EPW / EPC 依赖经过验证的 Wannier interpolation。
- `workflows/electronic/bands.md`：ab initio bands 是 Wannier validation 的对照基准。
- `physics-judgement/epc-data-chain-and-convergence.md`：EPC 对 Wannier validation 要求更高。

## 与 theory-minimum / workflows / standards 的关系

- 最低理论回查：[theory-minimum/dft-ks-scf.md](../theory-minimum/dft-ks-scf.md)、[theory-minimum/kpoints-symmetry-kpath.md](../theory-minimum/kpoints-symmetry-kpath.md)、[theory-minimum/pseudopotentials.md](../theory-minimum/pseudopotentials.md)。
- workflow 回查：[workflows/advanced/wannier90-overview.md](../workflows/advanced/wannier90-overview.md)、[workflows/electronic/bands.md](../workflows/electronic/bands.md)、[workflows/ground-state/nscf.md](../workflows/ground-state/nscf.md)。
- 记录规范：[standards/output-review-checklist.md](../standards/output-review-checklist.md)、[standards/calculation-record-template.md](../standards/calculation-record-template.md)。

## 来源与结论强度

| 结论 | 强度 | 来源边界 |
|---|---|---|
| Wannier functions 是 Bloch 子空间的局域表示，依赖 gauge 和子空间选择 | Strong | MLWF 方法论文与 Wannier90 文档 |
| entangled bands 需要 disentanglement window 和 frozen window 审阅 | Strong | Souza-Marzari-Vanderbilt entangled-band 方法 |
| spread 不能替代 band interpolation validation | Boundary | Wannier workflow 审阅规则 |
| interface 字段与 spinor/SOC 支持需查当前工具文档 | Version-sensitive | Wannier90 与 QE interface 文档 |

## 资料来源

- [Wannier90 documentation](https://wannier90.readthedocs.io/en/latest/)：Wannier90 输入、输出和 workflow 文档。
- Marzari and Vanderbilt, maximally localized Wannier functions：MLWF 基础。
- Souza, Marzari and Vanderbilt, disentanglement for entangled bands：window 与子空间选择。
- Marzari et al., Wannier functions review：Wannier interpolation 与应用边界。
- [workflows/advanced/wannier90-overview.md](../workflows/advanced/wannier90-overview.md)：本仓库 Wannier 边界页。
