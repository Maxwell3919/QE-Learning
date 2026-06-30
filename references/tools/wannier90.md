# Wannier90

## 1. 它是什么？

Wannier90 是最大局域 Wannier 函数工具，QE 通过 `pw2wannier90.x` 与其对接。

## 2. 适合学习或解决什么问题？

适合能带插值、轨道模型、Berry 相关性质和输运前处理。

## 3. 它覆盖哪些 QE workflow？

高级电子结构、插值 bands、Wannier transport、EPW 前处理。

## 4. 哪些部分适合跟做？

先从简单半导体 bands 插值开始，再进入复杂投影和拓扑/输运。

## 5. 哪些部分只适合作为参考？

初学者不应在 SCF、bands、DOS 尚不稳定时直接进入 Wannierization。

## 6. 它和本仓库哪些页面对应？

- `workflows/spectroscopy-transport/wannier-transport.md`（待补）
- `references/tools/epw.md`

## 7. 它的局限是什么？

Wannier 投影质量和能窗选择会强烈影响结果，需要单独验证，不是自动后处理。

## 8. 原始链接与引用

- [QE INPUT_pw2wannier90](https://www.quantum-espresso.org/Doc/INPUT_pw2wannier90.html)
