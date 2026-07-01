# Gamma phonon workflow

## 本页解决什么问题

Gamma phonon 用于计算 Brillouin zone 中心的振动模式，是 IR/Raman、Born effective charge、dielectric tensor 和 LO-TO splitting 等响应性质的入口。它回答的是 zone-center 模式是否可信，不回答全 Brillouin zone 的动力学稳定性。Gamma phonon 属于 DFPT response workflow，依赖已经审阅过的结构、final static SCF、赝势、cutoff、k mesh、occupation 和 `prefix/outdir` 数据链。

这页应按 handbook 使用：先确认 ground-state 数据链是否可信，再确认 `ph.x` 对 Gamma perturbations 的线性响应是否收敛，最后才解释 `dynmat.x` 给出的频率、模式和可选响应张量。Gamma 结果常被误用成“声子稳定性证明”，因此本页把它限定为 zone-center 证据；如果目标是 off-Gamma soft mode、zone-boundary instability 或热力学 phonon 数据，应转入 q-grid dispersion workflow。

## 上游依赖

- 结构状态：来自可信 `relax` 或 `vc-relax` 后的 final static SCF；如果固定晶格，应记录原因。
- SCF 状态：`pw.x scf` 已通过 [SCF workflow](../ground-state/scf.md) 和 [output review checklist](../../standards/output-review-checklist.md)。
- 文件链：`ph.x` 的 `prefix` 和 `outdir` 必须指向上游 final static SCF 的数据，不能混用旧 scratch。
- 赝势和泛函：与上游 SCF 保持一致；更换赝势或泛函后需要重新审阅 SCF 与 phonon。
- 响应分支：若开启 `epsil` 或 Raman 相关分支，需要先确认体系、版本和官方文档支持条件。

## 计算图

```text
relaxed structure
  -> pw.x final static scf
  -> ph.x at q = 0
  -> <system>.dynG
  -> dynmat.x
  -> Gamma frequencies / modes / optional response tensors
  -> output review
```

这条计算图的核心不是命令顺序，而是数据所有权。`ph.x` 不重新建立一个新的 ground state；它读取 `pw.x scf` 留在 `prefix/outdir` 中的电荷密度、波函数和相关数据，并在 q = 0 上求解线性响应方程。`dynmat.x` 只读取当前 Gamma dynamical matrix 并输出模式表、频率和可选的 ASR 后处理结果。只要 final static SCF、scratch 目录或 `fildyn` 文件有一环不可追踪，后面的频率表就只能作为未准入数据。

## 关键 QE 输入对象

Gamma phonon 的输入对象分成两个层次。`ph.x` 的 `&INPUTPH` 决定从哪个 SCF 数据链读取 ground state、求解哪些 perturbations、写出哪个 dynamical matrix；`dynmat.x` 的 `&INPUT` 决定对哪个 dynamical matrix 做模式分析和是否施加 ASR。两者共享的只是 `fildyn` 文件边界，不能把 `dynmat.x` 的后处理设置倒推为上游 DFPT 已经可靠。

```fortran
&INPUTPH
  prefix = '<system>',
  outdir = '<scratch_dir>',
  tr2_ph = <phonon_threshold>,
  trans = .true.,
  epsil = <true_or_false>,
  fildyn = '<system>.dynG',
  recover = .true.,
/
0.0 0.0 0.0

&INPUT
  fildyn = '<system>.dynG',
  asr = '<asr_scheme>',
  filout = 'dynmat.<system>.out',
/
```

| 字段 | 程序 | 判断意义 | 联动对象 | output 中如何验证 |
|---|---|---|---|---|
| `prefix` / `outdir` | `ph.x` | 指向上游 SCF 的 ground-state 数据 | `pw.x scf` 的 scratch | `ph.x` header、读取数据路径、warning |
| `tr2_ph` | `ph.x` | DFPT perturbation 自洽阈值 | SCF `conv_thr`、mixing、smearing | 每个 perturbation 的 convergence 信息 |
| `trans` | `ph.x` | 是否计算 phonon perturbations | `epsil`、EPC 分支 | output 是否列出 irreducible representations 和 modes |
| `epsil` | `ph.x` | 请求 dielectric tensor 和 Born effective charge | 绝缘体条件、Gamma limit | output 中 dielectric/Born 段落 |
| `fildyn` | `ph.x` / `dynmat.x` | Gamma dynamical matrix 文件 | `dynmat.x` 输入 | `dynmat.x` 是否读取当前 `fildyn` |
| `asr` | `dynmat.x` | 对 Gamma acoustic modes 施加 acoustic sum rule / ASR | 结构残余力、数值收敛 | ASR 前后 acoustic modes 的变化 |

