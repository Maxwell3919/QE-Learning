# Standards

`standards/` 是本仓库的规范入口，用来把 QE 学习笔记和个人科研计算记录从“能跑出文件”推进到“能复查、能迁移、能判断下游是否可信”。它服务两个场景：

- 个人学习：把 input、output、命令、参数来源和 review 结果写成可回看的记录，避免只记结论或只保存截图。
- 科研计算：把结构、赝势、环境、`prefix/outdir`、收敛证据和下游准入状态分开记录，方便之后复算、排错、汇总和写作。

本仓库不要求保存具体材料的 input/output 文件，也不要求把 scratch、wavefunction、restart 或大型中间文件纳入仓库。具体计算文件可以放在个人项目目录、集群工作目录或归档位置；本仓库只规定这些文件应如何命名、记录、审阅和引用。

## 规范页面

- [input-file-naming.md](input-file-naming.md)：解决 QE 文件链混乱的问题。它规定 input、output 和后处理文件名应表达程序、计算阶段、体系标识和文件类型，并区分人类可读文件名与 QE 程序读取的 `prefix/outdir`。
- [calculation-record-template.md](calculation-record-template.md)：解决计算记录不完整的问题。它给出每次计算至少应记录的结构来源、赝势来源、输入变化、运行命令、环境、output 摘要、收敛状态、问题证据和下一步动作。
- [project-layout.md](project-layout.md)：解决个人项目文件散落的问题。它建议把结构、赝势来源、inputs、outputs、scripts、records、figures 和 notes 分开，便于复查、迁移和归档；这是个人计算项目布局，不是本仓库目录结构。
- [pass-warn-block.md](pass-warn-block.md)：解决“JOB DONE 等于结果可用”的误判。它把 QE 运行结束后的状态分成 `PASS`、`WARN`、`BLOCK`，要求记录理由、证据和是否允许进入下游 workflow。
- [output-review-checklist.md](output-review-checklist.md)：解决只看最后几行 output 的问题。它按 SCF、relax/vc-relax、bands/DOS/PDOS、phonon 等 workflow 列出最小 review 项，要求从 output 中核对 convergence、关键数值、warning 和文件链。
- [citation-and-source-policy.md](citation-and-source-policy.md)：解决来源边界不清的问题。它要求区分官方参数定义、公开教程、论文、工具文档、个人推断和未验证经验；引用参数默认值、限制或新功能时必须能回到公开来源和版本边界。

## 维护规则

新增页面或修改页面前，先做三件事：

1. 检查来源：确认内容来自官方文档、公开教程、论文、工具文档、QE output 证据或本仓库已有规范；不要写无法追踪的经验断言。
2. 检查禁用词：避免使用“我记得”“网上常见说法”“一般都这样”等没有来源边界的表述；如果是个人判断，必须写明依据和适用范围。
3. 检查链接：只保留必要链接，优先链接到本仓库对应规范页或公开原始来源；不要把页面写成链接堆，也不要引用本地私有路径。
