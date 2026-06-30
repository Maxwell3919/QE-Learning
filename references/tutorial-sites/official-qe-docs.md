# Official Quantum ESPRESSO Documentation

## 1. 它是什么？

QE 官方文档是参数、程序边界和包结构的权威来源。它不应该被当作唯一学习路线，但每个 workflow 页面中的 input 参数和程序分工都必须回到官方文档核对。

## 2. 适合学习或解决什么问题？

- 查 `pw.x`、`ph.x`、`bands.x`、`pp.x` 等输入参数。
- 确认某个功能属于哪个 QE package。
- 判断参数默认值、适用条件和版本边界。
- 补充高级包入口，如 TDDFPT、XSPECTRA、GWL、PWcond、EPW、GIPAW、atomic。

## 3. 它覆盖哪些 QE workflow？

- Ground-state：PWscf / `pw.x`
- Phonon：PHonon / `ph.x`、`q2r.x`、`matdyn.x`、`dynmat.x`
- Post-processing：PostProc / `pp.x`、`bands.x`、`dos.x`、`projwfc.x`
- Transport and spectroscopy：PWcond、XSPECTRA、TDDFPT、GWL
- Pseudopotential generation：atomic / `ld1.x`
- Interfaces and ecosystem：Wannier90、YAMBO、PLUMED、EPW、GIPAW

## 4. 哪些部分适合跟做？

官方 docs 更适合核对，不适合作为跟做教程。适合跟做的材料应来自教程站点或个人学习示例，再回到官方 input reference 验证参数。

## 5. 哪些部分只适合作为参考？

- 单个程序的完整 input reference。
- user guide 中的并行、restart、包结构说明。
- 高级包说明和接口文档。

## 6. 它和本仓库哪些页面对应？

- 所有 `workflows/` 页面。
- [standards/citation-and-source-policy.md](../../standards/citation-and-source-policy.md)
- [references/source-index.md](../source-index.md)

## 7. 它的局限是什么？

官方文档以 package 和 executable 为中心，端到端教学较弱。它说明参数是什么，但通常不会告诉初学者如何组织 SCF -> convergence -> relax -> bands/DOS -> phonon 的学习路线。

## 8. 原始链接与引用

- [QE documentation](https://www.quantum-espresso.org/documentation/)
- [QE ecosystem](https://www.quantum-espresso.org/ecosystem/)
- [QE INPUT_PW](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
- [QE INPUT_PH](https://www.quantum-espresso.org/Doc/INPUT_PH.html)
- [QE INPUT_PP](https://www.quantum-espresso.org/Doc/INPUT_PP.html)
- [QE INPUT_BANDS](https://www.quantum-espresso.org/Doc/INPUT_BANDS.html)
- [QE INPUT_Q2R](https://www.quantum-espresso.org/Doc/INPUT_Q2R.html)
- [QE pseudopotentials](https://www.quantum-espresso.org/pseudopotentials/)
