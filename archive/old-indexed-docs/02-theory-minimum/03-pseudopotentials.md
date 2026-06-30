# 赝势

## 1. 这个理论在 QE 中对应什么问题？

对应 `ATOMIC_SPECIES` 中赝势文件的选择、cutoff 建议、泛函一致性和 transferability。

## 2. 如果不懂它，会在计算中犯什么错？

会混用不同泛函或不同质量的赝势，导致结果不可比。

## 3. QE 输入文件中哪些参数和它相关？

`pseudo_dir`、`ATOMIC_SPECIES`、`ecutwfc`、`ecutrho`、DFT+U/SOC 相关设置。

## 4. 输出文件中怎么看它是否出问题？

检查 output 中读取的 UPF 文件、PP 类型、valence configuration、warning。

## 5. 最低掌握深度

知道 NC/USPP/PAW 的基本差异，知道赝势来源必须记录，不能只记录文件名。

## 6. 后续深入阅读

- SSSP: <https://www.materialscloud.org/discover/sssp/table/efficiency>

