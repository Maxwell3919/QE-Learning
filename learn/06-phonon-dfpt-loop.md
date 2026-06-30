# Phonon DFPT Loop

## 能力目标

掌握 QE 中 phonon / lattice dynamics 的 DFPT response workflow：知道它依赖已经审阅过的结构和 SCF ground state，不是对任意 `pw.x` 输出做普通后处理；能区分 Gamma phonon、q-grid phonon dispersion、dielectric / Born response 分支；能从 output 中判断本轮结果是 PASS、WARN 还是 BLOCK，并说明允许进入哪些下游 workflow。

## 完成判据

- 能说明 phonon 属于 DFPT response workflow：`ph.x` 计算体系对原子位移或电场扰动的线性响应，输入质量直接受结构优化、SCF 阈值、cutoff、k 点、smearing 和 q-grid 影响。
- 能区分 Gamma phonon 与 q-grid phonon。
- 能说明 `ph.x`、`dynmat.x`、`q2r.x`、`matdyn.x` 分工。
- 能说明 ASR、negative frequency、Born effective charge、dielectric tensor 的最低理解和常见误用。
- 能说明 phonon 为什么通常要求比普通 bands/DOS 更严格的结构、SCF 和收敛审阅。
- 能写出 phonon output review，并给出 PASS / WARN / BLOCK 和下游准入。

## Phonon 在 QE workflow 中的位置

phonon 不是“已经有 charge density 之后画一张图”的后处理。它属于 DFPT response workflow：先用 `pw.x scf` 得到 ground state，再用 `ph.x` 解线性响应方程，得到 dynamical matrices 或电场响应相关量。

因此，phonon 的上游准入条件包括：

- 结构已经通过 relaxation output review，残余 force / stress 与本轮 phonon 目标匹配。
- final static SCF 已独立完成，并记录 cutoff、k 点、occupation、smearing、SCF threshold 和 warning。
- phonon 分支目标已经明确：Gamma modes、q-grid dispersion、phonon DOS、dielectric / Born response，或 debugging。

如果上游结构或 SCF 只是普通电子结构计算标准，phonon 结果最多进入 WARN 或 BLOCK，不能直接作为动力学稳定性、软模、IR/Raman 或热学分析的准入证据。

## Gamma phonon 与 q-grid phonon

Gamma phonon 只计算 q = 0 的响应，核心用途是审阅 Gamma 点振动模式、acoustic modes、部分 IR/Raman 前提，以及在有效条件下读取 dielectric tensor 和 Born effective charge。它的最小数据链是：

```text
relaxed structure + strict SCF
  -> ph.x at q = 0
  -> Gamma dynamical matrix
  -> dynmat.x
  -> Gamma modes / response tensors when valid
```

q-grid phonon 用 uniform q-grid 计算多个 q 点上的 dynamical matrices，再转换为 real-space force constants，并沿指定 q-path 插值得到 dispersion 或 DOS。它的最小数据链是：

```text
relaxed structure + strict SCF
  -> ph.x with ldisp=.true., nq1/nq2/nq3
  -> dynamical matrices
  -> q2r.x
  -> real-space force constants
  -> matdyn.x
  -> frequencies on q-path or phonon DOS
```

两者不能互相替代：

| 问题 | Gamma phonon | q-grid phonon |
|---|---|---|
| q 点范围 | 只看 q = 0 | 看 uniform q-grid，并可插值到 q-path |
| 常见输出 | Gamma modes、acoustic modes、可选 dielectric / Born response | dyn 文件链、force constants、dispersion / DOS |
| 主要后处理 | `dynmat.x` | `q2r.x`、`matdyn.x` |
| 常见误用 | 用单个 Gamma 结果解释全 Brillouin zone 稳定性 | 用过粗 q-grid 解释细节或把插值异常当最终结论 |

## QE 程序分工

`ph.x` 是 DFPT phonon workflow 的核心程序。它读取 SCF ground-state 数据，计算原子位移扰动；在 Gamma 分支中也可请求与电场相关的响应。output 中必须检查每个 q 点、每个 irreducible perturbation 是否收敛，以及是否生成完整 dynamical matrix 文件。

