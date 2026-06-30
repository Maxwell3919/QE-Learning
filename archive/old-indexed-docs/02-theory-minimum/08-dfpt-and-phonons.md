# DFPT 与 phonon

## 1. 这个理论在 QE 中对应什么问题？

对应 `ph.x` 如何在 SCF ground state 上计算一阶响应和 dynamical matrices。

## 2. 如果不懂它，会在计算中犯什么错？

会把 phonon 当成普通后处理，在未收敛结构上直接解释虚频。

## 3. QE 输入文件中哪些参数和它相关？

`tr2_ph`、`ldisp`、`nq1/nq2/nq3`、`epsil`、`recover`、`fildyn`。

## 4. 输出文件中怎么看它是否出问题？

检查每个 perturbation 的收敛、dyn 文件完整性、negative frequency、ASR 前后变化。

## 5. 最低掌握深度

能区分 Gamma phonon、q-grid phonon、IFC 插值、ASR 和非解析项。

## 6. 后续深入阅读

- QE `ph.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_PH.html>
- QE `matdyn.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_MATDYN.html>

