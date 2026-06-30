# DFPT and phonons

## QE 中对应的问题

QE phonon workflow 用 DFPT 计算原子位移、电场等微扰下的线性响应。它在基态电荷密度和波函数的基础上求解响应方程，再把响应量组装成 dynamical matrix；这比读取已有 SCF 输出的普通后处理更依赖上游数值质量。对 q-grid dispersion，还要通过 `q2r.x` 把 dynamical matrices 转换为 real-space force constants，再用 `matdyn.x` 沿目标 q-path 或网格插值。对单个 Gamma phonon，常见链路是 `ph.x -> dynmat.x`。

phonon 对上游数值质量更敏感，核心原因是频率来自能量对原子位移的二阶曲率，任何 SCF 噪声、结构残余力、基组误差、采样误差或占据误差都会被放大到 force constants 和低频模式中：

- SCF：`ph.x` 依赖稳定的基态电荷密度、势和波函数。普通总能量看似收敛时，响应方程仍可能对 `conv_thr`、mixing 和电子态近简并很敏感。
- 结构：phonon 默认围绕当前结构的平衡点做二阶展开。若 relax 后残余力或应力仍大，Gamma 声学支、软模和 negative frequency 可能反映的是未平衡结构，而不是真实动力学不稳定。
- cutoff：dynamical matrix 对力常数敏感，`ecutwfc/ecutrho` 不足会让力和应力误差进入二阶响应，表现为声学支漂移、频率整体偏移或虚频。
- k 点与 q-grid：k 点决定电子响应采样，q-grid 决定 force constants 的实空间范围和插值质量。k 点太粗会污染每个 q 点的响应，q-grid 太粗则不能可靠解释 dispersion 细节。
- smearing：金属和小带隙体系的 phonon 对费米面附近占据变化敏感。smearing 类型和宽度会改变电子响应，进而影响低频模式、软模和 Kohn anomaly 等判断。

本页只说明使用 QE 判断 phonon 结果所需的最低概念，不写完整 DFPT 推导，也不使用具体材料案例。

## 输入字段表

| 字段 | 所属程序 | 用来回答的问题 | 需要联动检查 | 常见误用 |
|---|---|---|---|---|
| `prefix` | `pw.x/ph.x` | `ph.x` 是否读取同一个 SCF 基态 | `outdir`、SCF output、scratch 内容 | SCF 与 phonon 使用不同前缀却当作同一任务 |
| `outdir` | `pw.x/ph.x` | `ph.x` 是否能找到波函数、电荷密度和 restart | scratch 路径、权限、是否清理过 | 复制 input 后忘记同步 scratch |
| `conv_thr` | `pw.x` | 上游 SCF 是否足够严格 | SCF estimated accuracy、metal smearing、k 点 | 用普通 DOS/bands 精度直接进入 phonon |
| `ecutwfc/ecutrho` | `pw.x` | 平面波基组是否足够支撑力常数 | 赝势建议值、force/stress 收敛、frequency 收敛 | 只按总能量收敛，不检查力和 phonon |
| `K_POINTS` | `pw.x` | 电子响应采样是否足够 | 金属/绝缘体、smearing、目标 q 点 | k 点过粗时解释软模或小虚频 |
| `occupations/smearing/degauss` | `pw.x` | 占据处理是否适合 phonon | 是否金属、费米面、SCF 稳定性 | 改 smearing 后不重新审阅 phonon |
| `tr2_ph` | `ph.x` | 每个 perturbation 的响应方程收敛到什么精度 | `conv_thr`、ph output 中各扰动收敛行 | 设得过松导致频率漂移或假虚频 |
| `trans` | `ph.x` | 是否计算原子位移声子扰动 | ph output 的 perturbation 列表 | 关闭后仍期待 phonon modes |
| `epsil` | `ph.x` | 是否计算介电张量和 Born effective charge 分支 | Gamma、绝缘体条件、LO-TO/non-analytic 项 | 在金属或不适用体系中解释电场响应 |
| `fildyn` | `ph.x/q2r.x/dynmat.x` | dynamical matrix 文件名或前缀是什么 | dyn 文件列表、q2r/dynmat 输入 | ph、q2r、dynmat 使用不一致的文件名 |
| `ldisp` | `ph.x` | 是否计算 uniform q-grid | `nq1/nq2/nq3`、dyn 文件数量 | 想要 dispersion 却只算 Gamma |
| `nq1/nq2/nq3` | `ph.x` | q-grid 尺寸与 IFC 插值范围 | 目标性质、q2r 读取数量、matdyn 插值 | q-grid 太粗却解释细小频率特征 |
| `recover` | `ph.x` | 中断后是否从已有 perturbation 续算 | scratch/restart 文件、output 续算信息 | scratch 丢失后以为能可靠续算 |
| `flfrc` | `q2r.x/matdyn.x` | real-space force constants 文件 | q2r output、matdyn input | `matdyn.x` 读取旧的或不匹配的 IFC |
| `zasr` | `q2r.x` | 在 IFC 层如何施加 acoustic sum rule | Gamma 声学支、ASR 前后对比 | 用 ASR 掩盖结构或收敛问题 |
| `asr` | `matdyn.x/dynmat.x` | 在频率/模式后处理中如何施加 ASR | 维度、极性、Gamma 附近声学支 | 不记录 ASR 设置就比较频率 |
| `q_in_band_form` | `matdyn.x` | q-path 是否按 band path 格式输入 | q 点段数、路径标签记录 | 路径格式和点数写错导致图像误读 |
| `flfrq` | `matdyn.x` | 频率输出写到哪里 | plot/data 文件、单位、q-path | 用错旧频率文件做结论 |

