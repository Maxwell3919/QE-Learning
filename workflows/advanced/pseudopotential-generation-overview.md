# Pseudopotential generation overview

## 页面定位

- 对应学习路线：[learn/09-feature-expansion-map.md](../../learn/09-feature-expansion-map.md)
- 上游依赖：基础 SCF、收敛性、结构和赝势记录
- 下游用途：高级物性分析或专题 workflow
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 适用问题

本页用于判断“是否需要进入赝势生成/改造”以及“生成后怎样证明它还不能或可以进入生产计算”。它只给出边界、输入字段、output review 和验证关系，不展开完整赝势生成教程。

适合阅读本页的问题包括：

- 目标元素、价态、relativistic/SOC 能力或计算类型在现成 UPF 中无法满足。
- 需要理解 `atomic` / `ld1.x` 生成、测试和 UPF 文件之间的关系。
- 需要为自生成或改造赝势建立记录、cutoff scan 和 transferability validation 的最低审阅框架。
- 需要判断一个新 UPF 是否只能用于探索，还是可以进入 SCF、relax、phonon、DOS/bands 等下游 workflow。

不适合用本页解决的问题：

- 本页不提供逐行 `ld1.x` 输入模板。
- 本页不提供具体元素案例。
- 本页不做公共赝势库排名。
- 本页不替代基础的赝势选择、SCF 和 cutoff convergence workflow。

## 进入该 workflow 前必须已经掌握什么

- 能阅读 `pw.x` input 中的 `pseudo_dir`、`ATOMIC_SPECIES`、`ecutwfc`、`ecutrho`、`input_dft`、`noncolin/lspinorb` 等字段。
- 已完成基础 SCF workflow，知道如何从 output 确认赝势文件、PP type、valence electrons、relativistic 信息和 warning。
- 已掌握 [cutoff convergence workflow](../ground-state/cutoff-convergence.md)，知道更换赝势后必须重新建立 cutoff policy。
- 已掌握 [Pseudopotentials](../../theory-minimum/pseudopotentials.md) 中的最低概念：泛函一致性、valence 空间、NC/USPP/PAW 差异、SOC 对 fully relativistic 赝势的要求。
- 能区分“生成成功”“原子测试通过”“solid-state 测试通过”和“目标物性可用”这几个不同层级。
- 已明确目标 workflow 的敏感量：总能、力、应力、能带、带隙、声子频率、磁性、SOC splitting 或其他性质。

## 关键 QE 程序或输入字段

| 字段或程序 | 所属阶段 | 用途 | 边界和审阅点 |
|---|---|---|---|
| `atomic` / `ld1.x` | atomic / PP generation | QE 的原子与赝势生成/测试程序 | 这是高级工具入口；本页只要求知道生成、测试、UPF 输出和审阅关系，不要求从零调参。 |
| `&input` | `ld1.x` | 原子计算和通用控制 | 关注 `atom` 或 `zed`、`config`、`dft`、`rel`、`iswitch`、`prefix`、`verbosity`。`iswitch=3` 对应 PP generation，`iswitch=2` 对应 PP test。 |
| `config` / all-electron cards | `ld1.x` | reference electronic configuration | 必须记录参考构型、占据、是否 relativistic，以及它与目标化学环境的关系。不要只记录“默认构型”。 |
| `dft` | `ld1.x` / `pw.x` | 交换关联泛函 | 生成泛函应与后续 `pw.x` 使用保持一致；若 `pw.x` 中手动设置 `input_dft`，必须检查是否与 UPF 元数据冲突。 |
| `rel` | `ld1.x` | relativistic 分支 | `rel=0/1/2` 分别对应 non-relativistic、scalar relativistic、full relativistic with spin-orbit 的边界；SOC workflow 需要能支持 spin-orbit 的赝势。 |
| `&inputp` | `ld1.x` | PP generation 控制 | 关注 `zval`、`pseudotype`、`file_pseudopw`、`lloc`、`nlcc`、`lpaw`、projector / cutoff radius 相关卡片。不要在未理解物理含义时机械套用。 |
| PP generation cards | `ld1.x` | 被 pseudize 的轨道与半径/能量 | 记录每个通道的 orbital、occupation、pseudization energy、cutoff radius；这是 transferability 与 cutoff 需求的核心 provenance。 |
| `&test` | `ld1.x` | PP test | 用于比较测试构型、cutoff 区间、logarithmic derivatives 或 ghost 相关信号；通过它只能说明原子级测试表现，不等价于材料 workflow 通过。 |
| UPF file | QE pseudopotential format | `pw.x` 读取的赝势文件 | 审阅 header/metadata：element、functional、PP type、relativistic、z valence、generation date/code、recommended cutoff、author/source/license 等。 |
| `ATOMIC_SPECIES` | `pw.x` | 结构中的 species 到 UPF 文件映射 | 新 UPF 必须先在基础 SCF 中确认加载的是目标文件，而不是目录中同名旧文件。 |
| `ecutwfc` / `ecutrho` | `pw.x` | plane-wave cutoff validation | 生成或更换 UPF 后必须重新做 cutoff scan；NC、USPP、PAW 对 `ecutrho` 的需求不同。 |
| validation tests | `ld1.x` + `pw.x` | 从原子测试到材料测试 | 原子测试、simple solid-state SCF、cutoff convergence、目标性质复查应分层记录，不要把单一 PASS 扩大到所有 workflow。 |

