# SCF workflow

## 页面定位

- 对应学习路线：[learn/02-first-scf-loop.md](../../learn/02-first-scf-loop.md)
- 结构输入边界：本页只说明 QE 对结构输入的要求；结构构建与标准化在独立项目中展开。
- 规范入口：[standards/calculation-record-template.md](../../standards/calculation-record-template.md)、[standards/pass-warn-block.md](../../standards/pass-warn-block.md)

## 计算目标

SCF（self-consistent field）计算在固定原子结构下求解 Kohn-Sham 方程的自洽基态电子密度、总能、势、特征值和后续 workflow 所需的 scratch 数据。

SCF 是 QE workflow 的基础数据边。relax、bands、DOS、PDOS、phonon、charge density、potential 等步骤都依赖一个物理合理且数值收敛的 ground state。

## 输入前提

- 已确认 `<structure>` 的晶胞、元素、原子坐标、维度和周期性边界。
- 已选择同一泛函族、来源明确、适合目标价态的 `<pseudo>`。
- 已确定初始 `<ecutwfc>`、`<ecutrho>`、`<k_mesh>` 和 occupation/smearing 策略。
- 已明确本次 SCF 的下游用途：电子结构、结构优化、声子或后处理。

## 计算图

```text
<structure> + <pseudo> + <numerical_policy>
  -> pw.x scf
  -> converged charge density / total energy / eigenvalues / scratch state
  -> downstream workflow
```

## 需要的 QE 程序

- `pw.x`：执行 `calculation='scf'`，生成 ground-state charge density、eigenvalues 和后续程序需要读取的 scratch 数据。
- 下游程序通过一致的 `prefix/outdir` 读取该 SCF 状态。

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
  smearing = '<smearing_type>',
  degauss = <smearing_width>,
/
&ELECTRONS
  conv_thr = <scf_threshold>,
  mixing_beta = <mixing_beta>,
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

## 输入字段说明

| 字段 | 作用 | 常见风险 | 输出中如何验证 |
|---|---|---|---|
| `calculation='scf'` | 固定离子位置求自洽电子基态 | 把 SCF 当结构优化 | output 开头显示 calculation 类型 |
| `prefix` | 连接同一 workflow 的 scratch 数据 | 不同任务混用同一 prefix | 下游程序使用同一 prefix |
| `outdir` | 保存 charge density、wavefunction 等中间状态 | 删除或混用旧 scratch | output 打印 temporary directory |
| `pseudo_dir` | 指向赝势目录 | 文件存在但来源混乱 | output 列出 pseudo 文件与类型 |
| `ecutwfc` | 波函数平面波 cutoff | 直接照抄而不做收敛测试 | output 中有 kinetic-energy cutoff |
| `ecutrho` | 电荷密度/势 cutoff | 对 USPP/PAW 设置过低 | output 中有 charge density cutoff |
| `occupations/smearing/degauss` | 控制占据方式 | 金属/绝缘体策略混用 | output 中有 occupation scheme 和 Fermi energy |
| `conv_thr` | 电子自洽阈值 | 过松导致下游性质不可信 | SCF iteration 的 estimated accuracy |
| `mixing_beta` | 电荷混合强度 | 金属或大胞中设太大导致振荡 | SCF 是否来回震荡 |
| `K_POINTS` | Brillouin zone 采样 | 用 bands path 做 SCF | output 中有 irreducible k-points |

## 通用输出审阅模板

```markdown
## Output Review

- QE 程序:
- 计算类型:
- QE version:
- Structure summary:
- Pseudopotentials loaded:
- Cutoff reported:
- K-points reported:
- Occupation / smearing:
- SCF convergence:
- SCF iterations:
- Final total energy:
- Estimated SCF accuracy:
- Fermi energy:
- Forces:
- Stress:
- Warnings:
- Restart / scratch status:
- PASS / WARN / BLOCK:
- Reason:
- Allowed downstream workflows:
```

## 输出判断标准

- `JOB DONE` 只说明程序结束；必须同时检查 SCF 是否达到 `convergence has been achieved`。
- `estimated scf accuracy` 应与 `conv_thr` 和下游用途匹配。
- output 中报告的 cutoff、k 点、赝势、occupation 应与 input 预期一致。
- Fermi energy、占据数和 smearing 信息应与体系的金属/绝缘体判断一致。
- warning、symmetry 改写、charge 不收敛、旧 scratch 读取等信息需要进入记录。

## 收敛性要求

SCF 自洽收敛不等于参数收敛。至少需要独立检查：

- `<ecutwfc>` 对总能、力、应力或目标性质的影响。
- `<ecutrho>` 对 USPP/PAW、应力和声子相关任务的影响。
- `<k_mesh>` 对总能、Fermi 附近态和下游性质的影响。
- smearing/occupation 对金属或小带隙体系的影响。

phonon 属于 DFPT response workflow，对 SCF、结构优化和收敛性要求高于普通电子结构后处理。

## 常见错误与诊断

| 现象 | 可能原因 | 优先排查 |
|---|---|---|
| SCF 不收敛 | `mixing_beta` 太大、smearing 不合适、结构质量差、k 点/cutoff 问题 | 先看 energy/accuracy 是否振荡，再调 mixing 和 smearing |
| 能量看似收敛但力很差 | cutoff/k 点不足，结构远离平衡 | 做结构优化或收敛性测试 |
| 下游程序找不到数据 | `prefix/outdir` 不一致或 scratch 被删 | 保持同一 prefix/outdir，记录路径 |
| 结果与外部教程数值不同 | 赝势、结构、k 点、QE 版本不同 | 比较输入来源和输出摘要，不只比较单个数值 |

## 通用学习模板

学习时应选择一个结构简单、赝势来源明确、物理状态清楚的体系，并完整记录结构、赝势、cutoff、k 点、运行命令和 output 判断。具体材料体系不写入本仓库主文档。

个人学习记录中应保留 input、output、运行命令和 output review；本页提供的是阅读任何 SCF input/output 的通用检查框架。

## 记录模板

```text
pw.scf.<system>.in
pw.scf.<system>.out
record.md
pseudo-source.md
```

`record.md` 应写清楚结构来源、赝势来源、命令、环境、收敛状态、PASS / WARN / BLOCK 判断和允许进入的下游 workflow。

## 与其他 workflow 的关系

- `relax` / `vc-relax`：离子优化内部会反复调用电子 SCF。
- `bands`：需要先有 SCF ground state，再沿 k-path 做 bands 计算。
- `DOS/PDOS`：通常需要 SCF 后接 dense k-mesh NSCF。
- `phonon`：`ph.x` 直接依赖 SCF 产生的数据，且对 SCF 质量更敏感。

## 后续完善重点

- 逐项解释 output 中 total energy、SCF accuracy、Fermi energy、cutoff、k-points、pseudopotential 段落。
- 补充 SCF 不收敛、旧 scratch 误读、occupation 设置错误的通用诊断笔记。

## 资料来源

- QE `pw.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE documentation: <https://www.quantum-espresso.org/documentation/>
- Pranab Das SCF tutorial: <https://pranabdas.github.io/espresso/hands-on/scf>
