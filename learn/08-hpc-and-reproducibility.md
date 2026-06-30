# HPC and Reproducibility

## 能力目标

理解 QE 运行环境和可复现记录，避免把完成运行误认为完成科学判断。到这一页为止，学习重点不再只是“能不能跑完”，而是能不能让另一个人根据记录判断：使用了什么程序、什么赝势、什么命令、什么并行方式、读写了哪些中间数据，以及 output 是否足以支持下游 workflow。

QE 计算依赖输入文件、赝势、程序版本、并行布局、运行目录和 scratch 数据。任何一个环节没有记录，都可能导致同一个 input 在另一台机器、另一个队列或另一次 restart 中得到不同的运行行为。可复现记录用于把计算结果从“本机跑过”提升为“可以复查”的状态。

## 覆盖内容

- shell scripts
- SLURM scripts
- `prefix/outdir`
- scratch and restart
- QE version
- parallel settings
- pseudopotential source
- command
- output review

## 为什么 QE 学习需要记录运行环境

QE 学习早期容易只关注 input 语法和 output 是否出现 `JOB DONE`。这不够。QE workflow 中的很多判断都依赖运行环境：

- QE version 决定可用输入变量、默认行为、warning 文本和部分 output 格式。
- MPI ranks、OpenMP threads、pool、band/task diagonalization 等并行设置会影响性能、内存分布、I/O 压力，有时也会影响是否能稳定完成。
- pseudopotential source 决定赝势文件、泛函、推荐 cutoff 和适用边界。
- `prefix/outdir` 决定本次计算是否读到了预期的 wavefunction、charge density、restart 文件或旧 scratch。
- scheduler、module、container、工作目录和提交命令决定 output 是否能追溯到实际运行方式。

因此，记录运行环境的目标是让 output review 能回答三个问题，而非复刻一台机器：

1. 这份 output 对应哪一组 input、赝势和命令？
2. 这次运行是否读写了可追踪的中间数据？
3. 当前结果是否允许进入下游，还是只能作为探索或失败记录？

## `prefix/outdir`、scratch 和 restart

`prefix` 和 `outdir` 是 QE workflow 的数据边。许多 QE 程序并不只通过文本 input 传递信息，还会通过 `outdir` 下以 `prefix` 命名的数据读取 charge density、wavefunction、XML/schema 文件、restart 文件和其他中间状态。后续 SCF、NSCF、bands、DOS、phonon 或 restart 是否接上了正确的上游，取决于这些字段是否一致且可追踪。

基本关系如下：

```text
input + pseudopotential + command + environment
  -> QE run
  -> text output
  -> outdir/prefix scratch data
  -> restart or downstream program
```

需要区分三个概念：

| 名称 | 含义 | 记录要求 |
|---|---|---|
| `prefix` | QE 用来标识同一组中间数据的名字 | 记录实际值；不同测试不要随意复用同一个 `prefix` |
| `outdir` | QE 写入 scratch / save 数据的位置 | 记录绝对路径或相对路径基准；说明是否保留 |
| scratch | 大型临时数据区域，常含 `.save`、wavefunction、charge density、restart 数据 | 不一定纳入文档仓库，但必须说明位置、生命周期和清理策略 |
| restart | 从旧中间数据继续运行或恢复失败运行 | 记录从哪里继续、是否复用旧 `prefix/outdir`、output 中是否有 restart 证据 |

常见风险是 `prefix/outdir` 混用：新 input 看起来已经修改，但程序实际读到了旧 charge density、旧 wavefunction 或旧结构状态。遇到这种情况，即使 output 正常结束，也应至少标为 `WARN`；如果无法确认数据来源，应标为 `BLOCK`。

## Shell 脚本记录字段

如果用 shell script 运行 QE，记录不只保存脚本文件名，还要保存足以复现命令行为的字段：

- script path and version: 脚本路径、修改时间或 git commit。
- working directory: 提交时所在目录。
- command: 实际执行命令，例如 `pw.x -in pw.scf.in > pw.scf.out`。
- executable path: `pw.x`、`ph.x` 等来自 PATH、module、conda、spack、container 还是手动编译目录。
- QE version: `pw.x -version`、output header 或 module 名称中可验证的版本。
- environment variables: 至少记录 `OMP_NUM_THREADS`、重要 library path、QE 相关 module。
- MPI launcher: `mpirun`、`mpiexec`、`srun` 或本地串行运行。
- parallel settings: MPI ranks、OpenMP threads、`-npool`、`-ndiag`、`-nband` 等实际使用值。
- input/output filenames: input、output、error log 的文件名。
- `prefix/outdir`: 与 input 中保持一致，并说明 scratch 是否保留。

学习记录中不要求保存所有 stdout/stderr 旁路文件，但至少要能把 `record.md`、input、output 和脚本互相对应起来。

## SLURM 脚本记录字段

SLURM 脚本还需要记录 scheduler 层信息，因为排队系统会改变运行资源、工作目录和实际启动方式：

- job id and job name: 用于追踪队列记录和输出文件。
- partition / queue / account: 说明运行在哪类资源上。
- nodes, tasks, cpus-per-task: 对应 MPI ranks 和 OpenMP threads。
- memory and walltime: 判断失败是否可能来自资源不足或超时。
- module / environment setup: `module load`、container image、激活环境。
- launch command: `srun pw.x ...`、`mpirun pw.x ...` 或站点推荐启动方式。
- stdout/stderr paths: SLURM output、QE output、error log 分别在哪里。
- scratch staging: 是否把数据复制到节点本地 scratch，结束后是否复制回项目目录。
- restart policy: 超时、失败或手动续算时如何复用或隔离 `prefix/outdir`。