## output review

phonon output review 先确认程序链完整，再看频率本身。推荐把判断拆成 `ph.x`、`q2r.x`、`matdyn.x`、`dynmat.x` 四层。

`ph.x`：

- 是否明确读取了目标 `prefix/outdir` 的 SCF 数据。
- output 是否报告与 input 一致的 cutoff、k 点、smearing、对称性和 q 点。
- 对 Gamma 或每个 q 点，所有 irreducible representations / perturbations 是否达到 `tr2_ph`。
- `ldisp=.true.` 时，q 点列表和完成的 dynamical matrix 文件数量是否与 `nq1/nq2/nq3` 一致。
- 是否出现 electric field、metal/insulator、symmetry、convergence、recover 相关 warning。
- `fildyn` 输出是否是本次任务的 dyn 文件，而不是旧文件或其他分支文件。

`q2r.x`：

- 是否读取了完整 q-grid 的 dynamical matrices；缺一个 q 点或 dyn 文件都不能进入可靠插值。
- `fildyn` 是否与 `ph.x` 输出前缀一致，`flfrc` 是否写出新的 IFC 文件。
- `zasr` 设置是否记录；如果使用 ASR，应保留 ASR 前后或至少记录判断依据。

`matdyn.x`：

- 是否读取了本次 `q2r.x` 生成的 `flfrc`。
- `q_in_band_form`、q-path 点数和 `flfrq` 是否符合预期。
- Gamma 附近三条 acoustic branches 是否接近 0；若不接近，应回查结构残余力、SCF、cutoff、k 点、q-grid 和 ASR。
- negative frequency 的位置、大小、是否局域在 Gamma 附近、是否随参数变化稳定存在，都要记录。
- 插值出来的 dispersion 只代表当前 q-grid 和 ASR 设置下的结果；不能把粗 q-grid 的细节当作最终物理结论。

`dynmat.x`：

- 是否读取单个 Gamma dynamical matrix，且 `fildyn` 与 `ph.x` Gamma 输出一致。
- `asr` 设置是否记录，Gamma acoustic modes 是否经过审阅。
- 若 `epsil` 分支有效，是否能看到 dielectric tensor、Born effective charges 或 non-analytic correction 相关输出；若物理条件不适用，应标为不解释。

## 常见错误

| 现象 | 常见原因 | 优先判断 |
|---|---|---|
| `ph.x` perturbation 不收敛 | SCF 太松、mixing 不稳、smearing/k 点不合适、`tr2_ph` 过严或体系本身困难 | 先复查 SCF output，再看每个 perturbation 的残差变化 |
| Gamma 声学支明显不为 0 | 结构未充分平衡、cutoff/k 点不足、ASR 未处理或处理方式不合适 | 比较 ASR 前后，回查 relax final forces/stress |
| 出现小 negative frequency | 数值噪声、ASR/q-grid/插值误差、结构残余力 | 不直接判定不稳定；做控制变量复查 |
| 大范围或稳定 negative frequency | 可能是真实动力学不稳定，也可能是结构/参数严重错误 | 用更严格 SCF、cutoff、k 点和关键 q 点复算确认 |
| `q2r.x` 报 dyn 不全或读错 | `ph.x` q-grid 未完成、`fildyn` 前缀不一致、文件被覆盖 | 检查 dyn 文件列表和 q2r output |
| dispersion 图可画但不可信 | q-grid 太粗、IFC 文件来自旧任务、ASR 设置未记录 | 回查 `nq1/nq2/nq3`、`flfrc` 时间和 record |
| `dynmat.x` Gamma 结果与 q-grid 插值 Gamma 不一致 | ASR 层级不同、输入 dyn/IFC 不同、non-analytic 项处理不同 | 明确比较的是直接 Gamma 还是 q-grid 插值 |
| `epsil`/Born 分支缺失或异常 | `epsil` 未开、金属体系不适用、Gamma 条件或电场响应设置不满足 | 只在有效物理条件下解释 dielectric/Born 输出 |
| 用 ASR “修复”虚频 | ASR 只能约束平移不变性相关误差 | ASR 后结果仍需回查结构和数值收敛 |

