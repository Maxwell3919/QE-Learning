# DFT, Kohn-Sham and SCF

## QE 中对应的问题

在 QE 里，本页关注 `pw.x` 的 SCF 在做什么、哪些输入会改变自洽过程、以及 output 中哪些信号决定这个 ground state 能否被下游 workflow 使用；完整 DFT 或 Kohn-Sham 方程推导不在本页展开。

SCF（self-consistent field）计算在固定原子结构、赝势、平面波 cutoff、k 点和占据策略下，迭代得到自洽的电子密度、有效势、总能和 Kohn-Sham 本征值。对 QE workflow 来说，SCF 是一个数据边：

```text
structure + pseudopotential + cutoff + k-points + electronic settings
  -> pw.x calculation='scf'
  -> charge density / potential / total energy / eigenvalues / scratch state
  -> relax, bands, DOS, PDOS, phonon, pp.x
```

最低理解应落在下面几件事：

- `pw.x` 会从初始电荷密度和势开始，解 Kohn-Sham 单电子问题，得到新的电子密度，再把新旧密度混合，进入下一轮迭代。
- `conv_thr` 判断的是电子自洽误差是否足够小；它不是 cutoff、k 点或 smearing 的收敛性测试。
- total energy 只可在相同结构、赝势、cutoff、k 点、occupation 等设置下比较。
- Kohn-Sham eigenvalues 是 DFT 辅助体系的本征值，不能直接等同于实验准粒子能级。
- 下游程序通常通过一致的 `prefix/outdir` 读取 SCF 产生的 charge density、wavefunction、XML 等 scratch 数据。

## 输入字段表

| 字段 | 所属位置 | QE 中控制的问题 | output 中应如何核对 | 常见误用 |
|---|---|---|---|---|
| `calculation='scf'` | `&CONTROL` | 选择固定离子位置的电子自洽计算 | output header / setup 显示 calculation 类型 | 把 SCF 当成结构优化，或用错误 calculation 生成下游数据 |
| `prefix` | `&CONTROL` | 标记本次数据集，供后续程序查找 scratch | output 中 prefix 与下游 input 一致 | 多个任务共用 prefix，导致读取旧数据 |
| `outdir` | `&CONTROL` | 保存和读取 charge density、wavefunction、XML 等临时数据 | output 打印 temporary directory；下游使用同一位置 | scratch 被删、路径换了、或误读旧 outdir |
| `pseudo_dir` | `&CONTROL` | 指向赝势文件目录 | output 列出的 pseudo 文件与 input 一致 | 文件名存在但来源、泛函族或元素标签混乱 |
| `ecutwfc` | `&SYSTEM` | 波函数平面波 cutoff，单位 `Ry` | output 中 kinetic-energy cutoff 与 input 一致 | 只照抄教程值，不做目标性质收敛测试 |
| `ecutrho` | `&SYSTEM` | 电荷密度和势的 cutoff，单位 `Ry` | output 中 charge density cutoff 与 input 一致 | 对 USPP/PAW 或声子/应力任务设置过低 |
| `occupations` | `&SYSTEM` | 控制电子占据方式，如固定占据、smearing、tetrahedra 等 | output 中 occupation scheme、Fermi energy、band occupation 可解释 | 金属、小带隙、绝缘体策略混用 |
| `smearing` | `&SYSTEM` | 当使用 smearing 占据时选择展宽函数 | output 中 smearing 类型与预期一致 | 为了让 SCF 收敛而任意加 smearing，不记录物理后果 |
| `degauss` | `&SYSTEM` | smearing 宽度，单位 `Ry` | output 中展宽信息与 input 一致；Fermi 附近占据合理 | 过大改变能量和占据，过小导致金属难收敛 |
| `conv_thr` | `&ELECTRONS` | SCF 收敛阈值；QE 以 estimated energy error 与它比较 | 最终 `estimated scf accuracy` 达到目标阈值，且出现 convergence 信息 | 阈值过松却进入 phonon、force/stress 敏感任务 |
| `mixing_mode` | `&ELECTRONS` | 控制电荷密度混合算法 | output 的 SCF history 不应持续振荡 | 不理解默认混合方式，在困难体系中只改步数 |
| `mixing_beta` | `&ELECTRONS` | 控制新旧电荷密度混合强度 | total energy / estimated accuracy 是否单调或逐步稳定 | 金属、大胞、slab 中过大导致 charge sloshing |
| `mixing_ndim` | `&ELECTRONS` | 控制混合历史长度 | 长周期振荡时可作为诊断项记录 | 把它当成普适加速参数 |
| `electron_maxstep` | `&ELECTRONS` | 限制电子自洽最大迭代步数 | output 是否因达到最大步数而停止 | 迭代耗尽后仍把结果当收敛 |
| `startingpot` / `startingwfc` | `&ELECTRONS` | 控制初始势和波函数来源 | output 是否从 atomic / file / restart 读取 | restart 来源不清，污染新测试 |
| `K_POINTS automatic` | card | SCF 的 Brillouin zone 均匀采样 | output 中 irreducible k-points、weights 与预期一致 | 把 bands k-path 当作 SCF k 点 |

