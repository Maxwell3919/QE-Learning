# AiiDA Quantum ESPRESSO

## 1. 它是什么？

AiiDA-QE 是 AiiDA 的 Quantum ESPRESSO plugin，把 QE 计算封装为带 provenance、restart 和 scheduler 语义的 calculations/workchains。

## 2. 适合学习或解决什么问题？

适合把 shell 脚本串联升级为可查询的 workflow graph，解决 provenance、restart、scheduler、批量运行和协议化默认值问题。

## 3. 它覆盖哪些 QE workflow？

SCF、relax、bands、DOS/PDOS、phonon、q2r、matdyn 等多个 QE 分支。

## 4. 哪些部分适合跟做？

在完成手动 SCF/bands/phonon 基础后，跟做 AiiDA-QE tutorials 和 `PwBandsWorkChain`。

## 5. 哪些部分只适合作为参考？

AiiDA 不适合作为第一个 QE 学习入口；它会引入数据库、daemon、scheduler 等额外复杂度。

## 6. 它和本仓库哪些页面对应？

- `learn/07-hpc-record-provenance-loop.md`
- `workflows/electronic/bands.md`
- `standards/source-traceability.md`

## 7. 它的局限是什么？

它提升 workflow 可靠性，但不会替代物理判断。错误赝势、未收敛参数或错误结构仍会产生错误结果。

## 8. 原始链接与引用

- [AiiDA-QE tutorials](https://aiida-quantumespresso.readthedocs.io/en/latest/tutorials/)
- [AiiDA-QE PwBandsWorkChain](https://aiida-quantumespresso.readthedocs.io/en/latest/howto/run_pwbands.html)
- [AiiDA-QE workflow logic](https://aiida-quantumespresso.readthedocs.io/en/latest/topics/workflow_logic.html)
- [AiiDA-QE protocols](https://aiida-quantumespresso.readthedocs.io/en/latest/topics/protocol.html)
