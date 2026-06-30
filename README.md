# QE Learning

这是一个面向计算材料科研实践的中文 Quantum ESPRESSO（QE）学习笔记仓库。它的目标不是收集零散教程链接，也不是把 input 文件堆在一起，而是把 QE 的真实计算流程整理成可执行、可复习、可追溯、可扩展的学习体系。

本仓库当前采用四条主线：

- `learn/`：按能力闭环组织的快速学习路径。
- `workflows/`：稳定的 QE workflow 规范，强调输入前提、计算图、output 判断和下游准入。
- `references/`：外部教程、官方文档和工具生态的学习资产库。
- `standards/`：命名、记录、引用、PASS/WARN/BLOCK 和 output review 标准。

旧的编号式文档已经归档到 `archive/old-indexed-docs/`，不再作为主要入口。

## 如何开始

从这里读：

1. [learn/00-start-here.md](learn/00-start-here.md)
2. [learn/01-first-scf-loop.md](learn/01-first-scf-loop.md)
3. [standards/pass-warn-block.md](standards/pass-warn-block.md)
4. [workflows/ground-state/scf.md](workflows/ground-state/scf.md)
5. [references/source-index.md](references/source-index.md)

学习路线不按时间承诺组织，而按“能力目标 + 完成判据 + 可验证输出”组织。只有当你能解释 input、读懂 output、记录 provenance，并判断结果是否能进入下游 workflow，才算完成一个学习闭环。

## 当前结构

```text
QE-Learning/
  learn/
  workflows/
  theory-minimum/
  references/
  cases/
  standards/
  structure-learning/
  archive/old-indexed-docs/
```

## 本轮资料基础

本轮重构优先读取两个 `.docx` 源文件：

- `/Users/paquette/Downloads/Quantum ESPRESSO Tutorial Website Ecosystem.docx`
- `/Users/paquette/Downloads/Quantum ESPRESSO as a Modern Research Stack.docx`

它们提供了教程网站、官方文档、QE 原生模块、外部工具生态、workflow 表示方式和学习路径的索引。完整来源整理见 [references/source-index.md](references/source-index.md)。

## 维护原则

- 每个知识点都尽量落到 QE input、output、workflow 或记录标准上。
- 每个外部教程页面必须保留原始链接，并说明适合跟做的部分与局限。
- 每个 input 模板必须说明来源：官方文档、教程改写或本仓库验证。
- 每个案例必须记录 QE 版本、结构来源、赝势来源、运行命令、环境和 PASS/WARN/BLOCK 状态。
- 结构操作学习单独放入 `structure-learning/`，QE workflow 页面只说明结构输入前提，不展开结构教学。
