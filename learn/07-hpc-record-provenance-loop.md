# HPC Record Provenance Loop

## 我为什么要学这个？

真实 QE 计算通常在 HPC 上运行。没有环境、脚本、`prefix/outdir`、restart 和输出记录，计算即使完成也难以复现。

## 它在 QE workflow 中处于哪个位置？

```text
workflow input
  -> local or SLURM script
  -> QE scratch state in prefix/outdir
  -> output + record.md
  -> PASS/WARN/BLOCK decision
```

## 我需要读哪些外部资料？

- QE user guide parallelization / restart notes
- AiiDA-QE workflow logic
- ENCCS / HPC QE tutorials

## 我需要跑什么最小案例？

同一 SCF 在本地脚本和 SLURM 脚本中的记录对照。

## output 应该怎么看？

- job 是否正常结束。
- walltime / restart 是否发生。
- `prefix/outdir` 是否保留。
- MPI ranks / OMP threads / QE version 是否记录。
- output 是否来自本次 input，而不是旧 scratch。

## 结果什么时候可以进入下游？

当 `record.md` 包含结构、赝势、input、command、environment、output summary、convergence status、physical result 和 next action 后，才能作为项目数据使用。

## 为什么不第一步就用 AiiDA？

AiiDA 是成熟 provenance/workflow layer，但会引入数据库、daemon、profile 和 scheduler 配置。先手动理解 QE 文件数据边，再接入 AiiDA 更稳。

## 当前页面还缺哪些验证？

- 还缺 SLURM 模板。
- 还缺 restart 成功/失败记录示例。