negative frequency 的最低判断顺序：

1. 先确认单位和符号。QE/后处理输出中负频常表示 imaginary mode，不等于“频率略小一点”。
2. 再定位出现位置：Gamma 附近、小范围 q 点、整条分支、还是大范围 BZ。
3. 然后判断数值敏感性：收紧 SCF、cutoff、k 点、q-grid、smearing 或改变 ASR 后是否稳定存在。
4. 最后才写物理结论。没有参数复查时只能写 `WARN` 或 `BLOCK`，不能写最终稳定性判断。

## 最低掌握深度

阅读和执行 phonon workflow 前，最低需要掌握以下层级：

- 能画出两条数据链：`pw.x scf -> ph.x Gamma -> dynmat.x`，以及 `pw.x scf -> ph.x q-grid -> q2r.x -> matdyn.x`。
- 能解释为什么 phonon 比普通 bands/DOS 更依赖严格 SCF、充分 relax、收敛的 cutoff/k 点、合适 smearing 和足够 q-grid。
- 能在 input 中识别 `tr2_ph`、`fildyn`、`ldisp/nq1/nq2/nq3`、`flfrc`、`asr/zasr`、`q_in_band_form`、`flfrq` 的作用和文件依赖。
- 能读 `ph.x` output，确认每个 q 点和 perturbation 是否收敛，以及 dyn 文件是否完整。
- 能读 `q2r.x` output，确认 q-grid dynamical matrices 被完整转换为 IFC。
- 能读 `matdyn.x` output 或频率文件，判断 q-path、Gamma acoustic branches、negative frequency 和 ASR 设置是否可追踪。
- 能读 `dynmat.x` output，知道它主要服务 Gamma modes 和相关模式分析，不等同于 q-grid dispersion。
- 能把结果分成 `PASS / WARN / BLOCK`：没有完整 dyn/IFC/收敛证据时不进入下游；有小虚频但未复查时不写最终物理结论。

## 对应 workflow

- [workflows/phonon/gamma-phonon.md](../workflows/phonon/gamma-phonon.md)：单个 Gamma phonon、`ph.x -> dynmat.x`、acoustic modes、`epsil` 分支。
- [workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)：q-grid dispersion、`ph.x -> q2r.x -> matdyn.x`、q-path 插值和 negative frequency 判断。
- [workflows/phonon/phonon-dos.md](../workflows/phonon/phonon-dos.md)：基于 `matdyn.x` 的 phonon DOS 分支。
- [workflows/phonon/phonon-debugging.md](../workflows/phonon/phonon-debugging.md)：不收敛、ASR、虚频、q-grid 和上游 SCF/relax 质量排查。
- [physics-judgement/imaginary-frequency-triage.md](../physics-judgement/imaginary-frequency-triage.md)：虚频三分法和 ASR 边界。

## 资料来源

- QE INPUT_PH reference: <https://www.quantum-espresso.org/Doc/INPUT_PH.html>
- QE INPUT_Q2R reference: <https://www.quantum-espresso.org/Doc/INPUT_Q2R.html>
- QE INPUT_MATDYN reference: <https://www.quantum-espresso.org/Doc/INPUT_MATDYN.html>
- QE INPUT_DYNMAT reference: <https://www.quantum-espresso.org/Doc/INPUT_DYNMAT.html>
- QE PHonon user guide: <https://www.quantum-espresso.org/Doc/ph_user_guide/>
- Pranab Das phonon tutorial: <https://pranabdas.github.io/espresso/hands-on/phonon/>
- Kyoto phonon DokuWiki: <https://www2.yukawa.kyoto-u.ac.jp/~koudai.sugimoto/dokuwiki/doku.php?id=quantumespresso%3Aphonon%3A%E3%83%95%E3%82%A9%E3%83%8E%E3%83%B3%E3%81%AE%E8%A8%88%E7%AE%97>
