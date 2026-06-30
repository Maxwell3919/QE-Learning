# Phonopy QE Interface

## 1. 它是什么？

Phonopy 是 finite-displacement phonon 工具，提供 QE 接口，可以用 QE 计算超胞力，再由 phonopy 组装 force constants 和声子性质。

## 2. 适合学习或解决什么问题？

适合和 QE DFPT phonon 对照，理解 finite-displacement 与 DFPT 的差别。

## 3. 它覆盖哪些 QE workflow？

phonon dispersion、phonon DOS、force constants、Born charge 输入辅助。

## 4. 哪些部分适合跟做？

生成 supercell input、提取 forces、生成 `FORCE_SETS` 的流程。

## 5. 哪些部分只适合作为参考？

它不是 `ph.x -> q2r.x -> matdyn.x` 的替代学习主线。先学 QE DFPT，再用 phonopy 做对照更稳。

## 6. 它和本仓库哪些页面对应？

- `workflows/phonon/phonon-dispersion-dfpt.md`
- `structure-learning/README.md`

## 7. 它的局限是什么？

finite-displacement 对超胞、位移、力收敛和结构标准化敏感，不能绕过结构和收敛性检查。

## 8. 原始链接与引用

- [Phonopy QE interface](https://phonopy.github.io/phonopy/qe.html)
