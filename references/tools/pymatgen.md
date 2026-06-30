# pymatgen

## 1. 它是什么？

pymatgen 是 Materials Project 生态中的材料结构、对称性、超胞和分析库。

## 2. 适合学习或解决什么问题？

适合处理结构标准化、超胞、缺陷前处理和材料数据结构。QE IO 需要谨慎，因为 `.docx` 调研指出 QE 支持主要依赖外部 `pymatgen-io-espresso`。

## 3. 它覆盖哪些 QE workflow？

结构准备、缺陷/超胞准备、材料数据清洗、可能的 input 生成。

## 4. 哪些部分适合跟做？

结构读写、primitive/conventional cell、space group、supercell。

## 5. 哪些部分只适合作为参考？

QE input/output 写入读取相关功能应先小案例验证，不应直接作为生产标准。

## 6. 它和本仓库哪些页面对应？

- `structure-learning/README.md`
- `workflows/electronic/bands.md`
- `workflows/advanced-native/neb-reaction-path.md`（待补）

## 7. 它的局限是什么？

它不是 QE-native workflow engine。QE provenance、restart、scheduler 仍需 AiiDA 或本仓库 standards 记录。

## 8. 原始链接与引用

- [pymatgen-io-espresso](https://github.com/oashour/pymatgen-io-espresso)
