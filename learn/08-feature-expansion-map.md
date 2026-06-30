# QE 功能覆盖图

本页用于管理 QE-Learning 的覆盖边界。它不是教程清单，而是后续扩展路线：哪些功能属于核心学习闭环，哪些属于高级扩展，当前仓库是否已有页面，最小案例应该如何设计。

## 功能覆盖表

| 功能名称 | QE 原生模块 / 外部工具 | 学习阶段 | 核心必学 | 高级扩展 | 当前页面 | 推荐目录 | 推荐参考资料 | 最小案例建议 |
|---|---|---|---|---|---|---|---|---|
| SCF | `pw.x` | Stage 1 | 是 | 否 | 有 | `workflows/ground-state/scf.md` | QE INPUT_PW, Pranab SCF | Si 或 Al |
| NSCF | `pw.x` | Stage 4 | 是 | 否 | 缺 | `workflows/ground-state/nscf.md` | QE INPUT_PW, Pranab DOS | Si DOS |
| cutoff convergence | `pw.x` | Stage 2 | 是 | 否 | 归档有，需迁移 | `workflows/ground-state/cutoff-convergence.md` | QE INPUT_PW, Pranab convergence | Si cutoff sweep |
| k-point convergence | `pw.x` | Stage 2 | 是 | 否 | 归档有，需迁移 | `workflows/ground-state/kpoint-convergence.md` | QE INPUT_PW | Si/Al k sweep |
| smearing / occupations | `pw.x` | Stage 2 | 是，金属必学 | 否 | 缺 | `workflows/ground-state/smearing-convergence.md` | QE INPUT_PW | Al |
| relax | `pw.x` | Stage 3 | 是 | 否 | 归档有，需迁移 | `workflows/ground-state/relax.md` | QE INPUT_PW, Pranab relax | Si |
| vc-relax | `pw.x` | Stage 3 | 是 | 否 | 归档有，需迁移 | `workflows/ground-state/vc-relax.md` | QE INPUT_PW | Si or GaAs |
| bands | `pw.x`, `bands.x`, SeeK-path | Stage 4 | 是 | 否 | 有 | `workflows/electronic/bands.md` | QE INPUT_BANDS, Pranab bands, AiiDA PwBands | Si/GaAs |
| DOS | `pw.x`, `dos.x` | Stage 4 | 是 | 否 | 归档有，需迁移 | `workflows/electronic/dos.md` | QE docs, Pranab DOS | Si/Al |
| PDOS | `projwfc.x` | Stage 4 | 是 | 否 | 归档有，需迁移 | `workflows/electronic/pdos.md` | Pranab PDOS | GaAs |
| Fermi surface | `fs.x`, XCrySDen | Stage 4 | 否 | 是 | 缺 | `workflows/electronic/fermi-surface.md` | QE PostProc docs | Al |
| charge density | `pp.x`, VESTA/XCrySDen | Stage 6 | 是 | 否 | 归档有，需迁移 | `workflows/electronic/charge-density.md` | INPUT_PP | Si |
| electrostatic potential | `pp.x`, `average.x` | Stage 6 | 否 | 是 | 缺 | `workflows/electronic/electrostatic-potential.md` | INPUT_PP | slab |
| ELF | `pp.x` | Stage 6 | 否 | 是 | 缺 | `workflows/electronic/elf.md` | INPUT_PP | Si |
| work function | `pp.x`, `average.x` | Stage 6 | 否 | 是 | 缺 | `workflows/electronic/work-function.md` | QE PostProc, slab refs | surface slab |
| spin-polarized calculations | `pw.x` | expansion | 视体系 | 是 | 缺 | `workflows/advanced-native/spin-polarization.md` | INPUT_PW | Fe or magnetic atom |
| noncollinear / SOC | `pw.x` | expansion | 否 | 是 | 缺 | `workflows/advanced-native/noncollinear-soc.md` | INPUT_PW | Bi-containing test |
| DFT+U | `pw.x` | expansion | 视体系 | 是 | 缺 | `workflows/advanced-native/dft-plus-u.md` | INPUT_PW HUBBARD docs | NiO |
| DFT+U+V | `pw.x`, `hp.x` | expansion | 否 | 是 | 缺 | `workflows/advanced-native/dft-plus-u-v.md` | QE Hubbard docs | oxide |
| vdW corrections | `pw.x` | expansion | 视体系 | 是 | 缺 | `workflows/advanced-native/vdW-corrections.md` | INPUT_PW | graphite/layered |
| hybrid functionals | `pw.x` | expansion | 否 | 是 | 缺 | `workflows/advanced-native/hybrid-functional-overview.md` | INPUT_PW | Si small cell |
| molecular dynamics | `pw.x`, `cp.x` | expansion | 否 | 是 | 缺 | `workflows/advanced-native/molecular-dynamics.md` | PW/CP docs | small molecule/cell |
| NEB reaction path | `neb.x`, PWneb | expansion | 否 | 是 | 缺 | `workflows/advanced-native/neb-reaction-path.md` | QE PWneb docs | toy diffusion path |
| ESM / electrochemistry | `pw.x`, Environ/ESM-RISM | expansion | 否 | 是 | 缺 | `workflows/advanced-native/electrochemistry-esm.md` | INPUT_PW ESM/RISM | slab |
| Gamma phonon | `ph.x`, `dynmat.x` | Stage 5 | 是 | 否 | 缺 | `workflows/phonon/gamma-phonon.md` | INPUT_PH, Kyoto | Si/GaAs |
| q-grid phonon dispersion | `ph.x`, `q2r.x`, `matdyn.x` | Stage 5 | 是 | 否 | 有 | `workflows/phonon/phonon-dispersion-dfpt.md` | Pranab phonon, Kyoto | Si/GaAs |
| phonon DOS | `matdyn.x` | Stage 5 | 是 | 否 | 缺 | `workflows/phonon/phonon-dos.md` | Kyoto | Si |
| dielectric / Born charge | `ph.x` | Stage 5 | 否 | 是 | 缺 | `workflows/phonon/dielectric-born-effective-charge.md` | INPUT_PH, Kyoto | GaAs |
| IR/Raman | `ph.x`, `dynmat.x` | expansion | 否 | 是 | 缺 | `workflows/phonon/ir-raman.md` | PHonon docs | GaAs |
| electron-phonon overview | `ph.x`, EPW | expansion | 否 | 是 | 缺 | `workflows/phonon/electron-phonon-overview.md` | EPW docs | simple metal |
| XSpectra | `xspectra.x` | expansion | 否 | 是 | 缺 | `workflows/spectroscopy-transport/xspectra.md` | QE XSPECTRA docs | reference example |
| TDDFPT / turbo | `turbo_*` | expansion | 否 | 是 | 缺 | `workflows/spectroscopy-transport/tddfpt.md` | QE TDDFPT docs | molecule/solid example |
| TurboEELS | `turbo_eels.x` | expansion | 否 | 是 | 缺 | `workflows/spectroscopy-transport/turboeels.md` | QE TDDFPT docs | simple solid |
| GW/BSE overview | GWL/YAMBO | expansion | 否 | 是 | 缺 | `workflows/spectroscopy-transport/gw-bse-overview.md` | QE GWL, YAMBO | reference only |
| PWcond transport | `pwcond.x` | expansion | 否 | 是 | 缺 | `workflows/spectroscopy-transport/pwcond-transport.md` | QE PWcond docs | model junction |
| Wannier transport | Wannier90, WanT | expansion | 否 | 是 | 缺 | `workflows/spectroscopy-transport/wannier-transport.md` | Wannier90/WanT | interpolated bands |
| pseudopotential selection | SSSP/PseudoDojo/QE PP | Stage 1-2 | 是 | 否 | 缺 | `workflows/pseudopotentials/pseudopotential-selection.md` | QE PP docs, SSSP | Si/Al |
| pseudopotential generation | `atomic`, `ld1.x` | expansion | 否 | 是 | 缺 | `workflows/pseudopotentials/pseudopotential-generation-atomic.md` | QE atomic docs | reference only |
| pseudopotential validation | SSSP-style tests | Stage 2+ | 是 | 是 | 缺 | `workflows/pseudopotentials/pseudopotential-validation.md` | SSSP, PseudoDojo | cutoff/Delta tests |

## 后续补页策略

优先补能打通核心闭环的页面：NSCF、cutoff/k-point/smearing convergence、relax/vc-relax、DOS/PDOS、Gamma phonon、phonon DOS、pseudopotential selection。高级功能先保持覆盖图，不直接写成教程。
