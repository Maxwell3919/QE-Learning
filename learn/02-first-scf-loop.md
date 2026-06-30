# First SCF Loop

## 能力目标

建立第一个可信 SCF 闭环，理解 `pw.x` 如何从结构、赝势、cutoff 和 k 点得到自洽基态。

本页不提供具体材料案例，也不提供真实 input/output 文件。所有名称、路径、数值和元素都使用通用占位符。实际学习记录应放在个人 calculation record 中。

## 完成判据

- 能解释 `&CONTROL`、`&SYSTEM`、`&ELECTRONS`。
- 能解释 `ATOMIC_SPECIES`、`ATOMIC_POSITIONS`、`K_POINTS`、`CELL_PARAMETERS`。
- 能在 output 中找到 total energy、SCF iteration、estimated accuracy、Fermi energy、cutoff、k-points、pseudopotential 信息。
- 能判断结果是 PASS、WARN 还是 BLOCK。
- 能说明该 SCF 是否允许进入 convergence、relax、bands/DOS/phonon 等下游 workflow。

## 第一个 SCF loop 在做什么

SCF（self-consistent field）是在固定原子结构下求自洽电子基态。对初学者来说，第一个可信 SCF loop 不追求发表级结果，而是建立一个闭环：

```text
structure + pseudopotential + cutoff + k-points + electronic settings
  -> pw.x calculation='scf'
  -> output review
  -> PASS / WARN / BLOCK
  -> downstream decision
```

这个闭环的关键是把 input、output 和下游准入条件连起来。`JOB DONE` 只说明程序结束，不说明 SCF 可信、参数收敛或结果可用于后续任务。

## 第一个 SCF input 的组成

一个最小但可审阅的 `pw.x` SCF input 通常由三个 namelist 和四类 cards 组成：

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

上面只是通用骨架。实际 input 必须根据结构维度、赝势建议、目标性质和下游 workflow 调整。

## Namelist 和 cards 各自负责什么

### `&CONTROL`

`&CONTROL` 负责运行层面的控制和文件路径。

- `calculation='scf'` 指定本次是固定离子位置的自洽电子计算。
- `prefix` 是同一 workflow 里下游程序识别 scratch 数据的名字。
- `outdir` 是 charge density、wavefunction、XML 等中间数据的位置。
- `pseudo_dir` 是 QE 读取赝势文件的位置。

审阅重点：output 中打印的 calculation 类型、temporary directory、prefix/pseudo 路径应与 input 和运行命令一致。`prefix/outdir` 混用旧数据会直接破坏下游可信度。

### `&SYSTEM`

`&SYSTEM` 负责物理体系和数值基组。

- `ibrav`、`CELL_PARAMETERS`、`nat`、`ntyp` 描述晶胞、原子数和元素种类。
- `ecutwfc` 控制波函数平面波 cutoff。
- `ecutrho` 控制电荷密度和势的 cutoff。
- `occupations`、`smearing`、`degauss` 描述占据方式，尤其影响金属、小带隙或半金属体系。

审阅重点：output 中 reported cutoff、cell/volume、number of atoms/species、occupation scheme、Fermi energy 或 occupation 信息应与预期一致。SCF 自洽收敛不等于 `ecutwfc`、`ecutrho`、k 点和 smearing 已经收敛。

### `&ELECTRONS`

`&ELECTRONS` 负责电子自洽迭代。

- `conv_thr` 是电子自洽收敛阈值。
- `mixing_beta` 控制电荷混合强度。
- 其他电子迭代设置会影响是否收敛、是否振荡和收敛速度。

审阅重点：output 中应出现 SCF iteration、estimated scf accuracy 和 convergence 信息。需要判断最终 estimated accuracy 是否满足本次目标，而不是只数迭代步数。

### Cards

Cards 负责把结构、赝势映射和 k 点采样写入 input。

- `ATOMIC_SPECIES`：元素标签、原子质量和赝势文件名。标签必须与坐标区一致。
- `ATOMIC_POSITIONS`：原子坐标和坐标单位。
- `K_POINTS`：SCF 用的 Brillouin zone 采样。SCF 通常使用 uniform mesh，不应把 bands k-path 当作 SCF k 点。
- `CELL_PARAMETERS`：当 `ibrav=0` 时显式给出晶格矢量和单位。

审阅重点：output 中 pseudopotentials loaded、number of k-points、crystal axes、atomic positions 或 symmetry 信息应能回扣 input。若 QE 改写了 symmetry、cell 或 k 点归约，需要记录。

## 必读 output

第一次读 `pw.scf.<system>.out` 时，不要只搜索最后几行。按下面顺序检查：

| Output 区域 | 必须读什么 | 为什么重要 |
|---|---|---|
| Program header | 程序名、QE version、并行信息、运行日期 | 记录环境和可复现性；不同版本可能影响默认行为和输出格式 |
| Calculation setup | calculation 类型、temporary directory、prefix、pseudo_dir | 确认 output 来自正确 input/command，没有读错任务 |
| Pseudopotentials loaded | 赝势文件、元素标签、PP 类型、价电子信息、泛函提示 | 防止赝势来源不明、标签错配或泛函混用 |
| Cutoff | kinetic-energy cutoff、charge density cutoff | 核对 `ecutwfc`、`ecutrho` 是否按 input 生效 |
| K-points | k mesh、shift、irreducible k-points、weights | 确认 SCF 使用 uniform sampling，且不是误用 bands path |
| SCF iterations | 每步 total energy、estimated accuracy、可能的振荡 | 判断电子自洽过程是否稳定 |
| Final total energy | 最终 `! total energy` 或等价汇总 | 记录本次 ground-state energy；只可与同设置计算比较 |
| Estimated accuracy | final estimated scf accuracy | 判断是否达到 `conv_thr` 和下游精度需要 |
| Fermi energy / occupation | Fermi energy、occupation、smearing 信息 | 判断 occupation 策略是否与体系预期一致 |
| Warnings | warning、error-like message、symmetry 变化、restart 信息 | 未解释 warning 会降低或阻断可信度 |

