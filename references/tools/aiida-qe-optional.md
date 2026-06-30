# AiiDA-QE Optional Reference

## 1. 它是什么？

AiiDA-QE 是 AiiDA 的 Quantum ESPRESSO plugin，用于把 QE calculations 组织为带 provenance、restart 和 scheduler 语义的 workflow graph。

## 2. 适合学习什么？

它适合作为高级 workflow/provenance 参考，帮助理解如何记录计算图、输入输出、远程调度和 restart。

## 3. 覆盖哪些 QE workflow？

AiiDA-QE 可以封装 SCF、relax、bands、DOS/PDOS、PHonon 等分支，但这些不是 QE 入门主线。本仓库优先学习 QE 原生命令行 workflow。

## 4. 哪些部分适合跟做？

在已经能手动完成 SCF、收敛、bands/DOS 和 phonon 后，可以跟做 AiiDA-QE tutorials 或 `PwBandsWorkChain` 来理解 provenance graph。

## 5. 哪些部分只适合参考？

daemon、profile、database、remote computer、scheduler 集成等内容只适合作为自动化和可复现系统设计参考。

## 6. 它和本仓库哪些页面对应？

- `learn/08-hpc-and-reproducibility.md`
- `standards/calculation-record-template.md`
- `standards/citation-and-source-policy.md`

## 7. 局限

AiiDA 不替代物理判断、收敛测试和 output review。错误结构、错误赝势或未收敛参数仍会产生错误结果。

## 8. 原始链接

- [AiiDA-QE tutorials](https://aiida-quantumespresso.readthedocs.io/en/latest/tutorials/)
- [AiiDA-QE PwBandsWorkChain](https://aiida-quantumespresso.readthedocs.io/en/latest/howto/run_pwbands.html)
- [AiiDA-QE workflow logic](https://aiida-quantumespresso.readthedocs.io/en/latest/topics/workflow_logic.html)
- [AiiDA-QE protocols](https://aiida-quantumespresso.readthedocs.io/en/latest/topics/protocol.html)
