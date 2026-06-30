# vdW corrections workflow

## 页面定位

- 对应学习路线：[learn/09-feature-expansion-map.md](../../learn/09-feature-expansion-map.md)
- 上游依赖：基础 SCF、relax、收敛性、结构和赝势记录
- 下游用途：需要弱相互作用描述的结构优化、静态 SCF 和专题 workflow
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

本页是 B 级高级边界页：目标是说明什么时候需要考虑 vdW correction、QE 输入边界在哪里、output 应如何审阅，以及它与基础 SCF/relax workflow 的依赖关系。本页不展开具体材料案例，不做完整 vdW 方法比较，也不提供未经来源核对的经验默认值。

## 适用问题

vdW correction 用于普通半局域交换关联泛函难以充分描述的长程或中程色散相互作用。进入本 workflow 的典型问题包括：

- 层间、分子间、吸附构型或有机/无机界面中，弱相互作用对几何、能量差或压力有显著影响。
- 结构优化目标包含距离、层间距、吸附高度、体积或相对构型稳定性，而这些量可能受色散相互作用控制。
- 文献、赝势说明或方法设计已经要求使用特定 vdW 方案，并且需要在 QE input/output 中把该方案记录为可追溯条件。

不应把 vdW correction 当作通用收敛修复工具。SCF 不收敛、力异常、真空层不足、k 点/cutoff 未收敛或结构本身错误时，应先回到基础 workflow 诊断。

## 进入该 workflow 前必须已经掌握什么

- [SCF workflow](../ground-state/scf.md)：能够审阅 `pw.x` 的 cutoff、k 点、occupation/smearing、自洽收敛、赝势加载和 `prefix/outdir`。
- [Relax workflow](../ground-state/relax.md)：能够区分 `JOB DONE`、电子 SCF 收敛、离子优化收敛、最终力和最终坐标。
- 结构边界：知道晶胞、真空、周期性方向、原子坐标和结构约束来自哪里；本仓库不负责结构构建。
- 赝势边界：知道赝势文件、来源、推荐 cutoff、生成时的 XC 信息，以及是否计划显式设置 `input_dft`。
- 方法边界：能说明为什么本任务需要 vdW correction，以及该选择来自文献、项目规范还是待验证假设。

## 关键 QE 程序或输入字段

主要程序是 `pw.x`。vdW correction 属于 ground-state Hamiltonian/energy model 的一部分，因此会影响 SCF 总能、力、应力，以及以该 ground state 为上游的 relax、static SCF、bands、DOS、phonon 和后处理可比性。

| 字段或程序 | 所属位置 | 边界说明 | 输出中如何验证 |
|---|---|---|---|
| `pw.x` | 程序 | 执行 `scf`、`relax`、`vc-relax` 等包含 vdW correction 的计算 | output 中确认 QE version、calculation type、XC/dispersion 相关回显、收敛状态 |
| `vdw_corr` | `&SYSTEM` | 选择部分半经验或后处理型 vdW correction；官方 `INPUT_PW` 列出可接受取值和默认行为，应以所用 QE 版本文档为准 | 在 output 中查找 vdW、dispersion、Grimme、TS、MBD、XDM、DFT-D 等相关回显；若无法定位，标记为 WARN/BLOCK 并核对 input 与版本 |
| `input_dft` | `&SYSTEM` | 显式指定交换关联泛函；若不指定，QE 会从赝势信息推断。使用 vdW-DF 等非局域泛函时，边界通常在 `input_dft`，而不是简单地把所有方法都归入 `vdw_corr` | output 中确认最终使用的 XC functional；记录是否覆盖了赝势内的 XC 信息 |
| `ATOMIC_SPECIES` / pseudo 文件 | input card / 文件 | 赝势必须来源明确，并与所用 XC/方法选择保持可解释关系；如果 `input_dft` 覆盖赝势 XC，需要在记录中说明理由 | output 列出加载的 pseudopotential 文件；记录 pseudo 来源与 XC 信息 |
| `calculation` | `&CONTROL` | `scf` 用于固定结构能量/电荷密度；`relax`/`vc-relax` 会让 vdW correction 参与力或应力相关优化 | output 确认计算类型，并分别审阅电子收敛、离子收敛、最终结构 |
| `CELL_PARAMETERS` / `ATOMIC_POSITIONS` | input cards | vdW correction 对层间距、吸附距离、体积和真空边界敏感；结构输入错误不能由 vdW 开关补救 | output 末尾最终坐标/晶胞是否与记录一致；必要时比较优化前后几何量 |
| `ecutwfc` / `ecutrho` / `K_POINTS` / occupation | `&SYSTEM` 和 `K_POINTS` | vdW 设置不替代基础数值收敛；不同 vdW 设置之间比较时，基础数值策略应保持可比 | output 中确认 cutoff、k 点、occupation/smearing 与基础 workflow 记录一致 |

### 与 XC、赝势和结构优化的关系

