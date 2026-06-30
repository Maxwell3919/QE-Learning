# Start Here

## 为什么先读这一页？

QE 学习不是先背参数，而是先形成计算闭环。一个闭环至少包括：明确物理目标，构造 input，运行程序，读 output，判断是否可信，记录结果和下一步。

如果只复制 tutorial input，你可能得到一个 `JOB DONE`，但仍然不知道赝势是否合适、cutoff 是否收敛、k 点是否足够、结构是否稳定，也不知道结果能否进入 bands、DOS 或 phonon。

## 本仓库学习顺序

1. 建立资料入口：官方文档、input reference、Pranab、Kyoto、AiiDA、Phonopy。
2. 完成第一个 SCF 闭环。
3. 完成 cutoff、ecutrho、k-point、smearing 收敛闭环。
4. 完成 relax / vc-relax 判断闭环。
5. 完成电子结构闭环：SCF -> bands，SCF -> NSCF -> DOS/PDOS。
6. 完成 DFPT phonon 闭环：SCF -> ph.x -> q2r.x -> matdyn.x。
7. 用 `standards/` 把每次计算记录为可追溯项目。

## 外部资料怎么读？

- 官方文档用于参数定义和程序边界，不适合直接当初学主线。
- Pranab Das QE tutorial 适合跟做 SCF、收敛、relax、bands、DOS、PDOS、phonon 等案例。
- Kyoto phonon DokuWiki 适合补充 phonon 的专家判断，尤其 Gamma、ASR、`epsil`、金属/绝缘体差异。
- AiiDA-QE 适合理解 workflow graph、provenance 和 restart，但不建议作为第一步。

## 结构学习的边界

结构文件读取、primitive/conventional cell、space group、Wyckoff、supercell、slab、vacuum、defect、strain、k-path 标准化等内容会在 [structure-learning/README.md](../structure-learning/README.md) 单独展开。本仓库的 QE workflow 页面只要求结构已经被检查，并在必要处指向 `structure-learning/`。

## 如何使用 cases 和 standards？

- `cases/` 用来放最小可复现案例。当前先建立案例入口，后续逐步补 input/output。
- `standards/` 用来规范文件命名、记录模板、引用策略、source traceability 和 output review。

## 进入下一阶段的判断

不要用时间判断学习进度。只有当你能留下可审查输出，才进入下一阶段：

- 能解释 input 参数。
- 能指出 output 中关键判断位置。
- 能说明下游 workflow 依赖什么文件和前提。
- 能把结果标成 PASS/WARN/BLOCK。
- 能说明当前页面或案例还缺哪些验证。
