# QE Learning

这是一个面向计算材料科研实践的 Quantum ESPRESSO（QE）中文学习笔记仓库，目标是帮助学习者从 input/output、workflow 和数值可靠性角度系统掌握 QE。

## 仓库不是什么

本仓库不是工具大全、官方文档翻译、input 文件合集、AiiDA 教程、高通量 workflow 教程、真实计算案例仓库，也不是结构操作学习仓库。

## 仓库提供什么

- QE 使用流程学习笔记
- QE 功能地图
- QE workflow 精细解释
- QE input/output 阅读笔记
- QE 数值可靠性判断笔记
- QE 官方文档和优秀教程的中文导读
- 长期维护的科研计算规范文档

## 主目录

```text
learn/              学习路径：按能力目标、完成判据和可验证输出组织
workflows/          QE 原生命令行 workflow 的输入、输出和可靠性判断
theory-minimum/     使用 QE 必须补的最低理论背景
references/         官方文档、优秀教程和辅助工具的中文导读
standards/          命名、记录、引用、output review 和 PASS/WARN/BLOCK 规范
```

## 学习顺序

1. 从 [learn/00-start-here.md](learn/00-start-here.md) 开始，理解为什么要优先学习 QE 原生命令行 workflow。
2. 阅读 [learn/01-qe-workflow-map.md](learn/01-qe-workflow-map.md)，建立 QE 主计算图。
3. 按 `SCF -> convergence -> relax -> electronic structure -> phonon -> postprocessing -> reproducibility` 的顺序推进。
4. 每进入一个 workflow，都同时阅读对应的 `standards/` 页面，留下可追溯记录。

## Structure-Learning 边界

材料结构操作、CIF 处理、对称性标准化、超胞、缺陷、slab 和低维结构构建会在独立的 Structure-Learning 项目中展开，本仓库只在 QE workflow 中说明结构输入前提。

## 外部工具边界

本仓库优先学习 QE 原生命令行 workflow。ASE、pymatgen、SeeK-path、phonopy、Wannier90、AiiDA 等工具只作为辅助参考，不替代 QE 基础学习。AiiDA / AiiDA-QE 属于高级 workflow/provenance 参考，不作为 QE 入门主线。

## 主要参考来源

本仓库参考 QE 官方文档、Pranab Das QE tutorial、Kyoto University QE phonon DokuWiki、Phonopy QE interface、Wannier90、ASE、pymatgen、SeeK-path 等公开资料。来源索引见 [references/source-index.md](references/source-index.md)。
