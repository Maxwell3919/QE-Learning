# Pseudopotential problems

## 核心问题

赝势错误通常不能靠调 `mixing_beta` 或 cutoff 完全解决。

## 可能表现

- cutoff 要求异常高。
- 结果与已知参考明显不符。
- 与目标泛函、SOC、semicore、磁性需求不匹配。

## 排查顺序

1. 查赝势来源和版本。
2. 检查泛函一致性。
3. 检查 valence configuration。
4. 与 SSSP/PseudoDojo 等来源比较。

