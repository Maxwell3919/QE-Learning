# Kyoto University QE Phonon DokuWiki

## 1. 它是什么？

Kyoto QE phonon DokuWiki 是研究者维护的专家笔记型教程。它不是零基础课程，而是围绕 phonon 计算提供密集的输入示例、命令链和物理判断提示。

## 2. 适合学习或解决什么问题？

它特别适合补充 DFPT phonon 的专家判断：

- Gamma phonon
- finite-q phonon
- q-grid phonon dispersion
- phonon DOS
- ASR
- `epsil`
- dielectric tensor
- Born effective charge
- metal vs insulator phonon 差异
- negative frequency 和 acoustic branch 判断

## 3. 它覆盖哪些 QE workflow？

- `pw.x scf -> ph.x` 的 Gamma 点 phonon。
- `ph.x` finite-q phonon。
- `ph.x(ldisp=.true.) -> q2r.x -> matdyn.x` 的 phonon dispersion / DOS。
- `dynmat.x` 的 Gamma 模式分析和 ASR。
- XCrySDen / AXSF 形式的模式可视化。

## 4. 哪些部分适合跟做？

- Gamma 点 `ph.x` 输入示例，尤其 `tr2_ph`、`prefix`、`epsil`、`fildyn`。
- `ldisp=.true.` + `nq1/nq2/nq3` 的 q-grid 设置。
- `q2r.x` 和 `matdyn.x` 的 force constants 与 dispersion 串联。
- acoustic sum rule 的输出比较。

## 5. 哪些部分只适合作为参考？

- 金属体系避开 Gamma electric-field 处理、轻微移动 q 点等建议属于专家经验，初学者应先理解适用条件，再在本仓库 workflow 中记录为 WARN 或实验性处理。
- 页面假设读者已有一定 QE 和 DFPT 背景，不适合作为第一条学习主线。

## 6. 它和本仓库哪些页面对应？

- [learn/05-phonon-dfpt-loop.md](../../learn/05-phonon-dfpt-loop.md)
- [workflows/phonon/phonon-dispersion-dfpt.md](../../workflows/phonon/phonon-dispersion-dfpt.md)
- `workflows/phonon/gamma-phonon.md`（待补）
- `workflows/phonon/phonon-dos.md`（待补）
- `workflows/phonon/dielectric-born-effective-charge.md`（待补）
- `workflows/phonon/phonon-debugging.md`（待补）

## 7. 它的局限是什么？

它是专家笔记风格，不是可复现项目模板。它强调关键物理/数值 caveat，但没有完整的 GitHub record、环境记录、source traceability 或 workflow graph 规范。

## 8. 原始链接与引用

- [phonon calculation DokuWiki](https://www2.yukawa.kyoto-u.ac.jp/~koudai.sugimoto/dokuwiki/doku.php?id=quantumespresso%3Aphonon%3A%E3%83%95%E3%82%A9%E3%83%8E%E3%83%B3%E3%81%AE%E8%A8%88%E7%AE%97)
- [phonon DOS and dispersion DokuWiki](https://www2.yukawa.kyoto-u.ac.jp/~koudai.sugimoto/dokuwiki/doku.php?id=quantumespresso%3A%E3%83%95%E3%82%A9%E3%83%8E%E3%83%B3%E3%81%AE%E7%8A%B6%E6%85%8B%E5%AF%86%E5%BA%A6%E3%81%A8%E5%88%86%E6%95%A3)
