# Phonopy QE Interface

## 1. 它是什么？

Phonopy QE interface 是 phonopy 官方文档中说明如何用 QE 作为力计算后端的页面。

## 2. 适合学习或解决什么问题？

适合建立 finite-displacement phonon 与 QE DFPT phonon 的对照。

## 3. 它覆盖哪些 QE workflow？

supercell generation、QE force calculations、`FORCE_SETS`、phonon bands / DOS。

## 4. 哪些部分适合跟做？

适合在掌握 `ph.x -> q2r.x -> matdyn.x` 后跟做。

## 5. 哪些部分只适合作为参考？

它不替代 QE DFPT 主线；超胞结构生成细节应归入 `structure-learning/`。

## 6. 它和本仓库哪些页面对应？

- `workflows/phonon/phonon-dispersion-dfpt.md`
- `references/tools/phonopy-qe.md`

## 7. 它的局限是什么？

finite-displacement workflow 对结构、位移、力收敛和超胞大小敏感。

## 8. 原始链接与引用

- [Phonopy QE interface](https://phonopy.github.io/phonopy/qe.html)
