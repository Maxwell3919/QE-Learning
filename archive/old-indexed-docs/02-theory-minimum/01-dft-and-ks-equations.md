# DFT 与 Kohn-Sham 方程

## 1. 这个理论在 QE 中对应什么问题？

对应 `pw.x scf` 在固定结构下求自洽电子密度、总能和 Kohn-Sham eigenvalues。

## 2. 如果不懂它，会在计算中犯什么错？

会把所有 eigenvalues 当作直接可观测量，或者把 SCF 收敛误解为所有物理性质都可靠。

## 3. QE 输入文件中哪些参数和它相关？

`calculation`、`conv_thr`、`occupations`、`startingwfc`、`startingpot`、`mixing_beta`。

## 4. 输出文件中怎么看它是否出问题？

检查 SCF iteration、estimated accuracy、total energy 变化、Fermi energy 和 warning。

## 5. 最低掌握深度

能解释“自洽”为什么存在、总能和密度的关系、Kohn-Sham eigenvalues 的使用边界。

## 6. 后续深入阅读

- QE `pw.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>

