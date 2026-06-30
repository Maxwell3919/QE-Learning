# QE Workflow Map

## 主计算图

```text
<structure> + <pseudopotential> + <numerical_policy>
  -> pw.x scf
  -> output review: ground state PASS / WARN / BLOCK
  -> branch:
       A. relax / vc-relax
       B. electronic structure: bands / DOS / PDOS
       C. phonon / DFPT
       D. post-processing: density / potential / ELF / work function
       E. advanced: SOC / DFT+U / Wannier / EPC / MD / NEB
```

QE 的主线从 `pw.x` 建立 ground state 开始，再由多个下游程序读取同一套 workflow 数据。学习本图时要把每条箭头理解成数据依赖，而不是只理解成运行顺序。

## 核心状态

- structure：晶胞、原子、边界条件和坐标。
- pseudopotential：元素、泛函、赝势类型和 cutoff 建议。
- ground state：`pw.x scf` 产生的电荷密度、势、波函数和总能。
- convergence policy：cutoff、k 点、smearing、SCF 阈值和下游准入。
- optimized structure：`relax` 或 `vc-relax` 后的结构和 final static SCF。
- electronic observables：bands、DOS、PDOS、charge density、potential、ELF、work function。
- response observables：phonon、dielectric tensor、Born effective charge、IR/Raman、EPC。

## `prefix/outdir` 是 workflow 数据边

QE input/output 文件是给人读和记录的；`prefix/outdir` 是给 QE 程序之间传递数据的。下游程序通常用同一个 `prefix` 和 `outdir` 找到上游写出的 scratch 数据，例如 charge density、wavefunction、potential、eigenvalue 或 DFPT restart 数据；`.out` 文件主要用于人工审阅和记录。

```text
pw.scf.<system>.in
  prefix = '<system>'
  outdir = '<scratch_dir>'
        |
        v
<scratch_dir>/<system>.*  or  <scratch_dir>/<system>.save/
        |
        +-> pw.x bands / nscf / relax restart
        +-> ph.x
        +-> dos.x
        +-> bands.x
        +-> projwfc.x
        +-> pp.x
```

因此，workflow 记录必须同时写清：

- input/output 文件名，例如 `pw.scf.<system>.in` 和 `pw.scf.<system>.out`。
- `prefix = '<system>'`。
- `outdir = '<scratch_dir>'`。
- 下游步骤实际读取的是哪一次 SCF、relax 后 final SCF 或 NSCF 数据。
- 这次下游计算是否允许复用旧 scratch；如果是 restart，必须明确说明。

如果 `JOB DONE` 存在但 `prefix/outdir` 不一致，下游仍然可能读不到数据，或更危险地读到旧数据。对 workflow map 来说，这属于数据边断裂或数据边污染。

## 程序关系

`pw.x` 是主入口，负责 SCF、relax、vc-relax、NSCF 和 bands 路径上的电子结构计算。其他程序大多读取 `pw.x` 产生的数据，再生成更专门的 observable 或中间文件。

| 程序 | 主要输入依赖 | 主要产物 | 与 `pw.x` 的关系 |
|---|---|---|---|
| `pw.x scf` | `<structure>`、`<pseudo>`、`<numerical_policy>` | ground-state density、potential、wavefunctions、total energy | 主数据源 |
| `pw.x relax` / `vc-relax` | 初始结构、赝势、SCF 参数、力/应力阈值 | optimized structure、每一步 SCF、final geometry | 内部反复做电子 SCF |
| `pw.x nscf` | 已通过 review 的 SCF 数据、dense `<k_mesh>` | dense eigenvalues/wavefunctions | DOS/PDOS 常用前置 |
| `pw.x bands` | 已通过 review 的 SCF 数据、`<k_path>` | k-path eigenvalues | bands 分支的前置 |
| `ph.x` | 已通过 review 的 SCF 数据，通常还要求 optimized structure | dynamical matrices、DFPT response | 直接读取 SCF `prefix/outdir` |
| `q2r.x` | `ph.x` 产生的 q-grid dynamical matrices | real-space force constants | 不直接读 `pw.x`，读 `ph.x` 产物 |
| `matdyn.x` | `q2r.x` 产生的 force constants、`<q_path>` 或 q mesh | phonon frequencies、modes、phonon DOS 数据 | 不直接读 `pw.x`，读 `q2r.x` 产物 |
| `dos.x` | 通常读取 SCF/NSCF 的 eigenvalues | total DOS | 常接在 dense NSCF 后 |
| `bands.x` | `pw.x bands` 的 k-path eigenvalues | 整理后的 band data | 接在 `pw.x bands` 后 |
| `projwfc.x` | 通常读取 SCF/NSCF wavefunctions 与 projections 所需信息 | PDOS、projected states | 常接在 dense NSCF 后 |
| `pp.x` | SCF/NSCF 保存的 density、potential 或 wavefunction 相关数据 | charge density、potential、ELF 等网格数据 | 后处理读取 `pw.x` 数据 |

