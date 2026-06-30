# Electronic Structure Loop

## 能力目标

掌握 QE 电子结构主线，并能把 bands、DOS、PDOS 放回同一个依赖图中审查。这个阶段要求能说明：哪一步读取 SCF ground state，哪一步需要 NSCF dense uniform k mesh，哪一步沿 high-symmetry k-path 取样，能量零点如何定义，以及 bands/DOS/PDOS 的结论是否互相支持。

本页只写通用 workflow 和审阅逻辑，不写具体材料案例，不保存真实 input/output。具体程序字段、模板和常见报错进入 `workflows/electronic/` 与 `workflows/ground-state/nscf.md`。

## 三条主线

### `SCF -> bands -> bands.x`

```text
<structure> + <pseudo>
  -> pw.x scf on <uniform_k_mesh>
  -> k-path generation / validation
  -> pw.x bands on <high_symmetry_k_path>
  -> bands.x
  -> band data / band figure / band-gap interpretation
```

这条线回答的是“电子能级沿高对称路径如何色散”。`pw.x scf` 先给出 ground-state charge density；`pw.x bands` 读取同一个 `prefix/outdir`，在给定 k-path 上求 Kohn-Sham eigenvalues；`bands.x` 再整理 bands 数据，供绘图和排序检查使用。

bands 图适合看高对称方向上的色散、直接/间接带隙候选、带交叉、简并和物理模型变化造成的能带移动。它不是 Brillouin zone 积分，因此不能用 bands k-path 替代 DOS 或收敛测试。

### `SCF -> NSCF -> dos.x`

```text
<structure> + <pseudo>
  -> pw.x scf on <scf_uniform_k_mesh>
  -> pw.x nscf on <dense_uniform_k_mesh>
  -> dos.x
  -> total DOS data / DOS figure / Fermi-level analysis
```

这条线回答的是“能量区间内有多少电子态”。NSCF 不重新建立 ground-state density，而是在已审阅 SCF 的固定电荷密度上，用更密的 uniform k mesh 和合适的 `nbnd` 生成下游需要的 eigenvalues。`dos.x` 读取 NSCF 数据并按能量窗口、步长和展宽设置生成 total DOS。

DOS 可以辅助判断 Fermi 附近是否有态、带隙范围是否合理、某个能区态密度是否集中。DOS 的可靠性强依赖 NSCF k mesh、smearing/broadening、能量窗口和能量零点记录。

### `SCF -> NSCF -> projwfc.x`

```text
<structure> + <pseudo>
  -> pw.x scf on <scf_uniform_k_mesh>
  -> pw.x nscf on <dense_uniform_k_mesh> with wavefunctions
  -> projwfc.x
  -> PDOS files by atom/orbital channel
```

这条线回答的是“总态密度中的贡献来自哪些原子和轨道通道”。`projwfc.x` 需要读取 NSCF wavefunctions，并把电子态投影到赝势定义的原子/轨道通道上。PDOS 解释必须以 `projwfc.x` output 中的 state labels 为准，不能只凭文件名或图例猜测轨道含义。

PDOS 应与 total DOS 对照：峰位、能量窗口、Fermi reference 和 broadening 设置应一致。PDOS 通道求和趋势通常应能解释 total DOS 的主要特征，但投影基组和赝势通道不保证给出严格完备分解。

## k-path 与 uniform k mesh

| 采样类型 | 常见输入语义 | 主要用途 | 不能替代什么 | 必须记录 |
|---|---|---|---|---|
| SCF uniform k mesh | `K_POINTS automatic` | 建立 ground-state charge density 和总能等积分量 | high-symmetry bands 图 | mesh、shift、对称性处理、收敛证据 |
| NSCF dense uniform k mesh | `K_POINTS automatic` | 为 DOS、PDOS、Fermi surface、Wannier 等下游生成密集 eigenvalues/wavefunctions | k-path 色散解释 | mesh、shift、`nbnd`、occupation/smearing、读取的 SCF 来源 |
| high-symmetry k-path | `K_POINTS crystal_b` 等路径输入 | 画 bands，检查高对称方向色散 | DOS/PDOS/积分收敛 | k-path 来源、cell convention、坐标基底、标签、是否人工修改 |

最小掌握要求是能区分三句话：

- SCF uniform mesh 是为了得到可信 ground state。
- NSCF dense uniform mesh 是为了给 DOS/PDOS 等积分或投影型后处理提供足够采样。
- bands k-path 是为了沿高对称线展示色散，不是为了积分。

