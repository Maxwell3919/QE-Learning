# K-point convergence workflow

## 1. 计算目标

确定 k 点采样对总能、力、应力、Fermi level、DOS 或 phonon 前置状态足够收敛。

## 2. 输入前提

结构、赝势、cutoff 和 smearing 策略固定。

## 3. 计算图

```text
fixed structure + cutoff policy
  -> series of pw.x scf with k meshes
  -> convergence table
  -> chosen k mesh
```

## 后续扩展

补金属、低维体系和 DOS/phonon 场景的不同判据。