## 命令与文件边界

```bash
ph.x -in ph.gamma.<system>.in > ph.gamma.<system>.out
dynmat.x -in dynmat.<system>.in > dynmat.<system>.out
```

`ph.x` 读取的是上游 SCF 的 `prefix/outdir`，输出 `fildyn` 所指定的 dynamical matrix。这个 `prefix/outdir` 应来自 final static SCF，而不是 relax 过程中的临时电子步，也不是同名体系的旧 scratch。`dynmat.x` 只对已有 Gamma dynamical matrix 做对角化和模式输出；它不能修复上游 SCF、结构或 DFPT perturbation 未收敛的问题。个人记录中应保留 `pw.scf.<system>.out`、`ph.gamma.<system>.out`、`dynmat.<system>.out`、`fildyn` 文件名、ASR 设置和是否开启 `epsil`。

文件命名需要能回答一个简单问题：当前 `dynmat.x` 到底读取了哪一次 `ph.x` 产生的 Gamma matrix。若同一目录里有多次 Gamma 测试、不同 ASR 试验或不同 `epsil` 分支，记录中应明确输入、输出和 `fildyn` 的对应关系。一个看似合理的频率表，如果无法追溯到当前 final static SCF 和当前 `ph.x` output，应降级为文件链风险。

## Output review

Output review 先看数据链，再看 perturbation convergence，最后看频率和模式解释。`JOB DONE`、频率表或 tensor 段落都不是单独的准入证据；它们必须和正确的 q-vector、收敛的 irreducible representations、当前 `fildyn` 以及上游 SCF 审阅记录一起出现，才构成可进入下游的 Gamma phonon 证据。

| 检查项 | 从哪里看 | 能证明什么 | 不能证明什么 |
|---|---|---|---|
| 上游 SCF 被正确读取 | `ph.x` header、`prefix/outdir`、warning | DFPT 使用了目标 ground-state 数据 | SCF 已对 phonon 目标充分收敛 |
| q 点为 Gamma | `ph.x` q-vector 段落 | 当前计算是 zone-center phonon | 全 BZ phonon 稳定 |
| perturbation 收敛 | 每个 irreducible representation / perturbation 的收敛信息 | 线性响应迭代完成 | cutoff、k mesh、smearing 已收敛 |
| Gamma acoustic modes | `dynmat.x` 频率表 | 平移模式是否接近零 | 小偏差是否一定是物理不稳定 |
| imaginary frequency | `ph.x` 或 `dynmat.x` 频率符号和单位 | 需要进入虚频 triage | 真实动力学不稳定已成立 |
| dielectric / Born 分支 | `ph.x` tensor 输出 | `epsil` 分支是否执行并输出张量 | 金属或不适用体系中可解释该张量 |
| 文件链 | `fildyn`、`dynmat.x` 输入输出 | 后处理读取了当前 dynamical matrix | 上游数据没有旧 scratch 污染 |

## 收敛与可靠性

- 程序完成只说明 `ph.x` 或 `dynmat.x` 正常结束，不说明 Gamma mode 可用于科研结论。
- Gamma phonon 对结构残余力、SCF `conv_thr`、cutoff、k mesh、occupation 和 smearing 敏感；这些上游状态必须进入记录。
- Gamma 无虚频不能证明完整 phonon dispersion 稳定；需要 q-grid dispersion 才能审阅 zone-boundary 或 off-Gamma soft modes。
- Gamma 有小虚频时，应先区分 acoustic numerical artifact、结构未充分优化、DFPT 未收敛和可能的真实软模。
- ASR 是后处理约束，用来审阅平移不变性相关误差，不能替代结构优化、SCF 或 DFPT 收敛。