`dynmat.x` 用于 Gamma 点 dynamical matrix 的对角化和模式分析。它适合检查 Gamma modes、acoustic modes、ASR 后处理效果，以及 Gamma response 分支相关输出；它不是 q-grid dispersion 的替代品。

`q2r.x` 读取 q-grid dynamical matrices，把它们 Fourier transform 成 real-space interatomic force constants。这里要检查 dyn 文件是否齐全、`fildyn` 前缀是否一致、`flfrc` 是否生成、`zasr` 是否记录清楚。

`matdyn.x` 读取 real-space force constants，在指定 q-path 上插值得到 phonon frequencies，或生成 phonon DOS 所需数据。这里要检查 q-path、ASR 设置、frequency file、Gamma 附近 acoustic branches 和 negative frequency。

## 最低概念边界

ASR 是 acoustic sum rule，用来处理平移不变性相关的数值误差。它可以帮助审阅 Gamma 附近 acoustic branches 是否合理，但不能修复未充分优化的结构、过松的 SCF、错误的 q-grid 或未收敛的 perturbation。必须记录 ASR 前后结果和使用位置，例如 `dynmat.x`、`q2r.x` 或 `matdyn.x`。

Negative frequency 通常表示 output 中出现虚频或负的频率表示。小的、接近 Gamma 的负频可能来自 ASR、结构残余力、cutoff / k 点 / q-grid、SCF 阈值、smearing 或插值误差；明显、可重复、对参数不敏感的负频才可能指向真实动力学不稳定。单次粗计算只能给出复查线索。

Born effective charge 是原子位移与极化响应之间的有效耦合量，不等同于静态化学价态。Dielectric tensor 描述电场响应。二者属于 Gamma DFPT response 分支，通常通过 `ph.x` 中的 `epsil` 请求，并且只在物理条件适用、electric-field perturbation 收敛、output 没有相关限制 warning 时解释。

介电张量、Born effective charge、LO-TO splitting、IR/Raman 相关判断都需要记录适用条件。若体系或计算设置不满足电场响应解释条件，应给 WARN 或 BLOCK，而不是硬解释输出中的数字。

## 对上游和网格的敏感性

phonon 对数值设置的敏感性高于很多普通电子结构后处理。学习本页时必须形成下面的审阅习惯：

- 结构优化：残余 force / stress 会直接污染 force constants 和 acoustic branches；不能把 `JOB DONE` 当结构可用证据。
- Static SCF：phonon 前应使用最终结构重跑 static SCF，并审阅 `conv_thr`、estimated accuracy、warning 和 restart / scratch 状态。
- Cutoff 与 pseudopotential：`ecutwfc`、`ecutrho` 和赝势质量会影响力常数和频率，尤其影响小虚频与声学支。
- k 点与 smearing：金属或小能隙体系对 k mesh、occupation 和 smearing 更敏感；phonon 记录必须保留这些设置。
- q-grid：q-grid 决定 force constants 和 interpolation 质量；过粗 q-grid 不能用于解释精细 dispersion、软模位置或 DOS 细节。
- 维度与长程相互作用：低维和极性体系需要额外检查真空、边界条件、non-analytic correction 和方向依赖，本页只要求知道这些是准入条件，不展开高级处理。

## 可验证输出

- `ph.x` perturbation convergence 阅读笔记：每个 q 点、每个 irreducible perturbation 是否收敛。
- Gamma phonon 或 q-grid phonon 的数据链说明：`*.dyn* -> *.fc -> *.freq`，或 `*.dynG -> dynmat.x`。
- ASR 设置、ASR 前后差异和 acoustic modes 判断记录。
- Negative frequency 的位置、大小、参数敏感性和下一步复查策略。
- `epsil`、Born effective charge、dielectric tensor 是否请求、是否出现、是否物理适用的记录。
- PASS / WARN / BLOCK 和 allowed downstream workflows。

## Phonon output review 最小清单

