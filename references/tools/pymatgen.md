# pymatgen

## 它适合解决什么学习问题？

处理结构对象、格式转换和材料数据接口，辅助生成 QE 学习输入和后处理数据。

## 覆盖哪些 QE workflow？

- structure preparation
- k-path and symmetry workflows through related tools
- data conversion

## 适合跟读的部分

- 结构读写、对象检查、与 seekpath/spglib 类工具配合。

## 只适合作为参考的部分

- QE provenance、restart、scheduler 仍需本仓库 standards 或高级 provenance 工具。

## 与本仓库页面的对应关系

- [workflows/electronic/bands.md](../../workflows/electronic/bands.md)
- [standards/calculation-record-template.md](../../standards/calculation-record-template.md)

## 局限

pymatgen 不是 QE 原生命令行 workflow 的替代层；生成文件后仍需 output review。

## 原始链接

- [pymatgen documentation](https://pymatgen.org/)
- [pymatgen-io-espresso](https://github.com/oashour/pymatgen-io-espresso)
