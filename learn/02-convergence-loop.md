# Convergence Loop

## 我为什么要学这个？

cutoff、`ecutrho`、k-points 和 smearing 决定 QE 数值问题本身。只看一个 input 能跑通，不能说明力、应力、DOS 或 phonon 可信。

## 它在 QE workflow 中处于哪个位置？

```text
first SCF
  -> cutoff / ecutrho tests
  -> k-point tests
  -> smearing tests
  -> selected numerical policy
  -> relax / bands / DOS / phonon
```

## 我需要读哪些外部资料？

- QE INPUT_PW
- Pranab convergence tutorial
- SSSP / pseudopotential recommended cutoff notes

## 我需要跑什么最小案例？

- Si cutoff + k-point convergence。
- Al smearing + k-point convergence。

## output 应该怎么看？

不要只看 total energy。根据目标 workflow 还要看：

- force max
- stress
- Fermi energy
- DOS near Fermi level
- phonon sensitivity

## 结果什么时候可以进入下游？

当 convergence report 明确记录测试序列、固定参数、目标判据和选择理由后，才允许进入 relax、bands/DOS 或 phonon。

## 当前页面还缺哪些验证？

- 还缺 convergence table 示例。
- 还缺不同目标性质的阈值建议。
