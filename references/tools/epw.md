# EPW

## 它适合解决什么学习问题？

作为 electron-phonon coupling、Wannier interpolation 和相关输运/超导性质的高级参考入口。

## 覆盖哪些 QE workflow？

- DFPT phonon
- electron-phonon overview
- Wannier interpolation
- advanced EPC analysis

## 适合跟读的部分

- 先读数据依赖图：SCF/NSCF、phonon、Wannier、EPW input 的关系。

## 只适合作为参考的部分

- EPW 不进入基础学习路径；需要先完成严格 SCF、phonon 和 Wannier 前置审阅。
- mobility、superconducting Tc、phonon linewidth、Eliashberg function、polaron 或 temperature-dependent spectra 是不同目标量，不能只看一个 `lambda` 数字。

## 最小 provenance 字段

EPW 相关记录至少应保留：

- SCF/NSCF 上游质量：cutoff、k mesh、smearing、Fermi level、`prefix/outdir`。
- `ph.x` q-grid、`tr2_ph`、dyn 文件完整性、`q2r.x/matdyn.x` 审阅。
- Wannier90 projection、window、QE bands 对照和接口文件。
- EPW coarse/fine k/q mesh、smearing、输出量名称、单位和定义。
- `PASS / WARN / BLOCK` 判断和允许进入的下游。

## BLOCK 条件

- phonon 或 Wannier 基础质量未通过，却解释 EPC 结果。
- 只报告 `lambda`、`Tc` 或单个 EPC 数值，没有 k/q mesh、smearing 和 Wannier sensitivity。
- coarse/fine mesh、单位或输出量定义不可追踪。
- `prefix/outdir` 或接口文件来自不一致的数据链。

## 与本仓库页面的对应关系

- [workflows/advanced/electron-phonon-overview.md](../../workflows/advanced/electron-phonon-overview.md)
- [workflows/advanced/wannier90-overview.md](../../workflows/advanced/wannier90-overview.md)
- [physics-judgement/epc-data-chain-and-convergence.md](../../physics-judgement/epc-data-chain-and-convergence.md)

## 局限

EPW 对前置数据质量高度敏感，不能用作修复未收敛 phonon 或不合格 Wannierization 的工具。

## 原始链接

- [EPW documentation](https://docs.epw-code.org/)
- [QE documentation](https://www.quantum-espresso.org/documentation/)
