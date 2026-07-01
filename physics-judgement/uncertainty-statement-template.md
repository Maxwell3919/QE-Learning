# Uncertainty Statement Template

## 本页解决什么问题

本页给出 DFT/QE 结果陈述的 uncertainty statement 模板。目标是把“算出来了”改写成可复查的科研表述：说明计算层级、输入模型、收敛证据、output 证据、限制和下游准入。

不确定性说明不是空泛免责声明。它应告诉读者哪些误差已经通过收敛测试降低，哪些来自模型选择，哪些来自边界条件或后处理，哪些仍然限制结论强度。

## 典型 output 现象

| 现象 | 可能提示 | 不能直接说明什么 |
|---|---|---|
| 只报告一个最终数值 | 缺少 sensitivity 和 provenance | 不能判断结论强度 |
| convergence table 只覆盖 total energy | force、stress、phonon、DOS 或 work function 仍可能不稳 | 不能把所有 observable 都判为 PASS |
| PP、functional、U、SOC、vdW 来源不清 | model uncertainty 不可追踪 | 不能做跨模型定量比较 |
| 下游结果读取旧 `prefix/outdir` | workflow propagation error | 不能写正式结论 |
| output 有 warning 但陈述未提及 | 证据与结论不匹配 | 不能通过审阅 |

## 可能原因

- 数值误差：cutoff、k mesh、smearing、q-grid、SCF threshold、force/stress threshold 或 broadening 未审阅。
- 模型误差：functional、DFT+U、hybrid、SOC、vdW、magnetic state 或 excited-state method 选择影响结论。
- 赝势误差：PP provenance、functional consistency、relativistic treatment 或 transferability 未记录。
- 边界条件误差：finite-size、vacuum、dipole、slab、charged state、dimensionality 或 pressure/temperature boundary 不清。
- 后处理误差：bands/DOS/PDOS、potential、Wannier、EPC、phonon DOS 或 spectra 的解释越界。
- workflow 传播误差：relax、final SCF、NSCF、phonon、post-processing 来自不同模型或旧 scratch。
- 真实物理效应：soft mode、magnetism、Fermi surface、polar response、anharmonicity 或 excitonic effects 需要在结论中单独标注。

## 必查 QE input/output

| 类别 | 必查记录 | 用途 |
|---|---|---|
| 计算层级 | SCF、relax、NSCF、bands、phonon、Wannier、EPW、GW/BSE 等 | 防止把不同层级结论混写 |
| 输入模型 | `input_dft`、PP、U、SOC、vdW、occupation、smearing | 说明模型边界 |
| 数值证据 | cutoff、k mesh、q-grid、threshold、broadening convergence | 说明 numerical uncertainty |
| output 证据 | warning、SCF accuracy、force/stress、Fermi energy、frequency、tensor、potential plateau | 说明结论来源 |
| 文件链 | `prefix/outdir`、input/output 文件名、post-processing files | 防止下游读错数据 |
| 来源 | 官方文档、canonical literature、内部 workflow/standards | 支撑边界判断 |

## 判断规则

1. 每个正式结论至少要说明计算层级、模型、收敛证据、output 证据和限制。
2. 数值收敛 statement 只能覆盖已测试 observable，不能自动扩展到 phonon、work function、EPC 或 spectra。
3. 模型选择 statement 应写明 functional、PP、U、SOC、vdW、excited-state method 等边界。
4. 若结论依赖可视化或后处理，必须说明上游数据链和后处理解释边界。
5. 未完成审阅的结果可以写成 internal note 或 exploratory trend，不能写成无边界定量结论。

## Claim strength 写法

| 证据状态 | 推荐写法 | 避免写法 |
|---|---|---|
| PASS，目标 observable 已收敛且模型边界清楚 | “在 `<model>` 和 `<convergence evidence>` 下，结果支持 `<bounded claim>`。” | “DFT 证明……” |
| PASS，但模型本身有已知边界 | “该结论是 `<DFT-level / harmonic / KS-level>` 层级判断。” | “与实验完全一致。” |
| WARN，缺少部分敏感性或对照 | “当前结果提示 `<trend>`，仍需 `<missing check>` 后才能定量化。” | “该性质已经确定。” |
| BLOCK，关键文件链或 output 证据缺失 | “当前输出不能支撑 `<claim>`，原因是 `<missing evidence>`。” | “结果仅供参考但仍给最终数值。” |

