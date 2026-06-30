# Smearing 与金属

## 1. 这个理论在 QE 中对应什么问题？

对应金属体系 occupation、Fermi surface 积分、SCF 稳定性和力/能量的一致性。

## 2. 如果不懂它，会在计算中犯什么错？

会把绝缘体设置套到金属，或把 DOS 积分设置用于结构优化。

## 3. QE 输入文件中哪些参数和它相关？

`occupations`、`smearing`、`degauss`、`tetrahedra` 相关设置。

## 4. 输出文件中怎么看它是否出问题？

检查 occupation scheme、Fermi energy、SCF 振荡、总能对 `degauss` 的敏感性。

## 5. 最低掌握深度

能为金属选择合理 smearing，并记录 `degauss` 收敛测试。

## 6. 后续深入阅读

- QE `pw.x` input reference: <https://www.quantum-espresso.org/Doc/INPUT_PW.html>

