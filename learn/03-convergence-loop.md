# Convergence Loop

## 能力目标

理解 cutoff、`ecutrho`、k-points、smearing / occupations 对 QE 数值结果的影响。

Convergence loop 的目的，是建立一组可复查的数值控制策略。最基本的四个数值控制维度是：

- `ecutwfc`：波函数平面波 cutoff。
- `ecutrho`：电荷密度和势的 cutoff。
- k mesh：SCF、NSCF、DOS、phonon 等 workflow 使用的 uniform k-point sampling。
- smearing / occupations：占据方式、展宽函数和 `degauss`。

这四个维度会相互影响，但测试时必须每次只改变一个变量。比如测试 cutoff 时固定结构、赝势、k mesh 和 smearing；测试 k mesh 时固定结构、赝势、cutoff 和 smearing；测试 smearing 时固定结构、赝势、cutoff 和 k mesh。否则 output 中的变化无法归因，也不能形成下游准入判断。

## 完成判据

- 能说明 `ecutwfc` 和 `ecutrho` 的区别。
- 能说明 k-point mesh 与 bands path 的区别。
- 能说明金属为什么需要 smearing。
- 能说明为什么每次 convergence scan 只能改变一个变量。
- 能把目标量从 total energy 扩展到 force、stress、band gap、phonon frequency 等下游相关指标。
- 能写出通用 convergence 记录表。
- 能判断一个 SCF 是否允许进入 relax、bands、DOS 或 phonon。

## 收敛目标量

Total energy 是常见的检查量，但不总是最终目标。不同下游 workflow 需要不同目标量：

| 下游目标 | 常见目标量 | 为什么不能只看 total energy |
|---|---|---|
| static SCF / energy comparison | total energy, Fermi level, charge convergence | 总能稳定只能说明当前 SCF 目标较稳定，不自动保证力、应力或响应性质稳定 |
| relax / vc-relax | force, stress, total energy trend | force 和 stress 对 cutoff、`ecutrho`、k mesh 更敏感，可能在总能变化很小时仍未稳定 |
| bands / gap | band gap, Fermi level, band ordering near target region | band gap 和费米面附近能级可能比总能更依赖 k mesh、occupation 和 smearing |
| DOS / Fermi surface | Fermi level, DOS near Fermi level, k-mesh dependence | DOS 平滑不等于收敛，dense mesh 与展宽需要单独记录 |
| phonon / DFPT | phonon frequency, acoustic mode behavior, q-point result | phonon frequency 对结构、force、stress、cutoff、`ecutrho`、k mesh 和 smearing 都更敏感 |

因此，convergence loop 的结论必须写清“对哪个目标量收敛”。如果只完成 total energy 收敛，只能说当前能量比较可用，不能自动放行 phonon、stress、force 或最终结构判断。

## 敏感性顺序

在许多 workflow 中，phonon、stress、force 比单点 total energy 更容易暴露参数不足。常见风险包括：

- cutoff 或 `ecutrho` 只按 total energy 选择，进入 relax 后 residual force 不稳定。
- k mesh 对 total energy 影响小，但对 Fermi level、DOS 或 band gap 判断影响明显。
- smearing 让 SCF 更容易收敛，但 `degauss` 过大可能改变 force、DOS 或 phonon frequency 的解释。
- phonon 计算出现小负频时，不能先把它解释成物理结论；应先复查结构、force、stress、cutoff、`ecutrho`、k mesh 和 smearing。

## 通用测试规则

每一轮 convergence scan 都应写清：

1. 固定参数：`<structure>`、`<pseudo>`、`<functional>`、`<calculation_type>`、`<prefix>`、`<outdir>`、k mesh、cutoff、smearing 等。
2. 被测试变量：只允许一个主变量变化，例如 `ecutwfc/ecutrho`、k mesh 或 `degauss`。
3. 目标量：本轮用 total energy、force、stress、band gap、phonon frequency 或其他可复查输出判断。
4. 判断阈值：使用 `<energy_tolerance>`、`<force_tolerance>`、`<stress_tolerance>`、`<gap_tolerance>`、`<frequency_tolerance>` 等占位符记录，不在通用页面写具体材料数值。
5. 下游准入：明确允许进入哪些 workflow，哪些仍然只能探索，哪些必须 BLOCK。

