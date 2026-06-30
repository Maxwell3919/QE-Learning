# Relax workflow

## 1. 计算目标

优化原子位置，使残余力达到目标阈值。

## 3. 计算图

```text
initial structure + SCF policy
  -> pw.x relax
  -> relaxed positions
  -> final pw.x scf
```

## 后续扩展

补 force 判据、BFGS/FIRE、restart 和常见失败。

