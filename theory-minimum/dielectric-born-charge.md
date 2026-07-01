# Dielectric tensor and Born effective charge

## QE 中对应的问题

介电张量和 Born effective charge 是 `ph.x` 在 Gamma limit 处理的电场线性响应量。它们主要回答三个 QE workflow 问题：

- `epsil=.true.` 是否物理有效，output 中是否真的产生 dielectric tensor 和 Born effective charge。
- Gamma 附近极性声子是否需要 non-analytic correction，以及 LO-TO splitting 能否被解释。
- `dynmat.x` 或 `matdyn.x` 后处理中使用非解析项时，输入张量、方向、ASR 和金属/绝缘体边界是否被记录清楚。

最低概念是：在绝缘体中，宏观电场微扰可以定义电子介电张量；原子位移引起的宏观极化变化给出 Born effective charge。长波极限 `q -> 0` 下，极性材料的库仑长程相互作用会让 dynamical matrix 出现方向相关的 non-analytic contribution，因此 Gamma 附近的 LO 和 TO 模式可能分裂。金属中自由载流子会屏蔽静态宏观电场，通常不能把 `epsil` 产生的电场响应当作绝缘体介电/Born 张量来解释。

本页只说明 QE input/output review 需要的最低判断，不写完整线性响应理论推导，也不保存具体材料案例。

## 输入字段表

| 字段 | 所属程序 | 用来回答的问题 | 需要联动检查 | 常见误用 |
|---|---|---|---|---|
| `epsil` | `ph.x` | 是否请求宏观介电张量和 Born effective charge | q 点是否为 Gamma、体系是否按绝缘体处理、`trans`、output warning | 在金属、未确认带隙或非 Gamma 分支中硬解释张量 |
| `trans` | `ph.x` | 是否计算原子位移 phonon perturbations | `epsil`、Gamma dynamical matrix、perturbation 收敛 | 只盯电场响应，忘记声子扰动也要收敛 |
| `fildyn` | `ph.x/dynmat.x/q2r.x` | 本次 Gamma dynamical matrix 或 q-grid dyn 前缀是什么 | 后处理读入文件、文件时间、是否来自同一 `prefix/outdir` | `dynmat.x` 或 `q2r.x` 读取旧 dyn 文件 |
| `tr2_ph` | `ph.x` | 电场和声子响应方程收敛到什么阈值 | 每个 perturbation 输出、SCF `conv_thr`、k 点和 cutoff | 只看 job 正常结束，不看响应收敛 |
| `prefix/outdir` | `pw.x/ph.x` | `ph.x` 是否读取同一个 SCF 基态 | SCF output、scratch/restart、赝势和结构版本 | SCF 和 PH 不是同一数据源却继续后处理 |
| `occupations/smearing/degauss` | `pw.x` | 体系是否被作为金属、绝缘体或边界小带隙问题处理 | band occupation、Fermi energy、DOS/bands、SCF 稳定性 | 有 smearing 或部分占据时仍按普通绝缘体解释 `epsil` |
| `K_POINTS` | `pw.x` | 电子响应采样是否足够 | 绝缘体/小带隙判断、SCF 收敛、Born charge 稳定性 | k 点太粗导致张量数值不稳定 |
| `ecutwfc/ecutrho` | `pw.x` | 力常数和响应量是否有足够基组质量 | cutoff 收敛、力/应力、phonon 频率 | 只按总能收敛就解释 Born charge |
| `asr` | `dynmat.x/matdyn.x` | 后处理频率和模式时如何施加 acoustic sum rule | Gamma acoustic modes、LO-TO 设置、ASR 前后记录 | 把 ASR 当作未收敛结构或错误张量的修复 |
| non-analytic / LO-TO 相关选项 | `dynmat.x/matdyn.x` | 是否在 Gamma limit 使用介电张量和 Born charges 加入非解析项 | 方向、张量是否完整、ASR、绝缘体条件 | 不记录方向却比较 LO-TO splitting |
| `zasr` | `q2r.x` | 在 IFC 转换阶段如何处理 Born effective charge 的 sum rule | q-grid dyn 文件、`flfrc`、后续 `matdyn.x` 设置 | q-grid 和 Gamma 后处理的 ASR 层级混淆 |

