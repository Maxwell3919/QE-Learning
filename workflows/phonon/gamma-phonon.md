# Gamma Phonon Workflow

## 计算目标

计算 Gamma 点声子、模式和特定响应分支。

## QE 程序 / 外部工具

`ph.x`, `dynmat.x`

## 输入前提

- 已有可信上游计算或清楚的输入来源。
- 赝势、cutoff、k 点和相关物理设置已记录。
- 后续完善重点：补充 output 阅读样例和 PASS/WARN/BLOCK 判断。

## output 应该怎么看

- 检查程序是否正常结束。
- 检查关键数值是否达到当前 workflow 的准入要求。
- 检查 warning 是否已解释。
- 用 `standards/pass-warn-block.md` 给出 PASS/WARN/BLOCK 判断。

## 推荐参考资料

- QE INPUT_PH, Kyoto phonon DokuWiki
