# Pranab Das QE Tutorial

## 1. 它是什么？

Pranab Das QE tutorial 是一个 GitHub Pages + GitHub repository 形式的 QE 教程系统。公开教程生态中可视为“modular workflow tutorial system”：左侧导航包含 Setup、Theory 和大量 Hands-on 分支。

## 2. 适合学习或解决什么问题？

它适合把 QE 初学者从单个 input 文件带到多个可跟做 workflow：SCF、convergence testing、structure optimization、DOS、bandstructure、PDOS、k-resolved DOS、graphene、magnetic systems、DFT+U、SOC、dielectric constants、Fermi surfaces、phonons、Wannier methods、molecular dynamics。

## 3. 它覆盖哪些 QE workflow？

- SCF：`pw.x`
- convergence：cutoff、k-points 等
- relax / structure optimization：`pw.x`
- bands：`pw.x` + `bands.x`
- DOS / PDOS：`dos.x`、`projwfc.x`
- phonon：`pw.x -> ph.x -> q2r.x -> matdyn.x`
- Wannier：QE 与 Wannier90 分支
- SOC、DFT+U、magnetism 等高级输入分支

## 4. 哪些部分适合跟做？

- SCF 页面：适合作为第一个可信 SCF 闭环的跟做材料。
- convergence 和 structure optimization 页面：适合补全 Stage 2/3。
- phonon 页面：适合跟做 GaAs phonon pipeline，尤其 `pw.scf.GaAs.in`、`ph.GaAs.in`、`q2r.GaAs.in`、`matdyn.GaAs.in` 的文件链。
- Jupyter setup：适合作为图像和后处理的辅助，不应替代 QE workflow 本身。

## 5. 哪些部分只适合作为参考？

- 高级分支如 SOC、DFT+U、Wannier 和 MD，应在完成 SCF/convergence/electronic/phonon 基础闭环后再读。
- repository 中的 `run.sh` 是教学级串联脚本，不是生产 HPC provenance 规范。

## 6. 它和本仓库哪些页面对应？

- [learn/01-first-scf-loop.md](../../learn/01-first-scf-loop.md)
- [workflows/ground-state/scf.md](../../workflows/ground-state/scf.md)
- [workflows/electronic/bands.md](../../workflows/electronic/bands.md)
- [workflows/phonon/phonon-dispersion-dfpt.md](../../workflows/phonon/phonon-dispersion-dfpt.md)
- [standards/input-file-naming.md](../../standards/input-file-naming.md)

## 7. 它的局限是什么？

它比普通博客更 workflow-aware，也提供 input 文件和 notebook，但仍主要靠文件名、命令顺序和 `run.sh` 串联。它没有把 workflow graph、provenance、restart、PASS/WARN/BLOCK gate 显式建模。

## 8. 原始链接与引用

- [主页](https://pranabdas.github.io/espresso/)
- [SCF](https://pranabdas.github.io/espresso/hands-on/scf)
- [phonon](https://pranabdas.github.io/espresso/hands-on/phonon/)
- [Jupyter setup](https://pranabdas.github.io/espresso/setup/jupyter)
- [GaAs phonon repository directory](https://github.com/pranabdas/espresso/tree/master/src/GaAs-phonon)
- [GaAs phonon run.sh](https://raw.githubusercontent.com/pranabdas/espresso/main/src/GaAs-phonon/run.sh)