如果 primitive cell、conventional cell、标准化结构和 k-path 生成工具之间没有对齐，bands 标签可能看起来合理但实际对应错误路径。每次 bands 记录都应写明 k-path 是从哪个结构、哪个 cell convention、哪个 reciprocal-coordinate basis 得到的。

## Fermi energy、VBM、CBM、band gap、DOS、PDOS 的关系

Fermi energy 是 QE output 和电子结构图中常见的能量参考。对金属，Fermi energy 附近存在占据变化，DOS 在 Fermi energy 处是否非零是常用审查点。对绝缘体或半导体，Fermi energy 可能落在带隙内；判断带隙时更应明确 VBM 和 CBM，不能只依赖 Fermi energy。

VBM 是价带最高能级，CBM 是导带最低能级。band gap 通常由 `CBM - VBM` 给出；若 VBM 和 CBM 在同一 k 点，是 direct gap；若在不同 k 点，是 indirect gap。bands 图能显示沿选定 high-symmetry path 的 VBM/CBM 候选，但如果真正的极值不在路径上，bands 图可能低估或漏判极值位置。

DOS 是对能量分布的统计描述。若 DOS 在某个能量窗口为零，通常对应没有电子态；若 Fermi 附近 DOS 非零，通常支持金属性判断。但 DOS 的分辨率和噪声取决于 uniform k mesh、broadening 和能量步长，不能脱离这些设置解释。

PDOS 把 DOS 的特征分解到原子/轨道通道上，用于解释峰或带边的主要贡献。PDOS 不应单独给出 band gap 结论；它应与 total DOS、bands 图、投影标签和赝势通道一起读。

## Output Review 检查清单

### Bands review

- SCF 与 bands 是否使用同一结构、赝势、cutoff、物理模型和 `prefix/outdir`。
- bands run 是否读取了目标 SCF ground-state 数据，而不是旧 scratch 或错误目录。
- k-path 来源、cell convention、坐标基底、高对称点标签和路径段数量是否记录完整。
- `nbnd` 是否覆盖关注的价带、导带和能量窗口。
- `bands.x` 是否生成预期 band data，是否有排序、交叉或异常跳变需要回看原始 eigenvalues。
- 图中的 energy zero 是 Fermi energy、VBM、CBM 还是自定义参考。

### DOS review

- NSCF 是否读取已通过审阅的 SCF，而不是重新定义 ground state。
- NSCF k mesh 是否比 SCF 更密，且适合目标 DOS 分辨率。
- `nbnd`、occupation、smearing/degauss 是否覆盖目标能区并与 SCF/解释目标一致。
- `dos.x` 是否读取目标 NSCF 数据，能量窗口和 `DeltaE` 是否覆盖 VBM/CBM、Fermi 附近或关注能区。
- DOS 图是否记录能量零点、展宽设置和单位。

### PDOS review

- `projwfc.x` 是否读取包含 wavefunctions 的目标 NSCF 数据。
- PDOS 文件是否完整生成，文件前缀是否可追溯。
- output 中的 atomic/orbital state labels 是否已摘录，图例是否与这些 labels 对齐。
- PDOS 与 total DOS 是否使用一致的 energy window、energy zero、broadening 和绘图处理。
- Lowdin charge 是否只作为辅助信息，未被单独当作充分价态证据。

## bands 与 DOS 不一致时如何排查

| 现象 | 可能原因 | 排查顺序 |
|---|---|---|
| bands 像绝缘体，DOS 在 Fermi 附近非零 | DOS k mesh/展宽导致尾巴，或 bands path 没经过真实带边/交叉 | 先对齐能量零点，再检查 DOS broadening 和 dense k mesh，最后检查是否需要更完整 k 空间搜索 |
| bands 显示带隙，DOS 带隙明显不同 | VBM/CBM 没在 k-path 上，或 `nbnd`/能量窗口不足 | 检查 NSCF eigenvalues 范围、DOS energy window、bands 的路径覆盖和 band extrema 记录 |
| bands 和 DOS 的 Fermi level 不同 | SCF/NSCF/bands 读取了不同 `prefix/outdir`，或 occupation/smearing 设置不一致 | 核对三套 input/output 的 `prefix/outdir`、QE version、occupation、smearing、electron count |
| DOS 很锯齿，但 bands 平滑 | NSCF k mesh 太稀或 `DeltaE` 太小 | 加密 NSCF uniform mesh，调整能量步长和展宽后重新审阅 |
| PDOS 峰位与 total DOS 对不上 | 能量零点、窗口、broadening 或绘图脚本不一致 | 用同一 Fermi/VBM reference 重新处理 total DOS 和 PDOS，并核对 `projwfc.x` output labels |
| `bands.x` 或 `dos.x` 读不到数据 | `prefix/outdir` 不一致、scratch 被清理、上游未完成 | 从 SCF output 开始逐步核对每个程序读取的目录和生成文件 |
| 结论只来自一张图 | 缺少上游审阅或记录 | 回到 SCF、NSCF、bands/DOS/PDOS output review，先给 PASS/WARN/BLOCK 再写解释 |

