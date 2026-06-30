# ASE

## 它适合解决什么学习问题？

把结构对象、格式转换和批量输入生成纳入脚本化学习，但仍保留 QE input/output 人工审阅。

## 覆盖哪些 QE workflow？

- structure preparation
- SCF input generation
- parameter sweeps
- light post-processing

## 适合跟读的部分

- read/write 结构对象、生成 QE input、管理参数扫描脚本。

## 只适合作为参考的部分

- ASE 的 QE IO 有实现边界；关键 input 仍需人工核对。

## 与本仓库页面的对应关系

- [workflows/ground-state/scf.md](../../workflows/ground-state/scf.md)
- [standards/input-file-naming.md](../../standards/input-file-naming.md)
- [standards/calculation-record-template.md](../../standards/calculation-record-template.md)

## 局限

ASE 不能判断赝势、cutoff、k 点或物理结果是否可信。

## 原始链接

- [ASE documentation](https://ase.gitlab.io/ase)
- [ASE espresso calculator source documentation](https://gitlab.com/ase/ase/blob/master/doc/ase/calculators/espresso.rst)