`epsil` 的关键边界：

- QE 官方 `ph.x` 输入中，`epsil` 只对 `q=0` 的非金属有效；它请求介电常数以及 Born effective charge。
- `trans=.true.` 且 `epsil=.true.` 时，Gamma phonon 与电场相关响应需要一起审阅。
- 当 `ldisp=.true.` 时，`epsil` 相关量通常只和 Gamma 分支相关，不能把它当成整条 q-grid dispersion 都已自动验证。
- 若上游 `pw.x` 使用金属占据、明显部分占据、无可靠带隙证据或边界小带隙行为，应先标记为 `WARN` 或 `BLOCK`，不要直接解释 dielectric tensor / Born effective charge。

## output review

`ph.x` output：

- 是否明确为 `q = (0, 0, 0)` 或 Gamma limit 响应；非 Gamma q 点不能期待同样的宏观电场张量输出。
- input summary 是否读到了 `epsil=.true.`、`trans=.true.`、目标 `prefix/outdir`、cutoff、k 点和 occupation policy。
- 是否出现 dielectric tensor 段落；张量是否为完整 3x3 信息，是否有对称性约化或警告。
- 是否出现 Born effective charge 段落；每个原子是否都有 3x3 有效电荷张量。
- electric-field perturbations、atomic displacement perturbations 是否都达到 `tr2_ph`。
- output 是否有 metallic、Fermi surface、electric field、polarization、symmetry、not converged、recover 相关 warning。
- `fildyn` 是否写出本次 Gamma dynamical matrix，且文件名能被后续 `dynmat.x` 或 `matdyn.x` 追踪。

`dynmat.x` / `matdyn.x` output：

- 后处理是否读取了同一次 `ph.x` 产生的 Gamma dyn 文件或同一套 IFC。
- 若使用 LO-TO splitting 或 non-analytic correction，是否记录了方向；Gamma limit 的非解析项是方向相关的，不能只写“Gamma 频率”。
- ASR 设置是否明确；比较频率时应说明 ASR 前后或使用的 `asr/zasr`。
- LO/TO 模式解释是否只建立在张量完整、响应收敛、绝缘体条件成立的前提上。
- 若没有 dielectric/Born 段落，后处理仍可给普通 Gamma phonon，但不能写 IR-active、LO-TO 或 non-analytic 结论。

PASS / WARN / BLOCK 最低判断：

| 判断 | 条件 | 允许下游 |
|---|---|---|
| `PASS` | 绝缘体条件明确；Gamma `epsil` 分支开启；电场和声子 perturbations 收敛；dielectric tensor 与 Born charges 完整；后处理方向和 ASR 记录清楚 | LO-TO splitting、non-analytic correction、IR 相关 review |
| `WARN` | 小带隙/占据边界、k 点或 cutoff 尚未充分复查；张量完整但数值敏感性未知；ASR/方向记录不充分 | 只允许探索性解释，不能写最终物理结论 |
| `BLOCK` | 金属或明显部分占据；非 Gamma 分支期待 `epsil`；电场响应未收敛；缺 dielectric/Born 输出；后处理读取错误文件 | 不进入 LO-TO / non-analytic / IR 结论 |

## 常见错误

