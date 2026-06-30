# Electronic Structure Loop

## 我为什么要学这个？

bands、DOS、PDOS 是最常见的电子结构输出，但它们依赖不同 k 空间采样，不能混为一个 workflow。

## 它在 QE workflow 中处于哪个位置？

```text
trusted SCF
  ├─> pw.x bands on k-path -> bands.x
  └─> pw.x nscf on dense mesh -> dos.x / projwfc.x
```

## 我需要读哪些外部资料？

- QE INPUT_BANDS
- Pranab bands / DOS / PDOS tutorials
- SeeK-path documentation
- AiiDA-QE PwBandsWorkChain

## 我需要跑什么最小案例？

Si：bands + DOS。GaAs：bands + PDOS。

## output 应该怎么看？

- k-path 标签是否匹配结构标准化。
- DOS mesh 是否是 uniform dense mesh。
- Fermi level / VBM / CBM 参考是否一致。
- `nbnd` 是否覆盖目标能量范围。
- PDOS 投影标签和轨道贡献是否可解释。

## 结果什么时候可以进入下游？

当 SCF 来源、k-path 来源、DOS k-mesh、能量零点和图像脚本都可追踪后，可以进入分析或 Wannier 等高级分支。

## 当前页面还缺哪些验证？

- 还缺标准 band plot / DOS plot 示例。
- 还缺 seekpath 生成路径与 QE input 对照。