可靠性判断还要区分“响应方程收敛”和“物理解释成立”。`ph.x` perturbations 全部收敛，只说明当前输入对象下的线性响应方程已求解；cutoff、k mesh、occupation、smearing、赝势和结构残余力是否足以支撑目标谱学或 Born charge 解释，需要沿用上游 workflow 的收敛证据。ASR 后 acoustic modes 接近零是一个一致性信号，但如果 ASR 前后差异很大，或者 ASR 掩盖了明显虚频，就应回到结构、SCF 和 DFPT 收敛排查。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | 上游 final static SCF 为 PASS；`prefix/outdir` 指向可信 ground-state 数据；`ph.x` perturbations 全部收敛；`dynmat.x` 读取当前 `fildyn`；acoustic modes、频率符号、单位和 eigenvector/context 已审阅；`epsil` 分支若使用则条件和输出完整 | 可进入 dielectric/Born、IR/Raman 或作为 q-grid phonon 的前置审阅；仍不得声明全 BZ 稳定 |
| WARN | acoustic modes 有小偏离、单个低频模式需复查、ASR 前后变化偏大、`epsil` 适用条件尚未充分说明，或当前结果只用于诊断 | 可进入受控复查、参数敏感性检查或 q-grid 计划，不应写入最终稳定性或谱学结论 |
| BLOCK | 上游 SCF/结构为 BLOCK；没有 final static SCF；`prefix/outdir` 不可信；`ph.x` perturbation 未收敛；`fildyn` 混用；明显虚频未 triage；用 ASR 修补坏数据；把 Gamma 结果直接外推成全 BZ 稳定性结论 | 不允许进入 IR/Raman、Born charge 解释、稳定性声明或图件归档 |

## 常见误区

- 把 Gamma phonon 当成完整 phonon dispersion。
- 只看 `JOB DONE` 或频率表，不检查 perturbation convergence。
- 用 ASR 消掉 acoustic 偏差后直接宣布结果可信。
- 开启 `epsil` 后不检查 metal/insulator 条件和张量输出。
- `dynmat.x` 读取了旧 `fildyn`，但仍与当前 SCF 记录合并。
- 不记录频率单位、负号约定和 ASR 设置。

## 下游影响

Gamma phonon 是 [dielectric tensor and Born effective charge workflow](dielectric-born-effective-charge.md) 与 [IR and Raman workflow](ir-raman.md) 的上游。它也为 [phonon debugging workflow](phonon-debugging.md) 提供 acoustic modes 和 zone-center 虚频证据。若目标是全 Brillouin zone 审阅，应进入 [phonon dispersion workflow](phonon-dispersion-dfpt.md)。

下游记录应继承本页的状态标签。Gamma `PASS` 可以支持 zone-center response 或作为 dispersion 前的 sanity check，但不能自动授权 phonon DOS、热学量、EPC 或相稳定性结论。Gamma `WARN` 适合保留为诊断证据，尤其是用于决定是否重做 final SCF、加严 DFPT 收敛或检查 eigenvector；Gamma `BLOCK` 则意味着所有依赖该 `fildyn` 的张量、模式图和解释都应停止传播。

## 来源与边界

- Stable: `ph.x` 的 `tr2_ph`、`trans`、`epsil`、`fildyn` 和 q 点输入语义以 QE `INPUT_PH` 为准；`dynmat.x` 的 `fildyn`、`asr` 和输出语义以 QE `INPUT_DYNMAT` 为准。
- Boundary: Gamma phonon 只能审阅 zone-center modes；全 Brillouin zone、低维边界和极性长程修正需要对应 workflow。
- Inference: 本页要求 final static SCF、可信 `prefix/outdir`、perturbation convergence、ASR 前后审阅和虚频 triage，是 workflow 准入标准；它不等同于 QE 官方给出的普适数值参数推荐。
- Internal standard: PASS / WARN / BLOCK 按 [pass-warn-block.md](../../standards/pass-warn-block.md) 与 [output-review-checklist.md](../../standards/output-review-checklist.md) 记录。

## 资料来源

- QE INPUT_PH reference: <https://www.quantum-espresso.org/Doc/INPUT_PH.html>
- QE INPUT_DYNMAT reference: <https://www.quantum-espresso.org/Doc/INPUT_DYNMAT.html>
- QE PHonon user guide: <https://www.quantum-espresso.org/Doc/ph_user_guide/>
- Baroni et al., phonons and related crystal properties from DFPT, Reviews of Modern Physics.
- 本仓库：[theory-minimum/dfpt-phonons.md](../../theory-minimum/dfpt-phonons.md)、[physics-judgement/09-phonons-soft-modes-and-dynamical-stability.md](../../physics-judgement/09-phonons-soft-modes-and-dynamical-stability.md)
