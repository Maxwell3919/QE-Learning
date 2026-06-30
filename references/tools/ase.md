# ASE

## 1. 它是什么？

ASE（Atomic Simulation Environment）是 Python 原子结构与计算工作流脚本库，可用于结构构建、格式转换、批量生成输入和轻量后处理。

## 2. 适合学习或解决什么问题？

适合把 CIF/结构对象转成可脚本处理的结构，生成参数扫描，管理小规模 QE input/output。它不替代 QE，也不替代收敛判断。

## 3. 它覆盖哪些 QE workflow？

结构准备、SCF input 生成、convergence sweep、relax 输入整理、批量 case 组织。

## 4. 哪些部分适合跟做？

学习 structure object、read/write、supercell、简单 QE input writer。

## 5. 哪些部分只适合作为参考？

ASE 的 QE IO 有实现边界；`.docx` 调研指出其读写路径并不覆盖全部 `ibrav` 模式，实际使用应优先保持 `ibrav=0` 风格并人工核对 cell。

## 6. 它和本仓库哪些页面对应？

- `structure-learning/README.md`
- `workflows/ground-state/scf.md`
- `standards/source-traceability.md`

## 7. 它的局限是什么？

ASE 能帮助构造文件，但不能判断赝势、cutoff、k 点和物理结果是否可信。

## 8. 原始链接与引用

- [ASE documentation](https://ase.gitlab.io/ase)
- [ASE espresso IO source](https://ase-lib.org/_modules/ase/io/espresso.html)
