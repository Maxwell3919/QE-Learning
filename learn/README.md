# Learn

`learn/` 是快速进入 QE 的操作路线，不是理论目录，也不是工具目录。每一页都围绕一个能力闭环：

- 为什么要学这个？
- 它在 QE workflow 中处于哪个位置？
- 需要读哪些外部资料？
- 需要跑什么最小案例？
- output 应该怎么看？
- 结果什么时候可以进入下游？
- 当前页面还缺哪些验证？

## 能力阶段

| 阶段 | 能力目标 | 完成判据 | 可验证输出 |
|---|---|---|---|
| Stage 0 | 环境与资料入口 | 能找到 QE 官方文档、input reference、Pranab/Kyoto 教程和本仓库主线 | source list + 学习入口记录 |
| Stage 1 | 第一个可信 SCF 闭环 | 能解释 SCF input 和 output 的关键字段 | `pw.scf.*.in/out` + `record.md` |
| Stage 2 | 数值收敛闭环 | 能完成 cutoff、ecutrho、k-point、smearing 的最小收敛测试 | convergence report |
| Stage 3 | 结构优化闭环 | 能判断 relax/vc-relax 的力、应力和晶胞约束 | relax record + final SCF |
| Stage 4 | 电子结构闭环 | 能完成 bands、DOS、PDOS 并解释 k-path/DOS mesh 差异 | bands/DOS/PDOS notes |
| Stage 5 | DFPT phonon 闭环 | 能完成 `ph.x -> q2r.x -> matdyn.x` 并判断 ASR/虚频 | phonon record |
| Stage 6 | 项目记录闭环 | 每次计算都有 provenance 和 PASS/WARN/BLOCK | project record tree |
