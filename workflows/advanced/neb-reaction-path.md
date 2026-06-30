# NEB Reaction Path Workflow

## 计算目标

NEB 用于计算两个已知端点结构之间的最小能量路径和能垒。它属于势能面和反应路径 workflow，不是 SCF 或 relax 的简单后处理。

## QE 程序 / 外部工具

`neb.x` / PWneb。

## 输入前提

- 初态和终态结构必须分别可信。
- 反应路径 images 的构造必须物理合理。
- 每个 image 的原子对应关系必须清楚。
- SCF、cutoff、k-points、smearing 策略应与目标体系匹配。

## 结构学习边界

NEB 强依赖路径结构构造；结构操作学习将作为独立项目展开，本页只说明 QE 对路径输入的要求。

## output 应该怎么看

- path 是否收敛。
- force along path 如何判断。
- barrier 是否对 image 数量收敛。
- 是否需要 climbing image。
- 如何记录每个 image 的结构来源。

## 推荐参考资料

- QE official documentation: https://www.quantum-espresso.org/documentation/
