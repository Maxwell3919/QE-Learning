# HPC and Reproducibility

## 能力目标

理解 QE 运行环境和可复现记录，避免把完成运行误认为完成科学判断。

## 覆盖内容

- shell scripts
- SLURM scripts
- `prefix/outdir`
- restart
- QE version
- pseudopotential source
- output review

## 完成判据

- 能说明 `prefix/outdir` 为什么是 workflow 数据边。
- 能记录 QE version、command、MPI/OMP 设置和运行环境。
- 能记录 pseudopotential source。
- 能按 `standards/output-review-checklist.md` 做 output review。

## AiiDA 边界

AiiDA / AiiDA-QE 可作为高级 workflow/provenance 参考，用于理解 provenance graph、restart 和 scheduler 集成；它不作为 QE 入门主线。

## 推荐阅读

- QE user guide parallelization / restart notes
- ENCCS QE HPC tutorials
- `references/tools/aiida-qe-optional.md`
