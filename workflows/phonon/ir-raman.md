# IR and Raman workflow

## 本页解决什么问题

IR / Raman workflow 用于把 Gamma phonon modes 与电场响应、Born effective charge、dielectric tensor、Raman tensor 和对称性选择定则联系起来。本页给出 QE/PHonon 中 IR/Raman 相关结果的最低审阅框架，帮助判断输出是否足以支持活性、模式归属和谱峰边界说明，并把 harmonic DFPT 结果与有限温实验谱线重建区分开。

本页的核心边界是：harmonic DFPT 给出的是在指定结构和电子基态上的 zone-center response；有限温度、非谐性、样品取向、实验仪器函数、峰宽、强度标定和真实 spectrum reconstruction 都在本页之外。IR/Raman activity label 只有在 Gamma modes、response tensor、单位、简并、warning 和适用条件都被审阅后才可进入下游表述。

## 上游依赖

- Gamma phonon 已通过 [Gamma phonon workflow](gamma-phonon.md) 审阅。
- IR 相关解释需要可信 Born effective charge 和 dielectric tensor。
- Raman 相关解释需要当前 QE 版本、编译能力和 `INPUT_PH` 中对应分支支持。
- 结构对称性、mode degeneracy 和 mode mixing 已记录；若结构或对称性改变，应重新审阅模式编号。
- 若与实验谱比较，必须记录 harmonic approximation、温度、anharmonicity、broadening、LO-TO splitting 和 functional 边界。

## 计算图

```text
final static SCF
  -> ph.x at Gamma with response branch
  -> Gamma dynamical matrix + optional dielectric / Born / Raman tensors
  -> dynmat.x mode analysis and optional cross-section post-processing
  -> IR / Raman activity boundary review
```

`ph.x` 是 response 数据的生成阶段，读取上游 `pw.x` 的 `prefix/outdir` 并写出 `fildyn`。`dynmat.x` 是 Gamma dynamical matrix 的后处理阶段，可读取 dynamical matrix 中的 Born effective charge、dielectric tensor 或 Raman tensor 信息并输出模式、频率、eigenvectors 以及可用时的 IR/Raman 相关结果。`dynmat.x` 不能从一个只含频率的文件中凭空生成缺失的 response tensor。

## 关键 QE 输入对象

```fortran
&INPUTPH
  prefix = '<system>',
  outdir = '<scratch_dir>',
  tr2_ph = <phonon_threshold>,
  trans = .true.,
  epsil = <true_for_ir_related_branch>,
  lraman = <true_if_supported_and_needed>,
  fildyn = '<system>.dynG',
/
0.0 0.0 0.0
```

| 字段 | 程序 | 判断意义 | 联动对象 | output 中如何验证 |
|---|---|---|---|---|
| `epsil` | `ph.x` | 请求 dielectric/Born response，支撑 IR 与 LO-TO 相关解释 | 绝缘体条件、Gamma limit | dielectric tensor 和 Born tensors 输出 |
| `zeu` / `zue` | `ph.x` | Born effective charge 的计算路径 | `epsil`、`trans`、dielectric response | Born tensors 是否输出及算法边界 |
| `lraman` | `ph.x` | 请求 Raman 相关 response 分支 | QE 版本、编译和方法支持 | Raman tensor 或 activity 相关输出 |
| `trans` | `ph.x` | 计算 phonon perturbations | Gamma modes | irreducible representations 收敛 |
| `fildyn` | `ph.x` / `dynmat.x` | 保存 Gamma dynamical matrix | mode analysis | `dynmat.x` 读取当前文件 |
| `q` | `dynmat.x` | LO non-analytic direction when applicable | Born charges、dielectric tensor、LO-TO splitting | output 中 LO mode 方向 |
| `asr` | `dynmat.x` | 模式频率和 acoustic modes 后处理 | Gamma acoustic modes、effective charges | ASR 设置与 mode list |
| `filout` / `fileig` | `dynmat.x` | 频率、displacement、eigenvector 输出 | mode assignment、degeneracy review | 文件名、单位和列含义 |

## 命令与文件边界

```bash
ph.x -in ph.ir-raman.<system>.in > ph.ir-raman.<system>.out
dynmat.x -in dynmat.<system>.in > dynmat.<system>.out
```

`ph.x` 是否输出 IR/Raman 相关量取决于输入字段、体系条件和当前 QE 功能支持。`dynmat.x` 可帮助审阅 Gamma modes、频率、eigenvectors、LO-TO 方向和模式信息，并在文件中有相应 response tensor 时计算或整理 IR/Raman 相关输出；但不能凭频率表本身推出 IR/Raman activity。记录中应保留 mode 编号、频率、单位、张量/强度输出是否存在、mode degeneracy、activity labels、broadening 或绘图处理方式。

文件链审阅应明确三件事。第一，`ph.x` 的 `prefix/outdir` 指向当前 final static SCF，Gamma q vector 和 `fildyn` 都可追踪。第二，IR 分支需要 `epsil`、Born effective charge 和 dielectric tensor 的输出及适用条件；对金属或 q/=0 场景不能强行解释这些量。第三，Raman 分支需要确认 `lraman` 实际执行、输出格式和当前 QE 版本/编译支持；没有 Raman tensor 或 activity 输出时，不能只凭模式频率补写 Raman 活性。

## Output review

