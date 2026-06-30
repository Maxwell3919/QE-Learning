# Wannier90

## 它适合解决什么学习问题？

学习 QE 电子结构到 Wannier interpolation 的数据边界，用于能带插值、Berry 相关量和输运前置分析。

## 覆盖哪些 QE workflow？

- QE -> pw2wannier90.x -> Wannier90
- bands interpolation
- advanced transport pre-processing

## 适合跟读的部分

- 理解 projections、disentanglement、frozen window 和 `pw2wannier90.x` 接口。

## 只适合作为参考的部分

- Wannierization 质量不能只看流程完成，需要检查插值 bands 与 QE bands 对照。
- 完整 window/projection 优化、Berry curvature、transport、topology 或 EPC 插值属于高级专题；本页只定义学习入口和审阅边界。

## 最小 provenance 字段

学习或科研记录中至少应保留：

- QE SCF/NSCF input 和 output。
- `pw2wannier90.x` input、output 和接口文件。
- Wannier90 `.win`、`.wout`、window、projection、`nbnd` 和 k mesh。
- QE reference bands 与 Wannier-interpolated bands 的对照。
- `PASS / WARN / BLOCK` 判断和进入下游的理由。

## BLOCK 条件

- 未验证 Wannier bands 是否复现目标能区的 QE bands。
- 未记录 projection、frozen window 或 disentanglement window。
- `pw2wannier90.x` 接口文件来源和 QE `prefix/outdir` 不可追踪。
- SOC/spinor 数据链与上游 QE 设置不一致。

## 与本仓库页面的对应关系

- [workflows/advanced/wannier90-overview.md](../../workflows/advanced/wannier90-overview.md)
- [workflows/electronic/bands.md](../../workflows/electronic/bands.md)
- [physics-judgement/wannier-validation-and-window-choice.md](../../physics-judgement/wannier-validation-and-window-choice.md)

## 局限

Wannier90 是高级电子结构工具，不替代 QE 基础 bands/DOS 学习。

## 原始链接

- [Wannier90](https://wannier.org/)
- [Wannier90 documentation](https://wannier90.readthedocs.io/)
- [QE INPUT_pw2wannier90](https://www.quantum-espresso.org/Doc/INPUT_pw2wannier90.html)