可使用下面的通用 output review 记录，不要把它当作 output 文件本身：

```markdown
## Output Review

- Program header:
- Calculation type:
- QE version:
- Command / input file:
- Prefix / outdir:
- Pseudopotentials loaded:
- Cutoff reported:
- K-points reported:
- Occupation / smearing:
- SCF iterations:
- SCF convergence message:
- Final total energy:
- Estimated SCF accuracy:
- Fermi energy / occupation:
- Warnings:
- Restart / scratch status:
- PASS / WARN / BLOCK:
- Reason:
- Allowed downstream workflows:
```

## PASS / WARN / BLOCK 判断

### PASS

可判为 PASS 的最低条件：

- output 来自正确 input、命令和工作目录。
- 程序完成，并且 SCF 明确达到 convergence。
- final estimated SCF accuracy 与 `conv_thr` 和本次目标相符。
- output 报告的赝势、cutoff、k-points、occupation 与 input 预期一致。
- 没有未解释 warning。
- calculation record 中记录了结构来源、赝势来源、参数、环境、output review 和下游决定。

PASS 只表示当前 SCF 对既定目标可用，不自动表示 cutoff、k 点、smearing 已完成系统收敛测试。

### WARN

以下情况通常是 WARN：

- 程序完成且 SCF 收敛，但 cutoff/k 点/smearing 还只是初始设置。
- 有 warning，但其含义已记录，且不直接破坏当前探索目标。
- 发生 restart 或 scratch 读取，但 `prefix/outdir`、输入和输出仍可追踪。
- occupation、Fermi energy 或 symmetry 信息提示需要进一步确认。

WARN 可以用于学习、探索或设计下一步测试；不应作为最终性质结论。

### BLOCK

以下情况应判为 BLOCK：

- SCF 未收敛，或 output 没有明确 convergence 证据。
- `prefix/outdir` 混用旧数据，或 output 与 input/command 对不上。
- 赝势来源不明、元素标签错配或泛函族混用。
- cutoff、k-points、occupation 与 input 预期不一致且无法解释。
- warning 指向输入错误、数值失败、对称性/结构严重异常或下游数据不可用。

BLOCK 不允许进入下游。下一步应是修复 input、清理 scratch、重新选择参数或重新设计 workflow。

## 下游准入

### 进入 convergence loop 前

需要一个能跑通并能审阅的 SCF 基线：

- SCF 至少达到 convergence，或明确知道当前 BLOCK 的原因。
- input/output/record 可以复现本次参数。
- 已记录初始 `ecutwfc`、`ecutrho`、k 点和 smearing 策略。

convergence loop 的目标是证明数值参数足以支持目标性质；它不是重新猜 input 的过程。

### 进入 relax / vc-relax 前

需要确认：

- 结构、赝势和 occupation 策略没有明显错误。
- 电子 SCF 能稳定收敛，否则 relax 中每个 ionic step 都可能失败。
- cutoff 和 k 点至少足以支撑力和应力的初步判断。

若后续目标是可信结构或声子，relax 之后通常还需要用最终结构重新做 static SCF。

### 进入 bands / DOS / PDOS 前

需要确认：

- 已有可信 SCF ground state，并保留一致的 `prefix/outdir`。
- SCF k 点是 ground-state sampling，不是 bands k-path。
- bands 需要后续沿 k-path 的计算；DOS/PDOS 通常需要 dense uniform mesh 的 NSCF。
- energy zero、Fermi level 和 occupation 策略已经能解释。

SCF PASS 是电子结构后处理的前置条件，但 bands/DOS/PDOS 还需要各自的 k 点和投影审阅。

### 进入 phonon / DFPT 前

需要更严格的前置条件：

- SCF 状态可信，且对 phonon 相关目标的 cutoff、k 点、smearing 已有收敛证据。
- 输入结构已经过适当 relax，残余力和应力状态可记录。
- `prefix/outdir` 可被 `ph.x` 正确读取。
- 没有未解释 warning、赝势混用或 scratch 混用。

phonon 对前置结构和电子基态质量更敏感。一个仅能用于学习的 WARN SCF 通常不足以支撑最终 phonon 结论。

## 可验证输出

- 一个使用通用命名的 `pw.scf.<system>.in`。
- 一个 `pw.scf.<system>.out` 的阅读笔记。
- 一份按 calculation record 模板填写的简短记录。
- 一个明确的 PASS / WARN / BLOCK 判断和下游准入决定。

## 推荐阅读

- `workflows/ground-state/scf.md`
- `standards/pass-warn-block.md`
- `standards/output-review-checklist.md`
- `standards/calculation-record-template.md`

## 后续进入哪个 workflow

完成后进入 `learn/03-convergence-loop.md` 和 `workflows/ground-state/cutoff-convergence.md`。

## 资料来源

- QE `pw.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>
- QE documentation: <https://www.quantum-espresso.org/documentation/>
- Pranab Das SCF tutorial: <https://pranabdas.github.io/espresso/hands-on/scf>
- 本仓库 `workflows/ground-state/scf.md`
- 本仓库 `standards/pass-warn-block.md`
- 本仓库 `standards/output-review-checklist.md`
- 本仓库 `standards/calculation-record-template.md`