好的 uncertainty statement 应减少读者猜测：它说明哪些证据已经通过，哪些证据仍缺失，以及下游可以做什么。它不是在结论末尾添加一句泛泛的“存在误差”，而是把误差来源和 claim strength 直接绑定。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | statement 包含来源、模型、convergence evidence、output evidence、限制和 downstream permission | 可进入报告、论文草稿或长期记录 |
| WARN | statement 可用于内部讨论，但缺少部分 sensitivity 或模型对照 | 只允许写条件结论或趋势 |
| BLOCK | statement 写成无边界绝对结论；关键来源、参数、output review 或文件链缺失 | 不得作为正式科研结论 |

## 不能做出的过度结论

- 避免写“DFT 证明……”，除非后面明确计算层级和模型边界。
- 避免写“该体系一定……”，应说明模型、温度、压力、结构和方法范围。
- 避免写“计算结果与实验完全一致……”，应说明比较对象和不确定性。
- 避免写“该参数已经足够……”，除非有目标 observable 的 convergence evidence。
- 避免用 `JOB DONE`、平滑图像、small spread、单个 `lambda` 或单点 total energy 替代完整证据。

## 下游影响

Uncertainty statement 应出现在以下记录中：

- energy difference / formation-energy-like statement；
- band structure、band gap、DOS、PDOS statement；
- phonon stability、phonon DOS、Born charge、IR/Raman statement；
- work function、electrostatic potential statement；
- DFT+U、SOC、vdW、hybrid、low-dimensional statement；
- Wannier、EPC、excited-state、free-energy statement。

## 与 theory-minimum / workflows / standards 的关系

- 最低理论回查：[theory-minimum/README.md](../theory-minimum/README.md)、[theory-minimum/dft-ks-scf.md](../theory-minimum/dft-ks-scf.md)、[theory-minimum/dfpt-phonons.md](../theory-minimum/dfpt-phonons.md)。
- workflow 回查：[workflows/README.md](../workflows/README.md)、[workflows/ground-state/scf.md](../workflows/ground-state/scf.md)、[workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)。
- 记录规范：[standards/output-review-checklist.md](../standards/output-review-checklist.md)、[standards/calculation-record-template.md](../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../standards/pass-warn-block.md)。

## 可复用 statement 模板

### Energy Difference

```markdown
本结论是 `<method_level>` 层级下的 energy difference 判断。输入模型为 `<functional>` / `<pseudopotential_set>` / `<spin_or_soc_or_u_or_vdw_setting>`，结构状态为 `<structure_state>`。数值收敛已针对 `<target_observable>` 检查，包括 `<cutoff_evidence>`、`<kmesh_evidence>` 和 `<force_stress_evidence>`。output 证据包括 `<total_energy_lines>`、`<scf_accuracy>`、`<warnings>`。该结论不包含 `<finite_temperature_or_free_energy_effects>`；允许进入 `<allowed_downstream>`。
```

### Band Structure / Gap

```markdown
本结论是 `<functional_level>` 的 Kohn-Sham electronic structure 判断。bands 使用 `<k_path_source>`，DOS 使用 `<uniform_mesh>`，能量参考为 `<fermi_or_vbm_reference>`。output 已审阅 `<nbnd>`、`<occupations>`、`<fermi_energy>`、`<warnings>`。band gap 仅作为 `<DFT_level_or_trend>` 表述，不写成 quasiparticle 或 optical gap；允许进入 `<allowed_downstream>`。
```

### DOS / PDOS

```markdown
DOS/PDOS 结论基于 `<SCF_to_NSCF_to_postproc_chain>`。已记录 `<kmesh>`、`<smearing_or_tetrahedron>`、`<degauss_or_broadening>`、`<energy_reference>` 和 `<projection_basis>`。PDOS 被解释为 projector-dependent analysis，不作为排他性化学分解；允许进入 `<figure_or_trend_or_quantitative_statement>`。
```