## 依赖顺序

### 1. SCF 是基础入口

```text
<structure> + <pseudo> + <numerical_policy>
  -> pw.x scf
  -> output review
```

SCF 通过后，才能把它作为 ground-state 数据源。通过的含义包括确认 calculation 类型、赝势、cutoff、k 点、occupation、SCF convergence、warning 和 scratch 状态都与目标一致，而不只看 `JOB DONE`。

### 2. relax / vc-relax 是结构优化分支

```text
<initial_structure>
  -> pw.x relax / vc-relax
  -> output review: forces / stress / final geometry
  -> final static pw.x scf on <optimized_structure>
```

`relax` 和 `vc-relax` 属于会改变结构的分支，不属于普通 SCF 的下游 observable。结构优化通过后，应对 final geometry 再做一次清楚记录的 static SCF，作为后续 electronic、phonon 和 post-processing 的数据源。

### 3. NSCF 是 DOS / PDOS 的常见前置

```text
reviewed pw.x scf
  -> pw.x nscf on dense <k_mesh>
  -> output review
  -> dos.x / projwfc.x
```

NSCF 读取 SCF 的自洽势，在更密的 k mesh 上计算 eigenvalues 和 wavefunctions。DOS/PDOS 通常不应直接拿粗 SCF k mesh 解释细节；是否需要 NSCF，取决于目标 observable 和收敛要求。

### 4. bands 使用 k-path，不等于 DOS 的 dense mesh

```text
reviewed pw.x scf
  -> pw.x bands on <k_path>
  -> output review
  -> bands.x
```

bands 分支沿高对称 k-path 取点，用于能带图；DOS/PDOS 分支通常用 uniform dense k mesh。二者都依赖 SCF，但它们是并列分支，不是“bands 算完再算 DOS”的线性关系。

### 5. phonon / DFPT 依赖更严格的 ground state

```text
optimized structure + reviewed strict pw.x scf
  -> ph.x
  -> output review: q points / perturbations / dyn files
  -> branch:
       Gamma: dynmat.x
       q-grid dispersion: q2r.x -> matdyn.x
       phonon DOS: q2r.x -> matdyn.x on q mesh
```

phonon 是 response workflow，对结构残余力、SCF 阈值、cutoff、k 点和 smearing 更敏感。`ph.x` 直接依赖 SCF 的 `prefix/outdir`；`q2r.x` 和 `matdyn.x` 则依赖 `ph.x` 产生的 dynamical matrices，不再直接读取 `pw.x` scratch。

### 6. post-processing 读取已有电子数据

```text
reviewed pw.x scf or nscf
  -> pp.x
  -> density / potential / ELF / other grid data
```

`pp.x` 属于后处理分支。它不能修复上游 SCF、结构或收敛问题，只能在已有数据可信时提取 density、potential、ELF 等网格量。

## 分支关系，不是单线流程

QE 学习主线可以按顺序学习，但实际 workflow 是分支图：

```text
                          +-> relax / vc-relax -> final scf
                          |
<structure> + <pseudo> -> scf -> convergence review
                          |
                          +-> bands branch: pw.x bands -> bands.x
                          |
                          +-> DOS branch: pw.x nscf -> dos.x
                          |
                          +-> PDOS branch: pw.x nscf -> projwfc.x
                          |
                          +-> phonon branch: ph.x -> q2r.x -> matdyn.x
                          |
                          +-> post-processing branch: pp.x
```

