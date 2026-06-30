# SCF 数值收敛

## 1. 这个理论在 QE 中对应什么问题？

对应电子密度混合、残差下降和自洽失败诊断。

## 2. 如果不懂它，会在计算中犯什么错？

会把混合振荡、结构错误或金属性问题误判为物理不稳定。

## 3. QE 输入文件中哪些参数和它相关？

`conv_thr`、`mixing_beta`、`mixing_mode`、`electron_maxstep`、`diagonalization`。

## 4. 输出文件中怎么看它是否出问题？

看 estimated accuracy 是否下降、总能是否振荡、是否达到 maximum number of iterations。

## 5. 最低掌握深度

能按顺序排查结构、赝势、smearing、mixing、cutoff/k 点。

## 6. 后续深入阅读

- AiiDA-QE restart/error handling docs: <https://aiida-quantumespresso.readthedocs.io/>

