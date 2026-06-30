# First SCF Loop

## 我为什么要学这个？

SCF 是大多数 QE workflow 的 ground-state 入口。没有可信 SCF，后续 relax、bands、DOS、PDOS、phonon 和 post-processing 都没有可靠基础。

## 它在 QE workflow 中处于哪个位置？

```text
checked structure + pseudopotential + cutoff + k-points
  -> pw.x scf
  -> ground-state density / total energy / scratch state
  -> downstream workflows
```

## 我需要读哪些外部资料？

- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
- [Pranab Das SCF tutorial](https://pranabdas.github.io/espresso/hands-on/scf)
- [official QE docs](../references/tutorial-sites/official-qe-docs.md)

## 我需要跑什么最小案例？

Si 或 Al。Si 适合第一个 fixed-occupation semiconductor SCF；Al 适合金属 smearing 入门。

## output 应该怎么看？

至少检查：

- `convergence has been achieved`
- final total energy
- SCF iterations
- estimated SCF accuracy
- Fermi energy
- loaded pseudopotentials
- kinetic-energy cutoff / charge density cutoff
- irreducible k-points
- warnings

## 结果什么时候可以进入下游？

只有当 SCF 收敛、输入来源清楚、赝势和 cutoff/k 点有最低依据，并写入 `record.md` 后，才允许进入 convergence、relax 或 electronic workflow。`JOB DONE` 不是充分条件。

## 当前页面还缺哪些验证？

- 还缺 Si 和 Al 的真实 input/output。
- 还缺 output 行号级解读示例。