如果 SLURM job 失败，不能只写“计算失败”。需要在 output review 中区分：QE 自身报错、scheduler 超时、节点/文件系统问题、module/library 问题、内存不足、scratch 未复制回，还是 restart 数据不完整。

## QE version、并行设置、赝势来源和命令

每次可复查计算至少记录以下四类字段。

### QE version

记录来源要可验证：

- output header 中的 QE version。
- `pw.x -version` 或对应程序的版本输出。
- module 名称、container tag 或编译目录。
- 如有必要，记录编译特征，例如 MPI、OpenMP、ELPA、ScaLAPACK、CUDA 等。

### Parallel settings

并行设置不要只写“用了 HPC”。至少记录：

- MPI ranks: 例如 SLURM `ntasks` 或 `mpirun -np`。
- OpenMP threads: `OMP_NUM_THREADS` 或 `--cpus-per-task` 对应值。
- QE parallel flags: 例如 `-npool`、`-ndiag`、`-nband`、`-ntg`。
- launcher: `srun`、`mpirun`、`mpiexec` 或串行。
- output 中的并行信息：程序启动时打印的 processors、pools、threads 等信息。

这些字段既用于性能复查，也用于解释并行 I/O、内存、walltime 和 restart 行为。

### Pseudopotential source

赝势不能只写文件名。记录应包含：

- library or website: 赝势库或下载来源。
- file names: 实际使用的 `.UPF` 文件。
- exchange-correlation: 与 input 中的泛函选择是否一致。
- PP type: NC、USPP 或 PAW。
- recommended cutoff: 来源给出的建议值，以及本次 input 使用值。
- local copy path or checksum: 至少能确认后续没有换文件。

如果赝势来源不明、泛函混用或文件名与 input 对不上，结果不能直接进入下游。

### Command

记录 command 时写实际执行的完整命令，而不是概念描述：

```bash
srun -n 32 pw.x -npool 4 -in pw.scf.in > pw.scf.out
```

如果 shell/SLURM 脚本中有变量展开，记录展开后的关键值，例如 input 文件、output 文件、`prefix`、`outdir`、MPI ranks 和 OpenMP threads。这样 output review 才能检查脚本、input 和 output 是否一致。

## Output review：restart、scratch 和并行信息

HPC 页面中的 output review 重点是确认运行证据链是否完整；物理结果判断仍回到对应 workflow 页面。审查时按下面顺序检查：

1. 程序身份：output header 是否显示预期 QE 程序和版本。
2. 命令对应：output 文件名、input 文件名和记录中的 command 是否一致。
3. 赝势对应：output 中读取的 pseudopotential 是否与记录中的来源和文件一致。
4. `prefix/outdir`：input、record 和脚本是否写同一组值；是否可能读到旧 scratch。
5. Restart 证据：output 是否显示从 restart 或已有数据继续；这是否符合记录中的意图。
6. Scratch 证据：scratch 数据是否保留、清理或复制回；下游程序是否依赖这些数据。
7. 并行信息：processors、pools、threads、diagonalization 等信息是否与脚本和资源申请一致。
8. 结束状态：区分 `JOB DONE`、SCF/ionic convergence、walltime kill、scheduler error 和 library/environment error。
9. 下游准入：根据证据给出 `PASS`、`WARN` 或 `BLOCK`。

判断示例不依赖具体材料：

- `PASS`：版本、命令、赝势来源、`prefix/outdir`、并行设置和 output convergence 均可追踪；没有未解释 warning。
- `WARN`：发生 restart 或 scratch staging，但来源清楚，output 与记录一致；可用于探索或继续检查。
- `BLOCK`：`prefix/outdir` 与记录不一致、scratch 丢失、restart 来源不明、赝势文件对不上、output 与 command 对不上，或 scheduler/environment 错误导致结果不完整。

对于 restart，尤其要写清它是“有意续算”还是“意外读到旧数据”。有意续算可以是正常 workflow 的一部分；意外读旧数据会破坏可复现性。

## AiiDA 边界

AiiDA / AiiDA-QE 可作为高级 workflow/provenance 参考，用于理解 provenance graph、restart management 和 scheduler integration。它适合帮助学习者理解为什么要记录输入、输出、代码版本、远程计算机、scheduler 和 restart 关系。

但本仓库的基础路线仍然是 QE 原生命令行 workflow。AiiDA 不替代手动阅读 input/output，不替代收敛测试，不替代 PASS / WARN / BLOCK 判断，也不作为本页的执行主线。建议在能够手动完成并审查 SCF、relax、电子结构、phonon 和后处理之后，再把 AiiDA-QE 作为 provenance 系统的参考材料阅读。

## 完成判据

- 能说明 `prefix/outdir` 为什么是 workflow 数据边。
- 能区分 scratch、restart 和普通文本 output。
- 能记录 QE version、command、MPI/OMP 设置和运行环境。
- 能记录 pseudopotential source，并检查它是否与 input/output 对应。
- 能从 shell 或 SLURM 脚本提取可复查字段。
- 能在 output review 中处理 restart、scratch 和并行运行信息。
- 能说明 AiiDA / AiiDA-QE 只作为 provenance 理解参考，不作为基础学习主线。

## 推荐阅读

- QE user guide: running, parallelization, restart and input/output behavior.
- ENCCS Quantum ESPRESSO HPC tutorials.
- `standards/calculation-record-template.md`
- `standards/project-layout.md`
- `standards/pass-warn-block.md`
- `references/tools/aiida-qe-optional.md`
