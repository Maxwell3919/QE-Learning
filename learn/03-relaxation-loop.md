# Relaxation Loop

## 我为什么要学这个？

许多性质依赖平衡结构。未 relax 的结构直接进入 phonon，可能把结构未平衡造成的虚频误判为真实不稳定。

## 它在 QE workflow 中处于哪个位置？

```text
SCF policy + initial structure
  -> pw.x relax / vc-relax
  -> final structure
  -> final static SCF
  -> electronic / phonon workflows
```

## 我需要读哪些外部资料？

- QE INPUT_PW: `&IONS`, `&CELL`
- Pranab structure optimization tutorial

## 我需要跑什么最小案例？

Si relax；随后做 final static SCF。

## output 应该怎么看？

- 每个 ionic step 的 SCF 是否稳定。
- final forces 是否满足要求。
- vc-relax 时 stress/cell 是否合理。
- final coordinates/cell 是否已保存并重新用于 static SCF。

## 结果什么时候可以进入下游？

当最终结构、残余力、应力、cell constraint 和 final SCF 都记录清楚后，才进入 bands、DOS 或 phonon。

## structure-learning 边界

结构操作细节，如 primitive/conventional cell、slab、vacuum、defect、strain，不在本页展开，见 [structure-learning/README.md](../structure-learning/README.md)。

## 当前页面还缺哪些验证？

- 还缺 relax/vc-relax output 示例。
- 还缺低维 `cell_dofree` 案例。
