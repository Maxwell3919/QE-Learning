# References

`references/` 用于把 QE 官方文档、教程网站和相关工具文档转成学习导读。这里不保存原文摘录，也不把外部资料改写成独立教程；每一页只说明该资料适合回答什么问题、应该怎样读、哪些结论需要回到原始来源核对。

本目录的重点是帮助读者在学习 QE 原生命令行 workflow 时选择资料，而不是堆链接。正文优先写阅读顺序、适用边界和核对方式；具体原始链接放在各导读页末尾，完整来源索引放在 [source-index.md](source-index.md)。

## 使用方式

- 查参数定义、默认值、输入卡片和程序说明：先读 [official-qe-docs.md](tutorial-sites/official-qe-docs.md)，再回到 QE 官方 `INPUT_*` reference 或 package guide 核对。
- 查 hands-on workflow 拆解：读 `tutorial-sites/` 下的教程导读，用它们理解命令顺序、文件关系和常见输出判断。
- 查辅助工具边界：读 `tools/` 下对应页面，确认该工具解决的是结构处理、路径生成、后处理、可视化、workflow provenance 还是高级耦合计算问题。
- 查论文级物理判断来源：读 [canonical-literature.md](canonical-literature.md)。该页维护 A0/A1/A2/B1/B2/C 分级和 Fast Core Loop；[canonical.bib](canonical.bib) 只作为 BibTeX 起点。
- 查文献维护规则：读 [reading-maintenance-policy.md](reading-maintenance-policy.md)，确认新增来源是否真的改变 input/output review 或 `PASS / WARN / BLOCK` 判断。

## 资料选择原则

- `official docs`：用于定义性问题，包括输入参数、程序行为、文件格式、package guide 和版本相关限制。凡是涉及参数含义、默认值或可执行语法，最终以官方文档为准。
- `tutorial-sites`：用于学习流程拆解和输出阅读。教程可以帮助建立操作顺序和判断习惯，但不能替代官方文档中的参数定义。
- `tools`：用于理解辅助工具在 QE workflow 中的位置和边界。工具页不是推荐榜，也不按流行度排序；排序只服务学习路径和主题归类。

## 工具页边界

工具页只回答三个问题：它在 QE workflow 中能帮助什么、它不能替代什么、使用时需要回到哪些原始文档核对。工具页不把工具包装成必选路线，也不把自动化工具放进基础学习路径。

AiiDA 保持 optional advanced reference：它适合用来理解 workflow/provenance 和自动化思想，但不是本仓库入门阶段的必读内容。

## 原始链接

- [Quantum ESPRESSO documentation](https://www.quantum-espresso.org/documentation/)
- [QE official learning resources](https://www.quantum-espresso.org/resources/)
- [references/source-index.md](source-index.md)
- [references/canonical-literature.md](canonical-literature.md)
- [references/reading-maintenance-policy.md](reading-maintenance-policy.md)
