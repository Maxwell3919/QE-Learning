# DFT, Kohn-Sham and SCF

## 本页解决什么问题

本页解释 `pw.x calculation='scf'` 在 QE workflow 中到底建立了什么：一个在给定结构、赝势、泛函、cutoff、k mesh 和 occupation 设置下自洽的 ground-state charge density。它帮助判断 SCF output 是否足以进入 relax、NSCF、bands、DOS、PDOS、`pp.x` 或 `ph.x`。本页不推导完整 DFT，只保留读 input/output 和做下游准入所需的最低概念。

## 最低掌握深度

科研用户最低需要理解到以下程度：

- DFT 是 ground-state density theory；QE 的普通 SCF 主要建立 ground-state density、potential、total energy 和 Kohn-Sham auxiliary eigenvalues。
- Kohn-Sham system 是辅助非相互作用体系；Kohn-Sham eigenvalue 可用于 bands/DOS workflow 的内部解释，但不能直接当作实验激发能。
- SCF convergence 说明电子自洽迭代满足停止条件；它不等于 cutoff/k mesh/smearing 对目标 observable 已收敛。
- `JOB DONE` 只说明程序结束；是否允许下游取决于 output evidence 和 PASS / WARN / BLOCK。

## QE 中的对应对象

| 对象 | QE 位置 | 用途 | output 证据 |
|---|---|---|---|
| `pw.x` | 程序 | 求解 ground-state electronic problem | program header、QE version、calculation type |
| `calculation='scf'` | `&CONTROL` | 固定结构下做电子自洽 | input echo、setup 段落 |
| `input_dft` | `&SYSTEM` | 选择或覆盖 exchange-correlation functional | XC functional 回显、pseudopotential warning |
| `conv_thr` | `&ELECTRONS` | SCF estimated energy error 阈值 | `estimated scf accuracy`、convergence message |
| `mixing_beta` / `mixing_mode` | `&ELECTRONS` | 电荷密度混合，影响 SCF 稳定性 | SCF iteration history、振荡或停滞 |
| `occupations` / `smearing` / `degauss` | `&SYSTEM` | 占据和 Fermi-level 附近积分处理 | occupation scheme、Fermi energy、band occupation |
| `prefix/outdir` | `&CONTROL` | 下游读取 SCF scratch 数据的数据边 | temporary directory、data-file 读取记录 |

相关 workflow：[SCF](../workflows/ground-state/scf.md)、[NSCF](../workflows/ground-state/nscf.md)、[bands](../workflows/electronic/bands.md)、[phonon DFPT](../workflows/phonon/phonon-dispersion-dfpt.md)。相关标准：[output review checklist](../standards/output-review-checklist.md)。

## 核心概念

SCF loop 的核心是密度自洽。QE 从初始 charge density / potential 出发，求解 Kohn-Sham 方程，得到新的电子密度，再通过 mixing 生成下一轮输入。迭代直到 estimated error 满足 `conv_thr` 或达到其他停止条件。

```text
structure + pseudopotential + XC + cutoff + k mesh + occupations
  -> initial density / potential
  -> Kohn-Sham solve
  -> new density
  -> mixing
  -> convergence check
  -> ground-state scratch data
```

Exchange-correlation functional 是物理近似来源；cutoff、k mesh、smearing 和 SCF threshold 是数值控制来源；`prefix/outdir` 是 workflow 数据依赖来源。这三类问题必须分开审阅。

## 对 input 的影响

- `input_dft` 或赝势自带的 functional 信息决定本次 SCF 的模型边界；改变 functional 等于改变 Hamiltonian，不是普通收敛参数调整。
- `conv_thr` 只控制电子自洽停止条件；进入 phonon、force/stress 或高精度能量差时，需要结合 workflow 重新审阅是否足够。
- `mixing_beta`、`mixing_mode`、`electron_maxstep` 是 SCF 稳定性控制项；增加最大步数不能修复错误 occupation、赝势、结构或 k mesh。
- `occupations/smearing/degauss` 会改变 Fermi 附近占据和能量表达；金属、绝缘体和小带隙体系不能混用解释。
- 下游读取 SCF 数据时，`prefix/outdir` 必须保持一致，且 scratch 来源应可追踪。

