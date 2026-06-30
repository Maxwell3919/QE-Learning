# Phonon DFPT Loop

## 我为什么要学这个？

phonon 不是普通后处理。`ph.x` 是 DFPT 响应计算，依赖高质量 SCF、充分 relax 的结构、更严格的收敛要求和清楚的 q-space 设计。

## 它在 QE workflow 中处于哪个位置？

```text
relaxed structure + strict SCF
  -> ph.x
  -> q2r.x
  -> matdyn.x
  -> phonon dispersion / DOS / mode analysis
```

## 我需要读哪些外部资料？

- [Pranab phonon tutorial](https://pranabdas.github.io/espresso/hands-on/phonon/)
- [Kyoto phonon DokuWiki](../references/tutorial-sites/kyoto-phonon-dokuwiki.md)
- QE INPUT_PH、INPUT_Q2R、INPUT_MATDYN

## 我需要跑什么最小案例？

Si phonon dispersion；GaAs polar phonon / Born charge branch 作为后续案例。

## output 应该怎么看？

- `ph.x` 每个 q-point / perturbation 是否收敛。
- `q2r.x` 是否读取完整 dyn 文件。
- `matdyn.x` 是否生成合理频率文件。
- Gamma 附近 acoustic branch 是否合理。
- negative frequency 是数值误差、待复查还是物理不稳定。

## 结果什么时候可以进入下游？

当结构、SCF、q-grid、ASR、negative frequency 判断和 phonon output 都记录为 PASS/WARN/BLOCK 后，才进入 phonon DOS、IR/Raman、EPC 或热学分析。

## 当前页面还缺哪些验证？

- 还缺真实 phonon output 片段。
- 还缺 GaAs `epsil` / Born charge 示例。
