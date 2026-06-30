# PASS / WARN / BLOCK

## 1. 为什么需要这个标准？

QE output 里出现 `JOB DONE` 只说明程序结束，不说明结果可以进入下游。PASS/WARN/BLOCK 用来把“运行结束”转化为“科学计算状态”。

## 2. 状态定义

| 状态 | 含义 | 是否允许下游 |
|---|---|---|
| PASS | 程序完成，关键阈值满足，来源和环境可追踪，当前目标所需收敛证据足够 | 允许 |
| WARN | 程序完成，但存在明确风险；可用于探索，不应用作最终结论 | 视下游风险决定 |
| BLOCK | 当前结果不能支撑下游；需要重算、修复输入或重新设计 workflow | 不允许 |

## 3. 常见 PASS 条件

- input 来源清楚。
- structure、pseudopotential、cutoff、k-points、smearing 有记录。
- output 中关键 convergence 达标。
- 没有未解释的 warning。
- `record.md` 完整。

## 4. 常见 WARN 条件

- 参数只做了初步测试。
- phonon 有小负频但尚未完成收敛排查。
- 结构来源可信但残余力偏高。
- HPC restart 发生过，但结果可追踪。

## 5. 常见 BLOCK 条件

- SCF 未收敛。
- `prefix/outdir` 混用旧数据。
- 赝势来源不明或泛函混用。
- phonon 输入结构未 relax。
- output 与 input/command 对不上。

## 6. GitHub 记录要求

每个 `record.md` 必须写状态和理由：

```markdown
## Convergence status

- Status: PASS / WARN / BLOCK
- Reason:
- Evidence:
- Downstream allowed:
```