需要特别区分：

- `relax` / `vc-relax` 改变结构；bands、DOS、PDOS、phonon 和 `pp.x` 通常应基于已确认的结构状态。
- bands 与 DOS/PDOS 都从 SCF 分支出来，但 k 点目的不同；k-path 不是 dense k mesh 的替代品。
- DOS 与 PDOS 常共享 dense NSCF 数据，但 `dos.x` 与 `projwfc.x` 是并列后处理。
- phonon 的 Gamma、q-grid dispersion、phonon DOS、dielectric/Born effective charge 是 DFPT 内部不同分支。
- `q2r.x -> matdyn.x` 是 phonon q-grid 后的分支，不是 electronic post-processing。
- `pp.x` 读取电子数据做实空间后处理，不应被放在线性主线末端当作所有任务都必须运行的步骤。

## 用 output review 判断能否进入下游

每个 workflow 进入下游前都要写 output review，并给出 `PASS / WARN / BLOCK`。推荐判断方式：

| 上游步骤 | 进入下游前必须检查 | 允许下游的最低判断 |
|---|---|---|
| SCF | `JOB DONE`、SCF convergence、estimated accuracy、cutoff、k points、occupation、pseudo、warning、`prefix/outdir` | ground state 与目标一致；没有会污染下游的 warning |
| relax / vc-relax | final geometry、forces、stress、是否达到阈值、是否有异常步、是否需要 final static SCF | 结构状态可解释；final structure 已记录；下游使用的是 final static SCF |
| convergence | cutoff/k mesh/smearing 对目标量的影响 | 参数策略足够支持目标 workflow |
| NSCF | 是否读取正确 SCF、dense k mesh 是否符合 DOS/PDOS 目标、band count 是否足够 | eigenvalue/wavefunction 数据覆盖目标能量范围 |
| bands | 是否读取正确 SCF、k-path 是否正确、bands 数是否足够 | k-path 数据可用于画图和判读，不替代 DOS 收敛 |
| DOS / PDOS | 是否读取正确 NSCF/SCF、能量网格、展宽、投影信息 | DOS/PDOS 设置与目标解释一致 |
| phonon `ph.x` | q 点列表、每个 perturbation 收敛、dyn 文件完整、warning、SCF 依赖 | dyn 数据完整且 DFPT response 收敛 |
| `q2r.x` | 是否读取完整 dyn q-grid、是否写出 force constants、ASR 设置 | force constants 可进入 `matdyn.x` |
| `matdyn.x` | q-path 或 q mesh、frequency 文件、Gamma 声学支、negative frequencies | 频率数据可解释；虚频判断有复查记录 |
| `pp.x` | 读取的数据源、plot quantity、网格和单位 | 上游电子数据可信，输出量与目标一致 |

`PASS` 表示可以进入明确列出的下游 workflow；`WARN` 表示可以继续做有限的探索或画图，但结论必须保留 caveat；`BLOCK` 表示不应进入下游解释，先修复输入、参数、结构、收敛或数据边问题。

常见 blocking 信号：

- 只有 `JOB DONE`，但没有达到 SCF 或 DFPT convergence。
- output 中实际 cutoff、k 点、occupation 或 pseudo 与记录不一致。
- `prefix/outdir` 指向旧 scratch，或下游读取来源不清楚。
- relax 后 residual force/stress 不满足下游目标，却直接进入 phonon。
- 用 bands k-path 数据替代 DOS/PDOS 的 dense k mesh。
- `ph.x` dyn 文件不完整，却继续运行 `q2r.x` 或解释 `matdyn.x` 结果。
- negative frequency 未区分数值误差、结构未平衡、ASR/插值问题和真实不稳定。

## 学习原则

每条边都要能回答三个问题：上游文件是什么，下游程序读什么，output 如何判断是否可信。

最低完成判据：

- 能画出 `pw.x scf` 到 electronic、phonon、post-processing 的分支图。
- 能说明 `prefix/outdir` 为什么是 QE workflow 的数据边。
- 能区分 SCF、relax、NSCF、bands、DOS、PDOS、phonon 和 `pp.x` 的依赖顺序。
- 能判断一个 output review 是允许下游、只允许探索，还是必须阻断。
