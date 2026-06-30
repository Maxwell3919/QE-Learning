# Source Traceability

## 1. 追踪什么？

- 结构来源。
- 赝势来源。
- input 模板来源。
- 参数选择理由。
- 命令和运行环境。
- output 文件和图像生成脚本。

## 2. 为什么？

QE workflow 通过 `prefix/outdir` 和 side-effect 文件传递状态。没有 source traceability，就无法判断下游 bands、DOS、phonon 是否读到了正确的上游数据。

## 3. 最小记录字段

```markdown
- Structure source:
- Pseudopotential source:
- Input source:
- Upstream calculation:
- prefix:
- outdir:
- Command:
- QE version:
- Output files:
```