官方文档给出参数含义、默认值、单位和合法选项；本页的 PASS/WARN/BLOCK 解释是本仓库 workflow 层面的实践判断。

## output review

读 SCF output 时，先确认文件对应正确 input，再判断自洽和下游准入。`JOB DONE` 只说明程序结束，不说明 ground state 可信。

| Output 信号 | 需要判断什么 | PASS / WARN / BLOCK 用法 |
|---|---|---|
| Program header / QE version | 程序、版本、并行环境和运行日期是否可记录 | 缺少环境记录通常是 WARN；读错 output 是 BLOCK |
| Calculation setup | `calculation`、`prefix`、`outdir`、`pseudo_dir` 是否与本次任务一致 | 任一项对不上且无法解释，应 BLOCK |
| Pseudopotentials loaded | 元素标签、赝势文件、PP 类型、价电子数、泛函提示是否符合预期 | 来源不明、标签错配、泛函混用通常 BLOCK |
| Cutoff reported | `ecutwfc`、`ecutrho` 是否实际生效 | 与 input 不一致且无法解释，应 BLOCK；未做收敛性测试通常 WARN |
| K-points reported | SCF 是否使用合适的 uniform mesh、shift 和 weights | 误用 bands path 或 k 点明显不对，应 BLOCK |
| Occupation / smearing | 占据策略、Fermi energy、smearing 信息是否能解释 | 金属/绝缘体策略未确认通常 WARN；明显不匹配应 BLOCK |
| SCF iterations | total energy 和 `estimated scf accuracy` 是否趋于稳定 | 振荡、停滞或达到最大步数未收敛，应 BLOCK |
| Convergence message | 是否出现 `convergence has been achieved` 或等价自洽成功证据 | 没有明确 convergence 证据时不能 PASS |
| Final total energy | `! total energy` 是否记录，且只在相同设置下比较 | 可记录为本次状态能量；不能单独证明参数收敛 |
| Estimated SCF accuracy | 最终 accuracy 是否满足 `conv_thr` 和下游用途 | 达标是 SCF PASS 的必要条件；phonon/力/应力任务需更谨慎 |
| Warnings / restart | warning、symmetry 变化、restart、old scratch 是否解释清楚 | 未解释 warning 通常至少 WARN；scratch 污染应 BLOCK |

### SCF loop 的最小读法

一个可审阅的 SCF loop 至少应能回答：

1. 每一轮迭代的 total energy 是否逐步稳定，还是在两个或多个状态之间来回跳。
2. `estimated scf accuracy` 是否下降到 `conv_thr` 附近或以下。
3. 最后一轮是否由自洽成功结束，而不是由 `electron_maxstep`、walltime、I/O 或 diagonalization 问题结束。
4. 若不收敛，问题更像 charge mixing、occupation/smearing、结构/赝势、cutoff/k 点，还是 scratch/restart 混乱。

### charge density / potential 的 workflow 含义

SCF 中真正被反复更新的是电子密度及其对应的有效势。QE 用这些数据生成并保存 ground-state scratch；后续 `bands`、`dos`、`projwfc`、`ph.x`、`pp.x` 等程序依赖它们。因而：

- `prefix/outdir` 是数据边的一部分，不只是文件命名习惯。
- 清理 scratch、换机器、改 prefix、改 outdir 后，下游读取的数据边必须重新确认。
- 对比两个 total energy 前，应确认它们来自同一类输入设置；否则差异可能只是数值设置差异。