| 检查项 | 从哪里看 | 能证明什么 | 不能证明什么 |
|---|---|---|---|
| Gamma phonon 状态 | `ph.x` / `dynmat.x` mode list | 基础 modes 已计算 | 谱学强度已可信 |
| `epsil` / Born 输出 | `ph.x` response 段落 | IR 相关 response 数据存在 | 金属或边界体系可解释 |
| Raman 输出 | `ph.x` Raman tensor/activity 段落 | Raman 分支被执行 | 当前结果能直接复现实验谱 |
| mode symmetry / degeneracy | `dynmat.x`、symmetry 输出 | 模式归属和简并关系 | mode mixing 已完全解决 |
| units / sign convention | 频率和强度输出 | 数值可复查 | 与实验单位、温度条件一致 |
| warnings | `ph.x` / `dynmat.x` output | 功能、收敛或 response 异常 | warning 可以忽略 |

操作审阅建议按顺序完成。先复查 Gamma phonon 是否为 PASS，包括 perturbation convergence、acoustic modes、ASR 和 imaginary frequency triage。再查 `epsil` 分支是否输出 macroscopic dielectric tensor 和 Born effective charges，并记录张量单位、索引顺序、charge-neutrality / ASR 修正和 LO-TO 方向。接着查 `lraman` 分支是否真的输出 Raman tensor、activity 或 cross-section 相关信息，不能把输入中写了 `lraman=.true.` 当作执行成功。最后按 mode 编号逐项记录 frequency、activity label、degeneracy、eigenvector / symmetry assignment、warning 和用于画图的 broadening；任何一个环节缺失，都应在 PASS/WARN/BLOCK 中降级。

## 收敛与可靠性

- IR/Raman 解释继承 Gamma phonon、SCF、response tensor 和结构对称性的全部误差。
- Harmonic DFPT 频率和强度不自动等同有限温实验光谱；温度、anharmonicity、样品取向、broadening 和 selection rules 都是解释边界。
- 没有 response tensor 或 activity 输出时，不能只凭频率表判断模式 IR/Raman 活性。
- mode degeneracy 或 mode mixing 会影响峰归属，应结合 symmetry 和 eigenvectors 审阅。
- Born effective charges、dielectric tensor、Raman tensor、LO-TO splitting 和 activity labels 的可靠性应分别审阅；一个分支 PASS 不自动保证另一个分支 PASS。
- 绘图 broadening 只能生成可读谱线，不能证明 mode assignment、强度或 finite-temperature spectrum 可靠。

## PASS / WARN / BLOCK

| 状态 | 条件 | 是否允许进入下游 |
|---|---|---|
| PASS | Gamma phonon 为 PASS；所需 IR/Raman response 分支实际执行并收敛；Born charges、dielectric tensor、Raman tensor 或 activity 输出、mode symmetry、degeneracy、单位、LO-TO / ASR 边界和 warning 审阅已记录 | 可进入受限的谱峰归属、图件整理或与实验的边界化比较 |
| WARN | Gamma modes 可信但 response tensor、symmetry、degeneracy、LO-TO 方向、单位转换、version support 或 broadening 仍需复查；只适合 harmonic / qualitative 对照 | 可用于学习和诊断，不应给强定量强度或实验对照结论 |
| BLOCK | Gamma phonon 为 BLOCK；response 分支未输出或未收敛；在不适用体系中解释 IR/Born；缺少 Raman tensor 却标注 Raman activity；只凭频率表推断活性；用 broadening 制造实验谱外观 | 不允许写入 IR/Raman 活性、强度或谱学结论 |

## 常见误区

- 把 harmonic DFPT peak position 当作实验光谱峰位。
- 没有响应张量输出却写 IR/Raman activity。
- 不记录 mode degeneracy、mode mixing 和 symmetry 边界。
- 不检查 Born effective charges、dielectric tensor、Raman tensor、单位和 activity label 是否实际出现在 output。
- 忽略 LO-TO splitting、Born charge 或 dielectric tensor 的适用条件。
- 用绘图 broadening 掩盖模式归属不清。

## 下游影响

本页依赖 [Gamma phonon workflow](gamma-phonon.md) 和 [dielectric tensor and Born effective charge workflow](dielectric-born-effective-charge.md)。若目标是完整光谱、温度效应或强非谐性，需要转入更高级的 spectroscopy 或 finite-temperature 方法；本页只提供 QE/PHonon output review 和边界框架。下游报告应把 harmonic frequency、activity label、response tensor、broadening 和实验比较分开写，避免把一个后处理图直接表述为有限温实验 spectrum。

## 来源与边界

- Stable: `ph.x` 中 `epsil`、`zeu/zue`、`lraman`、`trans`、`fildyn` 和 response 输出边界以 QE `INPUT_PH` 和 PHonon guide 为准；`dynmat.x` 的 `fildyn`、`q`、`asr`、`filout` 和 IR/Raman post-processing 边界以 QE `INPUT_DYNMAT` 为准。
- Version-sensitive: Raman 支持、输出格式、张量语义和编译条件需核对当前 QE 版本。
- Boundary: harmonic DFPT 与实验有限温 spectrum 之间需要额外近似边界说明。

## 资料来源

- QE INPUT_PH reference: <https://www.quantum-espresso.org/Doc/INPUT_PH.html>
- QE INPUT_DYNMAT reference: <https://www.quantum-espresso.org/Doc/INPUT_DYNMAT.html>
- QE PHonon user guide: <https://www.quantum-espresso.org/Doc/ph_user_guide/>
- 本仓库：[theory-minimum/dfpt-phonons.md](../../theory-minimum/dfpt-phonons.md)、[theory-minimum/dielectric-born-charge.md](../../theory-minimum/dielectric-born-charge.md)
