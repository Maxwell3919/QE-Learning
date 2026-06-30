# 平面波基组与 cutoff

## 1. 这个理论在 QE 中对应什么问题？

对应 `ecutwfc` 和 `ecutrho` 如何控制波函数、电荷密度和势的数值表示。

## 2. 如果不懂它，会在计算中犯什么错？

会使用未收敛 cutoff，导致总能、力、应力和 phonon 虚频判断不可靠。

## 3. QE 输入文件中哪些参数和它相关？

`ecutwfc`、`ecutrho`、赝势类型、FFT grid 相关输出。

## 4. 输出文件中怎么看它是否出问题？

检查 kinetic-energy cutoff、charge density cutoff、FFT grid、总能/力随 cutoff 的变化。

## 5. 最低掌握深度

能独立设计 cutoff 收敛测试，并理解 `ecutrho` 对 USPP/PAW 可能更敏感。

## 6. 后续深入阅读

- QE `pw.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>