## output review 要点

### `ld1.x` / atomic output

- QE / `ld1.x` version、输入文件名、`prefix`、`iswitch` 是否与记录一致。
- `atom` 或 `zed`、`config`、`dft`、`rel` 是否按预期被读取。
- all-electron reference calculation 是否收敛，能级、占据和总能是否有明显异常。
- PP generation 阶段是否实际写出 `file_pseudopw` 指向的 UPF 文件。
- 每个 pseudized channel、projector、cutoff radius、local channel、NLCC/PAW/USPP 设置是否能从输入和输出追溯。
- test 阶段是否包含多个测试构型，而不是只重复 reference configuration。
- logarithmic derivatives、AE vs pseudo energy differences、ghost-state 相关测试或 spherical Bessel basis 测试是否给出可审阅结果。
- output 中是否有 warning、non-convergence、unbound state、ghost、bad logarithmic derivative、异常占据或格式写出问题。

### UPF review

- 文件是否为 QE 可读的 UPF，且文件名、路径、hash、来源和生成脚本/输入能追溯。
- 元数据是否包含 element、functional、PP type、relativistic treatment、z valence、generation code/version、author/source/license。
- 是否包含推荐或可参考的 wavefunction / charge-density cutoff；若没有，必须标记为“需自行 scan”，不能沿用旧 cutoff。
- 对 SOC、PAW、USPP、GIPAW 或特殊重构信息的支持是否与目标 workflow 匹配。

### `pw.x` validation output

- output 中 pseudopotential 段加载的 UPF 是否是新文件，`ATOMIC_SPECIES` 映射是否正确。
- `ecutwfc`、`ecutrho`、k mesh、smearing 和结构是否按 validation 计划固定或扫描。
- SCF 是否收敛，是否有 charge sloshing、negative rho、too few bands、symmetry/pseudo mismatch 等 warning。
- cutoff scan 是否覆盖目标性质，而不只看 total energy。
- relax/force/stress/phonon/SOC 等下游敏感任务是否用独立 validation 结果支持。

### 最低记录字段

```markdown
## pseudopotential generation / validation review

- PP label:
- Element:
- UPF file:
- UPF hash:
- QE / ld1.x version:
- Generation input:
- Reference configuration:
- XC functional:
- Relativistic treatment:
- PP type:
- z valence:
- Local channel / projectors:
- NLCC / PAW / USPP settings:
- Recommended cutoff in UPF:
- Actual cutoff scan:
- Atomic tests:
- Solid-state validation systems:
- Target workflow validated:
- Warnings:
- PASS / WARN / BLOCK:
- Allowed downstream workflows:
- Reason:
```

## 常见风险

- 把 `ld1.x` 成功写出 UPF 误认为赝势可用于生产计算。
- 只在 reference configuration 上测试，未做 transferability 检查。
- 没有记录 `config`、`dft`、`rel`、`zval`、PP type、projector 和 cutoff radius，导致结果不可复现。
- 生成泛函和 `pw.x` 计算泛函不一致，或者混用不同库/版本的 UPF。
- SOC/noncollinear workflow 使用不支持目标 relativistic treatment 的赝势。
- 沿用旧赝势的 `ecutwfc/ecutrho`，没有重新做 cutoff convergence。
- 只用总能判断赝势，忽略力、应力、声子、能带、磁性或目标性质。
- 只在一个结构或一个体积点验证，直接扩展到缺陷、slab、高压、带电体系或 phonon。
- 忽略 ghost state、logarithmic derivative 异常或测试输出 warning。
- 没有保存 UPF hash、生成输入、测试输入、测试输出和 validation record。

## 与基础 workflow 的关系

- 本页依赖 [Pseudopotentials](../../theory-minimum/pseudopotentials.md) 的基础概念；没有完成基础赝势审阅前，不应进入生成。
- 新 UPF 进入 `pw.x` 前，先回到基础 SCF workflow，确认文件加载、泛函、PP type、valence 和 warning。
- 新 UPF 进入 relax、DOS、bands、phonon 或 SOC 前，必须回到 [cutoff convergence workflow](../ground-state/cutoff-convergence.md) 建立新的 `ecutwfc/ecutrho` policy。
- 如果目标是 force/stress/phonon，应把这些目标量纳入 validation，而不是只沿用 SCF 总能 cutoff。
- 本页的 PASS 只表示“在记录的 validation 范围内可用”。一旦更换结构族、价态、压力范围、磁态、SOC 设置或目标物性，应重新定义 validation。
- 高级 workflow 中的赝势生成结果应优先作为受控输入资产管理：UPF、生成输入、测试输入、测试输出、cutoff record 和下游许可范围必须一起保存。

## 资料来源

- QE `ld1.x` input description: <https://www.quantum-espresso.org/Doc/INPUT_LD1.html>
- QE pseudopotentials page: <https://www.quantum-espresso.org/pseudopotentials/>
- QE Notes on pseudopotential generation: <https://www.quantum-espresso.org/Doc/user_guide_PDF/pseudo-gen.pdf>
- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE documentation: <https://www.quantum-espresso.org/documentation/>