## 对 output review 的影响

SCF output 至少要读以下证据：

| output 证据 | 支持的判断 | 不能证明什么 |
|---|---|---|
| program header / QE version | output 属于目标程序和版本 | 结果科学可信 |
| pseudopotential loaded | 元素、UPF、价电子和 PP 类型被读取 | PP transferability 已验证 |
| cutoff / k-points reported | input 设置实际生效 | 目标 observable 已收敛 |
| SCF iterations | 自洽过程是否稳定 | cutoff/k mesh 已收敛 |
| `estimated scf accuracy` | 是否满足 `conv_thr` | 下游 phonon/bands/DOS 全部可用 |
| `! total energy` | 本次设置下的 total energy | 跨设置可直接比较 |
| Fermi energy / occupations | 占据策略和金属性边界 | Fermi level 具备绝对能量意义 |
| warnings / restart | 是否有未解释异常或旧 scratch 风险 | warning 可忽略 |

## 常见误区

- 把 Kohn-Sham eigenvalue 直接当作实验激发能或 quasiparticle energy。
- 把 `JOB DONE` 当作 SCF PASS。
- 把 total energy 平滑当作所有物理量收敛。
- SCF 不收敛时只增加 `electron_maxstep`。
- 金属、小带隙和绝缘体的 occupation 策略不区分。
- 换 `input_dft`、赝势或 structure 后沿用旧 scratch。

## PASS / WARN / BLOCK 关联

| 状态 | 理论依据 | 下游准入 |
|---|---|---|
| PASS | SCF 自洽收敛；input/output 一致；模型、数值参数和文件链已记录；没有未解释 warning | 可按 workflow 进入 relax、NSCF、bands、DOS、`pp.x` 或 DFPT 前置 |
| WARN | SCF 收敛但 cutoff/k mesh/smearing、functional 或 warning 边界尚未充分说明 | 可进入探索或敏感性测试，不应给最终结论 |
| BLOCK | SCF 未收敛；`prefix/outdir` 错配；PP/XC/structure 不一致；warnings 影响关键数据 | 不允许进入依赖 ground-state density 的下游 |

## 下游影响

- `relax/vc-relax`：每个 ionic/cell step 都依赖电子 SCF 质量。
- `NSCF/bands/DOS/PDOS`：读取或建立在同一个 ground-state density 上。
- `charge density / potential / ELF`：可视化数据来自 SCF density 或 potential。
- `phonon / DFPT`：对 SCF、structure、cutoff、k mesh 和 smearing 更敏感。
- `work function`：Fermi energy、electrostatic potential 和同一数据链必须一致。

## 与 physics-judgement 的边界

本页只解释普通 DFT/KS/SCF 的最低使用边界。以下问题应转到 `physics-judgement/`：

- [DFT ground-state boundary](../physics-judgement/dft-ground-state-boundary.md)
- [Kohn-Sham eigenvalue boundary](../physics-judgement/kohn-sham-eigenvalue-boundary.md)
- [ground-state vs excited-state](../physics-judgement/ground-state-vs-excited-state.md)
- [uncertainty statement](../physics-judgement/uncertainty-statement-template.md)

## 来源与边界

- Stable: `pw.x` 字段含义以 QE `INPUT_PW` 为准。
- Stable: ground-state density 与 Kohn-Sham auxiliary system 的理论边界来自 Hohenberg-Kohn 和 Kohn-Sham canonical literature。
- Boundary: 本页不判断某个 functional 是否适合特定材料；functional choice 进入 `physics-judgement/`。

## 资料来源

- QE INPUT_PW reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE documentation: <https://www.quantum-espresso.org/documentation/>
- Hohenberg and Kohn, Inhomogeneous Electron Gas.
- Kohn and Sham, Self-Consistent Equations Including Exchange and Correlation Effects.
- Kohn Nobel Lecture.
- 本仓库：[SCF workflow](../workflows/ground-state/scf.md)、[output review checklist](../standards/output-review-checklist.md)