### Phonon Stability

```markdown
phonon 结论基于 `<final_static_scf>`、`<ph_q_grid>`、`<q2r_matdyn_chain>`。已审阅 `<ph_convergence>`、`<acoustic_modes>`、`<ASR_setting>`、`<imaginary_frequency_triage>` 和 `<warnings>`。若存在 `<imaginary_modes>`，其来源被归类为 `<numerical_or_structural_or_physical>`；允许进入 `<stability_statement_or_followup>`。
```

### Work Function

```markdown
work function 结论基于 `<slab_model>` 的 `<SCF_to_pp_to_average_chain>`。已记录 `<surface_direction>`、`<vacuum_region>`、`<dipole_or_boundary_setting>`、`<fermi_energy>`、`<potential_plateau_evidence>` 和 `<warnings>`。该数值只在 `<boundary_condition>` 下成立；允许进入 `<trend_or_quantitative_report>`。
```

### DFT+U / SOC

```markdown
本结论使用 `<DFT_plus_U_or_SOC_setting>` 模型。U/SOC provenance 为 `<u_source_or_relativistic_pp_source>`，并记录 `<projector_or_manifold>`、`<functional>`、`<pseudopotential>`、`<spin_state>` 和 `<symmetry_review>`。输出已审阅 `<magnetization>`、`<Hubbard_or_SOC_messages>`、`<warnings>`。结论不跨 `<different_model_chain>` 比较。
```

### vdW / Low-Dimensional

```markdown
本结论使用 `<vdw_or_nonlocal_model>` 与 `<low_dimensional_boundary>`。已记录 `<functional>`、`<pseudopotential>`、`<cell_or_vacuum>`、`<dipole_or_isolated_setting>`、`<force_stress_review>` 和 `<downstream_recalculation>`。结论只在该 model/boundary 下成立，不作为所有 vdW 模型的普适排序。
```

### EPC

```markdown
EPC 结论基于 `<SCF_to_DFPT_to_Wannier_to_EPW_chain>`。已审阅 `<phonon_convergence>`、`<Wannier_band_validation>`、`<coarse_dense_kq_grids>`、`<smearing_broadening>`、`<lambda_alpha2F_linewidth_or_mobility_output>` 和 `<warnings>`。该结果用于 `<screening_or_quantitative_observable>`，并保留 `<remaining_uncertainty>`。
```

### Free Energy

```markdown
热力学结论层级为 `<static_energy_or_harmonic_free_energy_or_QHA_or_MD>`。已记录 `<temperature_pressure_boundary>`、`<phonon_or_sampling_evidence>`、`<imaginary_mode_status>`、`<model_assumptions>` 和 `<convergence_evidence>`。本结论不把单点电子能量直接写成有限温自由能；允许进入 `<allowed_thermodynamic_statement>`。
```

## 来源与结论强度

| 结论 | 强度 | 来源边界 |
|---|---|---|
| numerical uncertainty 与 model uncertainty 必须分开记录 | Strong | reproducibility literature 与本仓库 standards |
| statement 只能覆盖已审阅的 workflow 和 observable | Strong | output review 规范 |
| 具体 QE 字段和工具行为需查当前官方文档 | Version-sensitive | QE / PHonon / Wannier90 / EPW / Yambo docs |
| 缺少 evidence 的结果只能作为探索或内部记录 | Boundary | PASS/WARN/BLOCK 规范 |

## 资料来源

- Lejaeghere et al., reproducibility in DFT calculations of solids：可复现性与方法比较边界。
- [standards/output-review-checklist.md](../standards/output-review-checklist.md)：本仓库 output review 入口。
- [standards/calculation-record-template.md](../standards/calculation-record-template.md)：calculation record 模板。
- [references/canonical-literature.md](../references/canonical-literature.md)：canonical literature 分级。
- [references/source-index.md](../references/source-index.md)：公开来源索引。