```markdown
## phonon output review

- Workflow branch: Gamma phonon / q-grid dispersion / phonon DOS / dielectric-Born / debugging
- Upstream structure status:
- Upstream static SCF status:
- QE version:
- Program chain:
- Pseudopotentials:
- Cutoff reported:
- K-points and occupations:
- SCF threshold and convergence:
- ph.x q-point or q-grid:
- ph.x perturbation convergence:
- Dynamical matrix files:
- q2r.x force-constant file:
- dynmat.x or matdyn.x output file:
- ASR / zasr setting:
- Gamma acoustic branches:
- Negative frequencies:
- Dielectric tensor / Born effective charge branch:
- Warnings:
- PASS / WARN / BLOCK:
- Reason:
- Allowed downstream workflows:
```

审阅顺序应从上游到下游：先确认结构和 static SCF 是否允许进入 phonon，再看 `ph.x` perturbation 是否收敛，然后检查 dyn / fc / freq 文件链，最后才解释频率、模式、ASR、negative frequency 和响应张量。

## PASS / WARN / BLOCK 与下游准入

PASS 表示上游结构和 static SCF 已通过审阅，`ph.x` perturbation 收敛，文件链完整，ASR 设置可追踪，negative frequency 不存在或已被合理解释，且本轮分支目标与输出匹配。PASS 后可按分支进入 Gamma mode 分析、dispersion、phonon DOS、dielectric / Born、IR/Raman 或更高级 workflow。

WARN 表示结果可以作为学习记录或复查线索，但不应作为最终物理结论。例如 acoustic modes 有小偏移、存在小的参数敏感 negative frequency、q-grid 偏粗、SCF 阈值可能偏松、`epsil` 分支输出存在但适用条件尚未完全说明。WARN 的记录必须写清下一步要复查哪个变量。

BLOCK 表示不能进入下游解释。例如结构未充分优化、static SCF 未通过、`ph.x` perturbation 不收敛、q-grid dyn 文件不完整、`q2r.x` 没有读全文件、`matdyn.x` 读取错误 force constants、在不适用条件下解释 dielectric / Born 输出，或用单个 Gamma 结果声称全 q-space 稳定。

下游准入应写成具体 workflow，而不是泛泛写“可继续”。例如：允许进入 Gamma mode review、允许进入 phonon dispersion plot、只允许进入 debugging、不允许进入 IR/Raman、暂不允许进入 EPC / EPW / thermodynamics。

## 推荐阅读

- [workflows/phonon/gamma-phonon.md](../workflows/phonon/gamma-phonon.md)
- [workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)
- [workflows/phonon/phonon-debugging.md](../workflows/phonon/phonon-debugging.md)
- [theory-minimum/dfpt-phonons.md](../theory-minimum/dfpt-phonons.md)
- [theory-minimum/dielectric-born-charge.md](../theory-minimum/dielectric-born-charge.md)

## 后续进入哪个 workflow

完成后可进入 Gamma phonon、phonon dispersion、phonon DOS、dielectric/Born effective charge、IR/Raman 或 electron-phonon overview。若 output review 是 WARN 或 BLOCK，应先进入 phonon debugging，而不是继续堆后处理。

## 资料来源

- QE INPUT_PH reference: <https://www.quantum-espresso.org/Doc/INPUT_PH.html>
- QE INPUT_Q2R reference: <https://www.quantum-espresso.org/Doc/INPUT_Q2R.html>
- QE INPUT_MATDYN reference: <https://www.quantum-espresso.org/Doc/INPUT_MATDYN.html>
- QE INPUT_DYNMAT reference: <https://www.quantum-espresso.org/Doc/INPUT_DYNMAT.html>
- QE PHonon user guide: <https://www.quantum-espresso.org/Doc/ph_user_guide/>
- Pranab Das phonon tutorial: <https://pranabdas.github.io/espresso/hands-on/phonon/>
- Kyoto University QE phonon DokuWiki: <https://www2.yukawa.kyoto-u.ac.jp/~koudai.sugimoto/dokuwiki/doku.php?id=quantumespresso%3Aphonon%3A%E3%83%95%E3%82%A9%E3%83%8E%E3%83%B3%E3%81%AE%E8%A8%88%E7%AE%97>
