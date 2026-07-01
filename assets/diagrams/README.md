# Diagram Assets

本目录保存 QE-Learning 自制学习图。图像用于解释 workflow、理论对象或判断边界，不作为装饰，也不替代正文中的 output review 和来源说明。

## 命名规则

- `workflows/`：QE 程序链、文件链和下游准入图。
- `theory-minimum/`：最低理论对象和 QE input/output 映射图。
- `physics-judgement/`：可信度判断、误差来源和 `PASS / WARN / BLOCK` gate 图。

文件名使用小写英文和连字符，优先保存为 SVG。

## 来源边界

- 所有图均为本仓库原创示意图。
- 不复制外部论文、教程、网页或软件手册中的图片。
- 图中不写具体材料案例、不写固定参数推荐、不暗示某个方法普适更准。
- 每张图必须在至少一个 Markdown 页面中引用；未引用的图不提交。

## Markdown 引用方式

```markdown
![QE ground-state workflow chain](workflows/qe-ground-state-chain.svg)
```

图下方应写一句 caption，说明该图支持什么学习目标，以及它不能替代哪些 output review。
