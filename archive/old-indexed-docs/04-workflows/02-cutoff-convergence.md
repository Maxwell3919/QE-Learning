# Cutoff convergence workflow

## 1. 计算目标

确定 `ecutwfc` 和 `ecutrho` 对目标性质已经足够收敛。

## 2. 输入前提

结构、赝势、k 点和 occupation 策略固定。

## 3. 计算图

```text
fixed structure + pseudos
  -> series of pw.x scf with different cutoffs
  -> convergence table
  -> selected cutoff policy
```

## 后续扩展

补输入模板、能量/力/应力判据和报告示例。

