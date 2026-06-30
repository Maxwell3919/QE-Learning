# Citation and Source Policy

## 1. 为什么需要这个标准？

QE 笔记如果没有来源边界，很容易把官方参数定义、教程经验、个人推断和未验证说法混在一起。本仓库要求每个可执行知识点都可追踪。

## 2. 外部教程引用

每篇参考外部教程的网站页必须保留原始链接，并说明：

- 它是什么。
- 适合跟做什么。
- 只适合作为参考的部分。
- 它覆盖哪些本仓库 workflow。
- 它的局限。

## 3. input 模板来源

每个 input 模板必须说明来源：

- 官方文档直接改写。
- Pranab/Kyoto/Materials Cloud 等教程改写。
- 本仓库真实案例验证。
- 尚未验证，只作为结构占位。

不得出现“我记得”“网上常见说法”“一般都这样”这类无法追踪来源。

## 4. 案例来源

每个案例必须说明：

- QE 版本。
- 结构来源。
- 赝势来源。
- input 文件来源。
- command / job script。
- environment。
- output 摘要。
- PASS/WARN/BLOCK 状态。

## 5. 页面末尾来源

如果一个知识点来自 Pranab、Kyoto、官方文档、AiiDA、Phonopy、ASE 或其他工具文档，页面末尾必须列出来源链接。

## 6. 版本边界

QE 官方 input reference 会随版本变化。页面若引用参数默认值、限制或新功能，必须标注核对来源和版本；本轮核对的官方 input reference 页面显示为 QE 7.5。
