# Output Review Checklist

## SCF

- [ ] 是否达到 convergence。
- [ ] total energy、Fermi energy、SCF accuracy 已记录。
- [ ] cutoff、k-points、pseudopotential 信息与 input 一致。
- [ ] warning 已解释。

## Relax / vc-relax

- [ ] 每个 ionic step 的 SCF 没有异常。
- [ ] final force / stress 已记录。
- [ ] cell constraint 已记录。
- [ ] final structure 已重新用于 static SCF。

## Bands / DOS / PDOS

- [ ] bands k-path 来源明确。
- [ ] DOS 使用 dense uniform mesh。
- [ ] energy zero / Fermi level 明确。
- [ ] PDOS 投影标签可解释。

## Phonon

- [ ] SCF 前置状态可信。
- [ ] `ph.x` perturbation 收敛。
- [ ] dyn / fc / freq 文件链完整。
- [ ] ASR 设置已记录。
- [ ] negative frequency 已分类。
