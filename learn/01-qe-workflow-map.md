# QE Workflow Map

## 主计算图

```text
structure + pseudopotential
  -> pw.x scf
  -> convergence
  -> relax / vc-relax
  -> electronic: bands / DOS / PDOS / pp.x
  -> phonon: ph.x / q2r.x / matdyn.x / dynmat.x
  -> advanced: SOC / DFT+U / Wannier / EPC / MD / NEB
```

## 核心状态

- structure：晶胞、原子、边界条件和坐标。
- pseudopotential：元素、泛函、赝势类型和 cutoff 建议。
- ground state：`pw.x scf` 产生的电荷密度、势、波函数和总能。
- convergence policy：cutoff、k 点、smearing、SCF 阈值和下游准入。
- optimized structure：`relax` 或 `vc-relax` 后的结构和 final static SCF。
- electronic observables：bands、DOS、PDOS、charge density、potential、ELF、work function。
- response observables：phonon、dielectric tensor、Born charge、IR/Raman、EPC。

## 学习原则

每条边都要能回答三个问题：上游文件是什么，下游程序读什么，output 如何判断是否可信。