排查时不要先修图。先确认同一结构、同一赝势、同一物理模型、同一上游 SCF、同一能量参考；再检查 k 点语义；最后才调整绘图、插值或平滑参数。

## 下游准入

完成 electronic structure loop 后，可以进入以下分支，但必须写明准入边界：

- postprocessing：可以使用已审阅的 SCF charge density、potential 或 wavefunctions；不得把未审阅的 bands/DOS 图当作空间场后处理依据。
- Wannier90 overview：需要明确 NSCF k mesh、`nbnd`、投影窗口和 bands/DOS/PDOS 对目标能区的支持程度。
- Fermi surface 或输运相关分支：只适合在金属或目标能区明确的情况下进入，并需要更密 k mesh 与 Fermi-level 审阅。
- phonon/DFPT 或 electron-phonon 分支：电子结构结果可以作为背景解释，但声子准入仍以 SCF、结构、收敛和 DFPT output review 为准。

如果 bands、DOS、PDOS 之间存在未解释矛盾，本阶段只能给 `WARN` 或 `BLOCK`，不能进入依赖电子结构结论的下游解释。

## 记录要求

每次学习或计算记录至少包含：

- 上游 SCF：input、output、QE version、赝势来源、cutoff、SCF k mesh、occupation/smearing、收敛状态、`prefix/outdir`。
- Bands：bands input/output、`bands.x` input/output、band data、k-path 来源说明、cell convention、energy zero、`nbnd`、warnings。
- DOS：NSCF input/output、`dos.x` input/output、DOS data、dense uniform k mesh、energy window、`DeltaE`、broadening/energy reference。
- PDOS：NSCF wavefunction 记录、`projwfc.x` input/output、PDOS 文件清单、投影 labels 摘录、与 total DOS 的对照说明。
- 解释层：Fermi energy、VBM、CBM、band gap、DOS/PDOS 主要特征、bands-DOS 一致性检查、PASS/WARN/BLOCK 和下游允许范围。

推荐把图像和解释分开记录：图像说明只写数据来源、坐标轴、能量零点和处理方式；物理解释另写，并引用 output review 中的证据。

## 完成判据

- 能画出并解释 `SCF -> bands -> bands.x`、`SCF -> NSCF -> dos.x`、`SCF -> NSCF -> projwfc.x` 三条链。
- 能说明 k-path 与 uniform k mesh 的不同语义，并指出它们各自服务的下游任务。
- 能用 Fermi energy、VBM、CBM 和 band gap 解释 bands 与 DOS 的关系。
- 能说明 DOS 与 PDOS 的关系，以及 PDOS 解释为什么必须回看 `projwfc.x` labels。
- 能在 bands 与 DOS 不一致时给出按优先级排列的排查路径。
- 能写出一份包含上游依赖、energy reference、output review 和下游准入的记录。

## 可验证输出

- electronic structure 依赖图，至少包含 SCF、bands、bands.x、NSCF、dos.x、projwfc.x。
- bands k-path 来源说明。
- NSCF dense uniform k mesh 设置说明。
- energy zero / Fermi energy / VBM / CBM / band gap 说明。
- bands/DOS/PDOS output review。
- bands-DOS consistency check：给出 PASS/WARN/BLOCK 和理由。
- 下游准入说明：允许进入哪些 workflow，哪些结论仍需复查。

## 推荐阅读

- [workflows/electronic/bands.md](../workflows/electronic/bands.md)
- [workflows/electronic/dos.md](../workflows/electronic/dos.md)
- [workflows/electronic/pdos.md](../workflows/electronic/pdos.md)
- [workflows/ground-state/nscf.md](../workflows/ground-state/nscf.md)
- [theory-minimum/kpoints-symmetry-kpath.md](../theory-minimum/kpoints-symmetry-kpath.md)

## 后续进入哪个 workflow

完成后进入 postprocessing、Wannier90 overview、Fermi surface 或高级电子结构分支。进入前必须先写明：上游 SCF/NSCF 是否可追溯，bands/DOS/PDOS 是否一致，当前结论允许支持哪些下游问题。

## 资料来源

- QE `pw.x` input reference
- QE `bands.x` input reference
- QE `dos.x` input reference
- QE `projwfc.x` input reference
- SeeK-path documentation
- Pranab Das QE tutorial
