# SeeK-path

## 1. 它是什么？

SeeK-path 是用于晶体结构标准化和高对称 k-path 生成的工具。

## 2. 适合学习或解决什么问题？

它解决 bands workflow 中最常见的问题：k-path 必须与标准化后的晶胞和 reciprocal-space convention 一致。

## 3. 它覆盖哪些 QE workflow？

主要覆盖 `SCF -> bands` 的 k-path 生成，也影响结构标准化和 band label 解释。

## 4. 哪些部分适合跟做？

用 Si/GaAs 结构生成 primitive cell 与 high-symmetry path，并和 QE `K_POINTS crystal_b` 对照。

## 5. 哪些部分只适合作为参考？

复杂磁性、非标准 cell、低维体系或人为 symmetry breaking 场景，需要人工判断是否使用标准 k-path。

## 6. 它和本仓库哪些页面对应？

- `workflows/electronic/bands.md`
- 结构操作独立学习项目

## 7. 它的局限是什么？

SeeK-path 给出路径，不保证你的 SCF、结构、赝势和 bands 计算已经可信。

## 8. 原始链接与引用

- [SeeK-path documentation](https://seekpath.readthedocs.io/en/latest/maindoc.html)
