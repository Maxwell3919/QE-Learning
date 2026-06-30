
# Yambo

## 它适合解决什么学习问题？

Yambo 是 ground-state DFT 之外的 many-body perturbation theory 和 excited-state workflow 入口。它适合在学习者已经理解 SCF、NSCF、bands、DOS、GW/BSE 边界后，用来认识 quasiparticle correction、optical response 和 excitonic effect 的计算路径。

## 覆盖哪些 QE workflow？

- `workflows/electronic/bands.md`：提供 ground-state electronic structure 前置图像。
- `workflows/advanced/hybrid-functional-overview.md`：说明 ordinary DFT 与高级电子结构方法的边界。
- `physics-judgement/ground-state-vs-excited-state.md`：说明为什么普通 KS bands 不能替代 quasiparticle / optical spectra。

## 适合跟读的部分

- QE 到 Yambo 的数据接口说明。
- GW/BSE 基本 tutorial 的输入输出关系。
- convergence 参数、空带、介电矩阵、k mesh 和 cutoff 的独立审阅方式。

## 只适合作为参考的部分

完整 GW/BSE 生产 workflow、具体材料光谱拟合、并行调优和大规模计算策略只作为高级专题入口，不进入本仓库基础路线。

## 与本仓库页面的对应关系

- [physics-judgement/ground-state-vs-excited-state.md](../../physics-judgement/ground-state-vs-excited-state.md)
- [physics-judgement/kohn-sham-eigenvalue-boundary.md](../../physics-judgement/kohn-sham-eigenvalue-boundary.md)
- [learn/09-feature-expansion-map.md](../../learn/09-feature-expansion-map.md)

## 局限

Yambo 引入独立的文件转换、many-body convergence 参数和光谱解释框架。它不能替代 QE 基础 workflow 学习，也不能自动修复 ground-state 上游质量问题。

## 原始链接

- [Yambo educational wiki](https://wiki.yambo-code.eu/wiki/index.php?title=Main_Page)
- [Yambo learn](https://www.yambo-code.eu/learn/)
- [Yambo project](https://www.yambo-code.eu/)
