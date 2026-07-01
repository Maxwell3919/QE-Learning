# Cutoff convergence workflow

## 本页解决什么问题

本页说明如何用一组 `pw.x scf` 扫描建立 `ecutwfc/ecutrho` 策略。收敛测试的对象不是 cutoff 数字本身，而是目标 observable 对 cutoff 的敏感性，例如 total energy、force、stress、band gap、DOS 特征或后续 phonon frequency。

cutoff convergence 位于 SCF 基线之后、relax/vc-relax 和性质计算之前。没有 cutoff 策略的 SCF 即使电子自洽，也只能说明该单点在某组参数下结束，不能证明下游性质可用。

## 页面定位

- 对应学习路线：[learn/03-convergence-loop.md](../../learn/03-convergence-loop.md)
- 上游依赖：checked structure, pseudo, baseline SCF input
- 下游用途：cutoff policy for SCF, relax, bands, DOS, phonon
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

评估 `ecutwfc` 与 `ecutrho` 对目标量的影响，建立后续 workflow 可复用的 cutoff 策略。

## 上游依赖

- `<structure>`、`<pseudo>` 和 `<system>` 已记录。
- SCF 基础参数与本 workflow 目标一致。
- 已明确输出是否允许进入下游 workflow。
- 结构、赝势、k mesh、occupation/smearing、spin/SOC 和泛函设置在 scan 中保持固定。
- 每个 scan 点使用可追踪的 `prefix/outdir`，避免不同 cutoff 的 scratch 相互覆盖；可以复用 restart 作为数值加速，但必须记录来源并确认同一参数分支。

## 计算图

```text
<structure> + <pseudo> + fixed k mesh
  -> pw.x scf with ecutwfc/ecutrho scan
  -> convergence table
  -> cutoff policy for downstream workflow
```

## 需要的 QE 程序

`pw.x`

## 命令与文件边界

每个 cutoff 点应有独立 input/output 和可识别 scratch 记录：

```bash
pw.x -in pw.scf.<system>.ecut<cutoff_label>.in > pw.scf.<system>.ecut<cutoff_label>.out
```

如果多个 cutoff 点共用同一个 `outdir`，至少要用不同 `prefix` 或清理旧 scratch；否则 output 中的 convergence history、restart 和 wavefunction 来源可能混在一起。cutoff scan 表应保存每个点的 input label、实际 output cutoff、SCF 状态、目标 observable 和 WARN/BLOCK 原因。

## 通用输入模板

```fortran
&CONTROL
  calculation = 'scf',
  prefix = '<system>',
  outdir = '<scratch_dir>',
  pseudo_dir = '<pseudo_dir>',
/
&SYSTEM
  ibrav = 0,
  nat = <number_of_atoms>,
  ntyp = <number_of_species>,
  ecutwfc = <wavefunction_cutoff>,
  ecutrho = <charge_density_cutoff>,
  occupations = '<occupation_scheme>',
/
&ELECTRONS
  conv_thr = <scf_threshold>,
/
ATOMIC_SPECIES
  <Element> <Mass> <Pseudo.UPF>
ATOMIC_POSITIONS <coordinate_type>
  <Element> <x> <y> <z>
K_POINTS automatic
  <nk1> <nk2> <nk3> <sk1> <sk2> <sk3>
CELL_PARAMETERS <unit>
  <a1x> <a1y> <a1z>
  <a2x> <a2y> <a2z>
  <a3x> <a3y> <a3z>
```

## 关键 QE 输入对象

| 字段 | 所属程序 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|---|
| `ecutwfc` | pw.x | 波函数平面波 cutoff | 只看单点结果，不做 scan | output 显示 kinetic-energy cutoff |
| `ecutrho` | pw.x | 电荷密度和势 cutoff | USPP/PAW 设置过低 | output 显示 charge density cutoff |
| `ATOMIC_SPECIES` | pw.x | 赝势类型影响 cutoff 需求 | 换赝势后沿用旧 cutoff | output 中 pseudopotential type |
| `K_POINTS` | pw.x | cutoff scan 中应固定 k mesh | 同时改 cutoff 和 k mesh | output 中 k 点数量一致 |

## Output review

```markdown
## output review

- QE 程序:
- 计算类型:
- QE version:
- Input dependency:
- Structure summary:
- Pseudopotentials loaded:
- Cutoff reported:
- K-points reported:
- Convergence status:
- 本 workflow 关键输出:
- Warnings:
- Scratch / restart status:
- PASS / WARN / BLOCK:
- Reason:
- Allowed downstream workflows:
```

