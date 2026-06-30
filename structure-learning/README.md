# Structure Learning

本目录是后续单独研讨“材料结构操作学习”的入口。本轮只做边界说明，不深入展开。

## 未来覆盖内容

- CIF 文件读取与检查
- primitive / conventional cell
- symmetry detection
- space group
- Wyckoff positions
- supercell
- slab
- vacuum
- 2D materials
- defects
- substitution
- vacancy
- interstitial
- strain
- k-path 与结构标准化关系
- ASE / pymatgen / spglib / seekpath / VESTA / XCrySDen 在结构学习中的分工

## 与 QE-Learning 的边界

QE workflow 页面只说明“输入结构必须已检查”和“结构标准化会影响 k-path / phonon / slab / defects”。结构操作的具体教学、工具比较和案例构造后续在本目录展开。

## 当前引用点

- `learn/00-start-here.md`：说明结构学习单独展开。
- `workflows/ground-state/scf.md`：SCF 前要求结构已检查。
- `workflows/electronic/bands.md`：k-path 依赖结构标准化。
- `workflows/advanced-native/neb-reaction-path.md`：NEB 依赖路径结构构造。