| 现象 | 常见原因 | 优先判断 |
|---|---|---|
| output 没有 dielectric tensor | `epsil` 未开启、不是 Gamma、体系被识别为金属或电场响应条件不满足 | 查 `ph.x` input summary、q 点和 warning |
| Born effective charge 缺失 | `epsil` 分支没有成功完成、`trans/epsil` 组合或响应未收敛 | 查 electric-field 与 atomic perturbation 收敛 |
| 张量存在但数值看起来异常 | SCF 太松、k 点/cutoff 不足、结构未充分 relax、赝势或对称性问题 | 回到 SCF/relax/phonon convergence，不先解释物理 |
| 把 Born effective charge 当作静态价态 | 概念混淆 | Born charge 是极化对原子位移的动态响应，不是 Bader 电荷或形式价态 |
| LO-TO splitting 解释混乱 | 没有记录方向、non-analytic 设置、ASR 或 dielectric/Born 来源 | 补全后处理记录；缺记录时标 `WARN` |
| 金属里强行解释 `epsil` | 忽略自由载流子屏蔽和 QE 适用边界 | 金属或明显部分占据下标 `BLOCK` |
| q-grid dispersion 中误以为每个 q 点都有 `epsil` | 混淆 Gamma response 与 q-grid phonon | `epsil` 是 Gamma limit 相关分支，不替代 q-grid 收敛 |
| dynmat/matdyn 读错文件 | `fildyn/flfrc` 来自旧任务或不同 `prefix/outdir` | 核对文件名、时间、record 和 output header |

## 最低掌握深度

完成本页后，学习者至少应能做到：

- 说明 `epsil=.true.` 属于 `ph.x` 的 Gamma、非金属电场响应分支，不是普通 phonon dispersion 的通用开关。
- 区分 dielectric tensor、Born effective charge、LO-TO splitting 和 non-analytic correction：前两者是响应张量，后两者是 Gamma 附近极性声子后处理/解释中用到的效应。
- 解释 Born effective charge 是原子位移与宏观极化之间的动态响应量，不能当作静态价态解释。
- 在 input 中识别 `epsil`、`trans`、`fildyn`、`tr2_ph`、occupation policy、`asr/zasr` 和后处理非解析项之间的依赖。
- 在 output 中找到 dielectric tensor、Born effective charge、electric-field perturbation 收敛和 warning。
- 对金属、小带隙、smearing 或部分占据体系给出 `WARN/BLOCK`，而不是硬套绝缘体 LO-TO 解释。
- 记录 LO-TO / non-analytic correction 的方向、ASR 和张量来源；缺一项时不写最终结论。

## 对应 workflow

- [Dielectric tensor and Born effective charge workflow](../workflows/phonon/dielectric-born-effective-charge.md)：`epsil=.true.`、Gamma dielectric/Born 输出、LO-TO 和 non-analytic 前提。
- [Gamma phonon workflow](../workflows/phonon/gamma-phonon.md)：单个 Gamma phonon、`ph.x -> dynmat.x`、acoustic modes 和 `epsil` 分支。
- [Phonon dispersion DFPT workflow](../workflows/phonon/phonon-dispersion-dfpt.md)：q-grid dispersion 中使用 non-analytic correction 前的依赖和记录边界。
- [IR/Raman workflow](../workflows/phonon/ir-raman.md)：IR-active 模式、Born charge 和介电响应的下游使用。
- [PASS / WARN / BLOCK](../standards/pass-warn-block.md)：把金属边界、张量缺失、收敛不足和文件依赖错误转成准入判断。

## 资料来源

- QE INPUT_PH reference: <https://www.quantum-espresso.org/Doc/INPUT_PH.html>
- QE INPUT_DYNMAT reference: <https://www.quantum-espresso.org/Doc/INPUT_DYNMAT.html>
- QE INPUT_Q2R reference: <https://www.quantum-espresso.org/Doc/INPUT_Q2R.html>
- QE INPUT_MATDYN reference: <https://www.quantum-espresso.org/Doc/INPUT_MATDYN.html>
- QE PHonon user guide: <https://www.quantum-espresso.org/Doc/ph_user_guide/>
- Kyoto phonon DokuWiki: <https://www2.yukawa.kyoto-u.ac.jp/~koudai.sugimoto/dokuwiki/doku.php?id=quantumespresso%3Aphonon%3A%E3%83%95%E3%82%A9%E3%83%8E%E3%83%B3%E3%81%AE%E8%A8%88%E7%AE%97>
- Pranab Das phonon tutorial: <https://pranabdas.github.io/espresso/hands-on/phonon/>
