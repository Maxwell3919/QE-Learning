# Symmetry 与 space group

## 1. 这个理论在 QE 中对应什么问题？

对应 k 点约化、结构标准化、band path、简并和不可约表示。

## 2. 如果不懂它，会在计算中犯什么错？

会使用错误 k-path，或误判由对称性导致的能带简并/劈裂。

## 3. QE 输入文件中哪些参数和它相关？

`ibrav`、`CELL_PARAMETERS`、`ATOMIC_POSITIONS`、`nosym`、`noinv`、`K_POINTS`。

## 4. 输出文件中怎么看它是否出问题？

检查 QE 识别的 symmetry operations、space group 相关信息、irreducible k-points。

## 5. 最低掌握深度

能解释 primitive/conventional cell、标准 k-path 与 symmetry reduction 的关系。

## 6. 后续深入阅读

- SeeK-path: <https://seekpath.readthedocs.io/>

