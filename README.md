# QE Learning

面向 Quantum ESPRESSO（QE）科研计算的中文参考手册与 DFT 学习知识库，内容围绕原生命令行 workflow、input/output 阅读、数值收敛、后处理、DFT 理论边界和结果可靠性判断组织。

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

从 [learn/00-start-here.md](learn/00-start-here.md) 开始，先建立 QE 原生命令行 workflow 的整体图景，再进入 [workflows/](workflows/) 阅读具体计算步骤。遇到最低理论概念时查 [theory-minimum/](theory-minimum/)，需要解释物理近似和失败模式时查 [physics-judgement/](physics-judgement/)，需要外部资料时查 [references/](references/)，每次学习或计算记录按 [standards/](standards/) 执行。

## 边界说明

结构构建、CIF 操作、对称性标准化、超胞、缺陷和 slab 构建会在独立 Structure-Learning 项目中展开；本仓库只说明 QE workflow 对结构输入的要求。

AiiDA、AiiDAlab 等自动化工具放在高级参考中，用于理解 workflow/provenance，不作为基础学习路径。
