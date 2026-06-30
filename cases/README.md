# Cases

本目录用于放最小可复现 QE 案例。本轮只建立目录和案例规范，不提供完整 input/output。

## 每个 case 最终必须包含

```text
README.md
record.md
pseudo-source.md
run.sh 或 job.slurm
inputs/
outputs/
figures/
notes.md
```

## record.md 必须包含

- System
- Goal
- Structure source
- Pseudopotential source
- Input files
- Command
- Environment
- Output summary
- Convergence status
- Physical result
- Problems encountered
- Next action
- PASS / WARN / BLOCK

## 优先案例

- Si：SCF、convergence、bands、DOS、phonon 基础。
- Al：metallic smearing、DOS、Fermi surface。
- GaAs：polar phonon、Born effective charge、LO-TO splitting。
- Graphene：2D 材料、k-path、phonon 特殊问题。