### 必查 output 证据

| 检查项 | 从哪里看 | 能证明什么 | 不能证明什么 | WARN/BLOCK 触发 |
|---|---|---|---|---|
| 实际 cutoff | output 中 kinetic-energy cutoff 和 charge density cutoff | QE 实际使用的 `ecutwfc/ecutrho` | 不证明已收敛 | 与 input/scan 表不一致为 `BLOCK` |
| 赝势类型 | pseudo loading 段、UPF 信息 | NC/USPP/PAW 等类型可追踪 | 不证明推荐 cutoff 足够 | 换 PP 后未重做 scan 为 `BLOCK` |
| SCF 状态 | convergence message、estimated accuracy | 每个 scan 点电子收敛 | 不证明 cutoff 收敛 | 任一用于表格的点未收敛为 `BLOCK` |
| 目标 observable | total energy、forces、stress、band gap 或后续性质记录 | cutoff 对目标量的影响 | 不证明其他 observable 同样收敛 | 只记录 total energy 但下游关心 force/stress/phonon 为 `WARN` |
| 文件链 | prefix/outdir、input/output label | scan 点可复查 | 不证明数据未混用 | restart 或 scratch 来源不清为 `WARN/BLOCK` |

## 输出判断标准

- 每个 scan 点都应确认 SCF 已收敛。
- 记录 total energy、force、stress、band gap 或 phonon frequency 等目标量。
- PASS 需要目标量变化低于自定准则；WARN 表示可探索但不足以下游定稿；BLOCK 表示需继续加大 cutoff 或更换策略。

## 收敛与可靠性

- 固定结构、赝势、k mesh、smearing，只改变 cutoff 变量。
- NC、USPP、PAW 对 `ecutrho` 敏感性不同，需按赝势类型记录。
- phonon、stress、force 任务通常比单纯总能更敏感。
- `ecutwfc` 和 `ecutrho` 可以联动扫描，也可以先确定 `ecutwfc` 再扫描 `ecutrho`；无论采用哪种策略，都必须说明为何该策略能支撑目标 observable。
- 参数增大带来基组误差降低，但不修复错误赝势、错误结构、错误泛函或错误 occupation 这类模型问题。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| `PASS` | 所有 scan 点 SCF 完成；实际 cutoff 与记录一致；目标 observable 随 cutoff 变化满足自定准则；赝势类型和下游用途已记录 | 允许把 cutoff policy 用于声明的 SCF、relax、NSCF 或后续性质 workflow |
| `WARN` | total energy 已稳定但 force/stress/phonon 等目标量未检查；scan 范围偏窄；只适合初步探索 | 允许继续参数摸底或低风险探索，不应用于最终性质 |
| `BLOCK` | scan 点未收敛；赝势更换后沿用旧策略；cutoff 与 output 不一致；目标 observable 未记录；scratch 混用不可追踪 | 不允许把该 cutoff policy 传递到下游 |

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| 总能收敛但力不稳定 | 只用 total energy 判断 | 加入 force/stress 目标量 |
| 换赝势后结果漂移 | cutoff policy 未重做 | 从最低 cutoff 重新扫描 |
| phonon 前出现虚频 | cutoff/ecutrho 或结构未足够严格 | 提高 cutoff 并复查 SCF/relax |

## 通用学习模板

使用 `<system>`、`<structure>`、`<pseudo>`、`<k_mesh>`、`<ecutwfc>`、`<ecutrho>` 等占位符记录个人学习任务。本仓库提供通用审阅框架，不保存具体计算结果。

## 记录模板

```text
pw.scf.<system>.ecut<value>.in
pw.scf.<system>.ecut<value>.out
cutoff-convergence.<system>.md
record.md
```

## 下游影响

- SCF 使用 cutoff policy。
- relax/vc-relax 应使用通过力/应力审阅的 cutoff。
- phonon 前应复查 cutoff 对频率的影响。

## 来源与边界

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE pseudopotentials page: <https://www.quantum-espresso.org/pseudopotentials/>
- 本仓库 output review 标准：[standards/output-review-checklist.md](../../standards/output-review-checklist.md)
- Canonical literature: [references/canonical-literature.md](../../references/canonical-literature.md)

`ecutwfc/ecutrho` 字段定义以 QE `INPUT_PW` 为准；赝势类型对 cutoff 的影响属于 stable 方法边界；具体 cutoff policy 是体系、赝势和目标 observable 相关判断，不应写成通用参数。
