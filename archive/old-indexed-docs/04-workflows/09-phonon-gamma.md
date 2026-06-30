# Gamma phonon workflow

## 1. 计算目标

在 Gamma 点计算 phonon modes、介电响应、Born effective charges 或 IR/Raman 相关量。

## 3. 计算图

```text
relaxed structure
  -> pw.x scf
  -> ph.x at q=0
  -> dynmat.x
```

## 后续扩展

重点区分金属/绝缘体、`epsil`、LO-TO、ASR 和传播方向。

