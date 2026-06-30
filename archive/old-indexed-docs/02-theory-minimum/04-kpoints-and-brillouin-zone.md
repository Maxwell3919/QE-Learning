# k 点与 Brillouin zone

## 1. 这个理论在 QE 中对应什么问题？

对应 `K_POINTS automatic`、bands high-symmetry path、DOS dense mesh 和 symmetry reduction。

## 2. 如果不懂它，会在计算中犯什么错？

会混淆 bands path 与 DOS mesh，或使用与晶胞不匹配的高对称路径。

## 3. QE 输入文件中哪些参数和它相关？

`K_POINTS`、`nosym`、`noinv`、`occupations`、`nbnd`。

## 4. 输出文件中怎么看它是否出问题？

检查 irreducible k-points、k 点坐标、symmetry 信息和 bands 路径标签。

## 5. 最低掌握深度

能区分 uniform mesh 与 high-symmetry path，知道 primitive cell 与 k-path 的关系。

## 6. 后续深入阅读

- SeeK-path: <https://seekpath.readthedocs.io/>

