# First SCF Loop

## 能力目标

建立第一个可信 SCF 闭环，理解 `pw.x` 如何从结构、赝势、cutoff 和 k 点得到自洽基态。

## 完成判据

- 能解释 `&CONTROL`、`&SYSTEM`、`&ELECTRONS`。
- 能解释 `ATOMIC_SPECIES`、`ATOMIC_POSITIONS`、`K_POINTS`、`CELL_PARAMETERS`。
- 能在 output 中找到 total energy、SCF iteration、estimated accuracy、Fermi energy、cutoff、k-points、pseudopotential 信息。
- 能判断结果是 PASS、WARN 还是 BLOCK。

## 可验证输出

- 一个 `pw.scf.<system>.in`。
- 一个 `pw.scf.<system>.out` 的阅读笔记。
- 一份简短 calculation record。

## 推荐阅读

- QE INPUT_PW reference
- Pranab Das QE SCF tutorial
- `workflows/ground-state/scf.md`

## 后续进入哪个 workflow

完成后进入 `learn/03-convergence-loop.md` 和 `workflows/ground-state/cutoff-convergence.md`。