## 常见错误

| 错误 | 为什么危险 | 最低诊断动作 |
|---|---|---|
| 只看 `JOB DONE` | 程序完成不等于 SCF 收敛 | 搜索 convergence message、final accuracy、warning |
| 把 `conv_thr` 当成总能收敛测试 | `conv_thr` 是电子自洽阈值，不是 cutoff/k 点收敛证据 | 单独做 cutoff/k 点/smearing convergence loop |
| SCF 不收敛时只加 `electron_maxstep` | 只是允许失败持续更久，不解决振荡原因 | 先看 energy/accuracy 历史，再检查 `mixing_beta`、occupation、结构和赝势 |
| 金属或小带隙体系占据策略不清 | occupation/smearing 会影响 Fermi 附近占据和能量 | 在 output review 中记录 occupation、smearing、degauss、Fermi energy |
| 混用 `prefix/outdir` | 下游可能读取旧 charge density 或 wavefunction | 为新验证使用清晰 scratch，记录路径和 restart 状态 |
| 直接解释 Kohn-Sham eigenvalues | KS 本征值不是实验准粒子能级 | 只在 DFT workflow 语境下解释 bands/DOS，避免过度物理结论 |
| total energy 跨设置比较 | 不同赝势、cutoff、k 点或 smearing 下能量零点和误差不同 | 只比较同一计算协议内的能量差 |
| 把 SCF PASS 当作 phonon PASS | phonon 对结构、SCF、cutoff、k 点、smearing 更敏感 | 进入 DFPT 前要求更严格的收敛和结构证据 |

## 最低掌握深度

学到本页即可停止继续推导，但必须能完成这些判断：

- 能说清 `pw.x calculation='scf'` 的输入来自结构、赝势、cutoff、k 点、occupation 和电子迭代设置。
- 能解释 `&CONTROL` 中 `prefix/outdir/pseudo_dir` 为什么决定下游能否读到正确 ground state。
- 能解释 `&SYSTEM` 中 `ecutwfc/ecutrho/occupations/smearing/degauss` 分别影响基组、密度/势和占据。
- 能解释 `&ELECTRONS` 中 `conv_thr/mixing_beta/electron_maxstep` 对 SCF loop 的作用。
- 能从 output 找到 SCF iterations、`! total energy`、`estimated scf accuracy`、Fermi energy、occupation、warning。
- 能区分三件事：程序结束、电子 SCF 收敛、数值参数对目标性质收敛。
- 能给出 PASS / WARN / BLOCK 判断，并说明允许进入哪些下游 workflow。

不需要在本页掌握完整交换关联泛函理论、严格 KS 方程推导、线性响应理论或准粒子修正；这些不是第一个 QE SCF 闭环的准入条件。

## 对应 workflow

- [workflows/ground-state/scf.md](../workflows/ground-state/scf.md)：SCF 输入模板、output review 模板和 PASS/WARN/BLOCK 判断。
- [learn/02-first-scf-loop.md](../learn/02-first-scf-loop.md)：第一个可审阅 SCF 闭环的学习路线。
- [workflows/ground-state/relax.md](../workflows/ground-state/relax.md)：relax 中每个 ionic step 都依赖电子 SCF 稳定收敛。
- [workflows/electronic/bands.md](../workflows/electronic/bands.md)：bands 读取 SCF ground state，但沿 k-path 另做电子结构计算。
- [workflows/phonon/phonon-dispersion-dfpt.md](../workflows/phonon/phonon-dispersion-dfpt.md)：`ph.x` 依赖 SCF 数据，对前置结构和收敛证据要求更高。
- [physics-judgement/dft-ground-state-boundary.md](../physics-judgement/dft-ground-state-boundary.md)：ground-state DFT 可判定边界。
- [physics-judgement/kohn-sham-eigenvalue-boundary.md](../physics-judgement/kohn-sham-eigenvalue-boundary.md)：Kohn-Sham eigenvalue 的解释边界。

## 资料来源

- QE `pw.x` INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE documentation: <https://www.quantum-espresso.org/documentation/>
- QE `pw.x` input page checked during this edit: official page reports `pw.x / PWscf / Quantum ESPRESSO` input description, version 7.5.
- Pranab Das SCF tutorial: <https://pranabdas.github.io/espresso/hands-on/scf>