## 可验证输出

| tested parameter | fixed parameters | observed output | decision |
|---|---|---|---|
| `ecutwfc` | structure, pseudo, k mesh | total energy / force / stress | PASS / WARN / BLOCK |
| k mesh | structure, pseudo, cutoff | total energy / Fermi level | PASS / WARN / BLOCK |
| smearing | metallic system setup | SCF stability / energy trend | PASS / WARN / BLOCK |

## 通用 convergence table

```markdown
## Convergence table: <tested_variable>

- System: <system>
- Structure source: <structure_source>
- Pseudopotentials: <pseudo_set>
- QE version: <qe_version>
- Calculation type: <scf/relax/vc-relax/phonon/other>
- Fixed parameters: <structure>, <pseudo>, <functional>, <k_mesh>, <ecutwfc>, <ecutrho>, <occupations>, <smearing>, <degauss>
- Tested variable: <tested_variable>
- Target observable: <total_energy/force/stress/band_gap/phonon_frequency/other>
- Convergence criterion: <criterion>
- Downstream target: <relax/bands/DOS/phonon/other>

| run id | tested value | fixed values changed? | SCF status | target observable | delta from previous | warnings | decision | downstream allowed |
|---|---|---|---|---|---|---|---|---|
| <run-001> | <value-1> | no | PASS / WARN / BLOCK | <observable-1> | <delta-1> | <warnings> | PASS / WARN / BLOCK | <allowed_workflows> |
| <run-002> | <value-2> | no | PASS / WARN / BLOCK | <observable-2> | <delta-2> | <warnings> | PASS / WARN / BLOCK | <allowed_workflows> |
| <run-003> | <value-3> | no | PASS / WARN / BLOCK | <observable-3> | <delta-3> | <warnings> | PASS / WARN / BLOCK | <allowed_workflows> |
```

`fixed values changed?` 应优先为 `no`。如果某一行必须改动额外参数，例如 SCF 不稳定而改变 mixing 或 diagonalization，应把该行标为 WARN 或另开一轮 scan，不能把它直接混入同一条收敛曲线。

## 结论记录和下游准入

Convergence 结论不要只写“参数已收敛”。应按目标量和下游用途记录：

```markdown
## Convergence conclusion

- Tested variable: <tested_variable>
- Selected value: <selected_value>
- Target observable: <target_observable>
- Criterion: <criterion>
- Evidence: <run_ids_or_output_paths>
- Status: PASS / WARN / BLOCK
- Reason:
- Allowed downstream workflows:
- Not allowed / needs retest:
- Retest trigger: <new_pseudo/new_structure/new_functional/new_target_observable/new_workflow>
```

下游准入可以按以下原则判断：

| 状态 | convergence 含义 | 下游准入 |
|---|---|---|
| PASS | 目标量在记录阈值内稳定，input/output/provenance 可复查，无未解释 warning | 允许进入对应 downstream workflow |
| WARN | 参数只覆盖初步目标，或存在已记录风险 | 可用于探索；不能作为最终结论或高敏感 workflow 的依据 |
| BLOCK | SCF 未收敛、变量混改、warning 未解释、目标量未稳定或 provenance 不清 | 不允许下游；需要重新设计 scan 或修复输入 |

如果后续更换 `<structure>`、`<pseudo>`、`<functional>`、`<smearing_policy>`，或从 energy comparison 转向 force、stress、phonon frequency 等更敏感目标，应重新审查 convergence 结论。

## 推荐阅读

- QE INPUT_PW reference
- Pranab Das convergence materials
- `standards/output-review-checklist.md`

## 后续进入哪个 workflow

完成后进入 `learn/04-relaxation-loop.md` 或电子结构 workflow。
