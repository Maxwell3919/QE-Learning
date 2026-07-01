# QE Learning

QE-Learning 是面向科研计算的 Quantum ESPRESSO（QE）中文参考手册与 DFT / DFPT / 固体计算知识体系。它用于长期查阅 QE 原生命令行 workflow、input/output 阅读、数值收敛、后处理、理论边界、结果可靠性判断和科研记录规范。

## 内容范围

```text
learn/              按能力目标、完成判据和可验证输出组织的学习路径
workflows/          QE 原生命令行 workflow 的输入、输出和可靠性判断
theory-minimum/     使用 QE 所需的最低理论背景
physics-judgement/  DFT/QE 计算判断所需的进阶物理理论背景
references/         官方文档、教程网站和辅助工具导读
standards/          文件命名、记录、引用和 output review 规范
```

## 学习主线

```text
structure + pseudopotential
  -> SCF
  -> convergence
  -> relaxation
  -> electronic structure
  -> phonon / DFPT
  -> post-processing
  -> reproducibility
```

## 使用方式

需要按问题快速查阅时，先看 [HANDBOOK_INDEX.md](HANDBOOK_INDEX.md)。从 [learn/00-start-here.md](learn/00-start-here.md) 开始，先建立 QE 原生命令行 workflow 的整体图景，再进入 [workflows/](workflows/) 阅读具体计算步骤。遇到最低理论概念时查 [theory-minimum/](theory-minimum/)，需要解释物理近似和失败模式时查 [physics-judgement/](physics-judgement/)，需要外部资料时查 [references/](references/)，每次学习或计算记录按 [standards/](standards/) 执行。读完页面只代表完成阅读，是否掌握需要由 output review、收敛记录和 `PASS / WARN / BLOCK` 判断来证明。

v0.2 内容完成版补充了跨目录导航、原创 SVG 学习图、批次 QA 报告和发布说明。需要了解当前手册完成范围、验证方式和已知限制时，阅读 [RELEASE_NOTES_v0.2.md](RELEASE_NOTES_v0.2.md)；需要查图像资源时，阅读 [assets/diagrams/README.md](assets/diagrams/README.md)。

## 边界说明

本仓库保存通用参考手册、学习路线、审阅框架和记录规范；个人计算的完整 input/output、scratch、restart 和大文件应留在个人项目或计算环境中。QE 参数定义和程序行为仍以 QE 官方文档和 `INPUT_*` reference 为准，本仓库负责中文导读和科研判断框架。

结构构建、CIF 操作、对称性标准化、超胞、缺陷和 slab 构建会在独立 Structure-Learning 项目中展开；本仓库只说明 QE workflow 对结构输入的要求。

AiiDA、AiiDAlab 等自动化工具放在高级参考中，用于理解 workflow/provenance，不作为基础学习路径。
