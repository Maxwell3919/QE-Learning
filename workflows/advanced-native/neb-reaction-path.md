# NEB Reaction Path Workflow

## 计算目标

NEB 用于计算两个已知端点结构之间的最小能量路径和能垒。它属于势能面和反应路径 workflow，不是 SCF 或 relax 的简单后处理。

## QE workflow 位置

```text
initial state + final state + intermediate images
  -> neb.x / PWneb workflow
  -> energy profile
  -> barrier analysis
```

## 输入前提

- 初态和终态结构必须分别可信。
- 反应路径 images 的构造必须物理合理。
- 每个 image 的原子对应关系必须清楚。
- SCF、cutoff、k-points、smearing 策略应与目标体系匹配。

## structure-learning 边界

NEB 强依赖路径结构构造，但路径构造、端点匹配、image interpolation、缺陷/扩散路径设计不在本页展开。后续在 [structure-learning/README.md](../../structure-learning/README.md) 单独讨论。

## 输出判断

当前页面只建立边界。后续补充时必须说明：

- path 是否收敛。
- force along path 如何判断。
- barrier 是否对 image 数量收敛。
- 是否需要 climbing image。
- 如何记录每个 image 的结构来源。

## 参考

- QE official documentation: <https://www.quantum-espresso.org/documentation/>