- `vdw_corr` 和 `input_dft` 不是同一个层级的选择。`input_dft` 定义交换关联泛函；`vdw_corr` 启用 QE 支持的特定色散修正。某些 vdW 相关方案以 `input_dft` 的非局域泛函形式进入，而不是只靠 `vdw_corr`。
- 赝势不是可忽略背景。QE 可以从赝势读取 XC 信息；如果 input 中显式覆盖，应把覆盖原因、赝势来源和 output 回显一起记录。
- 一旦 vdW correction 改变 Hamiltonian/能量模型，原来的无 vdW relax 结构不能自动视为同一方法下的优化结构。若目标性质依赖几何，通常需要在同一 vdW 设置下重新 relax 或 vc-relax，再做 static SCF。
- `relax` 审阅重点是力和最终坐标；`vc-relax` 还必须审阅 stress、压力和晶胞变化。对弱相互作用问题，结构变化本身往往就是需要记录的关键结果。

## output review 要点

```markdown
## vdW Output Review

- QE 程序:
- QE version:
- 计算类型:
- Structure / cell source:
- Pseudopotentials loaded:
- Pseudo XC information:
- `input_dft` in input:
- XC functional reported in output:
- `vdw_corr` in input:
- vdW / dispersion line found in output:
- Cutoff reported:
- K-points reported:
- Occupation / smearing:
- SCF convergence:
- Final total energy:
- For relax/vc-relax: final forces:
- For vc-relax: stress / pressure / final cell:
- Geometry quantities affected by vdW:
- Warnings:
- Scratch / restart status:
- Comparison target, if any:
- PASS / WARN / BLOCK:
- Reason:
- Allowed downstream workflows:
```

审阅时至少检查：

- output 是否能定位到最终使用的 XC functional，以及是否能定位到 vdW/dispersion 相关设置。若 input 设置了 `vdw_corr` 但 output 没有任何可核对痕迹，不应直接 PASS。
- `calculation='relax'` 或 `vc-relax` 时，最后一次电子 SCF 是否收敛、离子优化是否收敛、最终坐标是否已进入记录。
- `vc-relax` 时，stress、压力、晶胞变化和真空/周期性边界是否符合任务目标。
- 与无 vdW 或另一 vdW 设置比较时，必须确认结构、赝势、cutoff、k 点、occupation、QE version 和优化状态具有可比性；否则只能作为 WARN 级观察，不作为定量结论。
- 后续 bands、DOS、phonon 或 post-processing 必须读取同一方法下生成的 ground-state 数据；不要混用不同 `input_dft`/`vdw_corr` 的 `prefix/outdir`。

## 常见风险

- 把 `JOB DONE` 当作方法有效。程序结束只说明运行完成；仍需检查 SCF、力、应力、结构和 vdW 回显。
- 只改 `vdw_corr` 或 `input_dft` 后直接复用旧结构、旧 charge density 或旧下游结果，导致方法链不可追溯。
- 在没有核对官方文档和 QE 版本的情况下照抄 `vdw_corr` 取值。不同 QE 版本支持的选项和输出细节可能不同。
- 把所有 vdW 方法都归入 `vdw_corr`。非局域 vdW 泛函可能通过 `input_dft` 进入；方法边界应回到官方 `INPUT_PW`。
- 赝势、`input_dft` 和文献方法不一致，却没有在记录中说明。
- 用 vdW correction 掩盖结构构建错误、真空层不足、周期性方向设置错误或基础参数未收敛。
- 不同 vdW 设置之间直接比较总能，却没有保证相同结构优化状态和相同基础数值策略。

## 与基础 workflow 的关系

vdW correction 应放在基础 workflow 之上，而不是绕过基础 workflow：

```text
structure + pseudo + SCF numerical policy
  -> choose XC / vdW boundary
  -> pw.x relax or vc-relax with chosen method
  -> output review of SCF + forces/stress + vdW settings
  -> static SCF with the same method
  -> downstream workflow with method provenance recorded
```

- 对 [SCF workflow](../ground-state/scf.md)：vdW 设置改变 ground-state energy model，因此 SCF output review 必须额外记录 XC/vdW 信息；cutoff、k 点、occupation 和赝势审阅仍然保留。
- 对 [Relax workflow](../ground-state/relax.md)：如果目标性质依赖几何，应在选定 vdW 设置下完成结构优化，并保存最终结构；无 vdW relax 的最终坐标不能自动替代 vdW relax。
- 对 convergence workflow：vdW correction 不给出数值收敛豁免。cutoff、k 点、smearing、真空和结构约束仍需独立检查。
- 对 downstream workflow：bands/DOS/phonon 等下游任务应明确读取哪一个 `prefix/outdir`，并记录它对应的 `input_dft`、`vdw_corr`、结构和赝势。

## 记录模板

```text
pw.relax.<system>.vdw.in
pw.relax.<system>.vdw.out
pw.scf.<system>.vdw.in
pw.scf.<system>.vdw.out
record.md
pseudo-source.md
```

`record.md` 至少写清楚：为什么需要 vdW correction、`input_dft` 是否显式设置、`vdw_corr` 取值、赝势来源、结构来源、基础数值参数、运行命令、output 中如何确认 XC/vdW 设置、PASS/WARN/BLOCK 判断，以及允许进入的下游 workflow。

## 资料来源

- QE `pw.x` input reference, `&SYSTEM input_dft` and `vdw_corr`: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE documentation index: <https://www.quantum-espresso.org/documentation/>
