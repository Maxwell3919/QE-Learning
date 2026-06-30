# QE Project Template

```text
project-name/
  README.md
  environment.md
  pseudo-source.md
  structures/
    original/
    standardized/
    relaxed/
  calculations/
    01-scf/
    02-convergence/
    03-relax/
    04-bands/
    05-dos-pdos/
    06-phonon/
    07-postprocessing/
  scripts/
    run.slurm
    parse_outputs.py
  records/
    calculation-records.md
  figures/
  data/
```

## README.md 至少包含

- 体系与科学问题。
- 当前 workflow 状态。
- 结构来源和赝势来源。
- 哪些计算已达到 `PASS`，哪些是 `WARN` 或 `BLOCK`。
- 下游计划。

## environment.md 至少包含

- QE 版本。
- 编译方式或模块名。
- MPI/OpenMP 设置。
- Python/ASE/pymatgen/seekpath 版本。
- 集群和队列信息。

