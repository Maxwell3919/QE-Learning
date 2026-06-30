# AiiDA QE Tutorials

## 1. 它是什么？

AiiDA-QE tutorials 是 `aiida-quantumespresso` 的教程入口，展示如何把 QE calculations 组织成 provenance-aware workflows。

## 2. 适合学习或解决什么问题？

适合学习 workflow graph、typed inputs/outputs、restart、scheduler 和 protocol。

## 3. 它覆盖哪些 QE workflow？

SCF、bands、relax、DOS/PDOS、PHonon 相关 workchains，具体以 AiiDA-QE docs 为准。

## 4. 哪些部分适合跟做？

完成手动 QE 基础闭环后，跟做 `PwBandsWorkChain` 是理解 graph 化 bands workflow 的好入口。

## 5. 哪些部分只适合作为参考？

daemon、profile、database、remote computer 设置对初学 QE 不是第一优先。

## 6. 它和本仓库哪些页面对应？

- `learn/07-hpc-record-provenance-loop.md`
- `references/tools/aiida-qe.md`

## 7. 它的局限是什么？

AiiDA 提供 workflow 记忆，不替代物理判断和收敛验证。

## 8. 原始链接与引用

- [AiiDA-QE tutorials](https://aiida-quantumespresso.readthedocs.io/en/latest/tutorials/)
- [PwBandsWorkChain](https://aiida-quantumespresso.readthedocs.io/en/latest/howto/run_pwbands.html)
