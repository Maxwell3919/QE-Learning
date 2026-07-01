# Project Completion Report

本报告记录本轮 realignment 的完成状态。目标是把 QE-Learning 保持为中文 Quantum ESPRESSO 计算参考手册与 DFT 固体计算知识库，而不是自动化执行系统、工具接口规范或具体材料案例库。

## 1. 修改了什么

### 项目审计

- 新增 [PROJECT_REALIGNMENT_AUDIT.md](PROJECT_REALIGNMENT_AUDIT.md)，记录当前结构判断、偏移点、空泛页面、来源缺口、理论缺口、重复页面和优先级。
- 审计结论：项目主体方向正确，主要问题是少量入口表述、短 workflow 深度、physics-judgement 双轨重复、references 失效链接和 standards 字段不统一。

### 核心入口修正

- [README.md](README.md) 明确定位为 QE 科研计算中文参考手册与 DFT 学习知识库。
- [learn/README.md](learn/README.md) 明确 `09-feature-expansion-map.md` 是扩展功能地图，不是基础学习闭环必修阶段。
- [physics-judgement/README.md](physics-judgement/README.md) 将“结论型补写页面”改为读者可理解的“快速判断页”，并说明编号页与快速判断页的关系。
- [physics-judgement/CONCLUSION_REFERENCE_BOOK.md](physics-judgement/CONCLUSION_REFERENCE_BOOK.md) 去除工具协作语境，保持为物理判断内核。

### workflow 精修

补强以下短页面的 command boundary、output evidence、版本边界和下游解释边界：

- [workflows/electronic/charge-density.md](workflows/electronic/charge-density.md)
- [workflows/electronic/elf.md](workflows/electronic/elf.md)
- [workflows/electronic/fermi-surface.md](workflows/electronic/fermi-surface.md)
- [workflows/phonon/ir-raman.md](workflows/phonon/ir-raman.md)

这些修改没有加入具体材料案例，也没有把任何参数写成跨体系通用答案；它们只加强如何读 output、如何判断可视化/响应结果是否可解释。

### theory-minimum 补强

- 新增 [theory-minimum/functional-choice.md](theory-minimum/functional-choice.md)，把 exchange-correlation functional choice 写成 QE 用户最低理论页。
- 更新 [theory-minimum/README.md](theory-minimum/README.md) 索引。
- 修正 physics-judgement 中指向不存在 finite-size theory 页的链接，改为现有理论入口。

### physics-judgement 修正

- 修复 [physics-judgement/work-function-and-electrostatic-boundary.md](physics-judgement/work-function-and-electrostatic-boundary.md) 的 theory-minimum 路由和来源边界。
- 修复 [physics-judgement/vdw-and-low-dimensional-boundary.md](physics-judgement/vdw-and-low-dimensional-boundary.md) 的理论路由，并将来源改为更贴合 vdW/低维边界的 QE 和内部页面。
- 修复 [physics-judgement/11-surfaces-interfaces-and-electrostatics.md](physics-judgement/11-surfaces-interfaces-and-electrostatics.md) 的重复链接。

### references 修正

- 修复 QE learning resources、Yambo documentation 和 ASE espresso documentation 的失效入口。
- 更新 [references/source-index.md](references/source-index.md)、[references/README.md](references/README.md)、[references/tools/yambo.md](references/tools/yambo.md)、[references/tools/ase.md](references/tools/ase.md)。
- 补齐 [references/canonical.bib](references/canonical.bib)，使其与 [references/canonical-literature.md](references/canonical-literature.md) 的 20 篇 Fast Core Loop 对齐。

### standards 修正

- [standards/pass-warn-block.md](standards/pass-warn-block.md) 明确最终记录字段：`Status`、`Reason`、`Evidence`、`Downstream allowed`、`Uncertainty statement`。
- [standards/calculation-record-template.md](standards/calculation-record-template.md) 增加 `Review status` 区块，避免把单项 convergence 误当最终状态。
- [standards/input-file-naming.md](standards/input-file-naming.md) 增加来源入口。
- [standards/output-review-checklist.md](standards/output-review-checklist.md) 修正“四步/五步”表述不一致。

## 2. 为什么这样修改

- 入口页只负责定位和路线，不扩写成教程。
- workflow 页必须回答“能否进入下游”，而不是只列 input 字段。
- theory-minimum 页必须把理论落到 QE input/output，而不是写教材。
- physics-judgement 页必须帮助判断结果可信度，而不是扩展成论文综述。
- references 页必须让关键判断回到公开来源。
- standards 页必须让计算记录可复查、可迁移、可审计。

## 3. 仍需后续完善

### P1：workflow 深度

- `workflows/phonon/gamma-phonon.md`
- `workflows/phonon/phonon-dos.md`
- `workflows/phonon/dielectric-born-effective-charge.md`
- `workflows/phonon/phonon-debugging.md`
- `workflows/ground-state/relax.md`
- `workflows/ground-state/vc-relax.md`

这些页面已经可用，但仍可补更具体的 command boundary、source/version boundary 和 page-specific downstream gates。

### P2：physics-judgement 去重

`physics-judgement/` 目前保留编号主题页和快速判断页。长期建议是一主题一 canonical 页：编号页作为 canonical topic page，快速判断页逐步合并成页面顶部 judgement card，`CONCLUSION_REFERENCE_BOOK.md` 只保留总路由。

### P3：source-index 扩展

`learn/09-feature-expansion-map.md` 中的 XSpectra、TurboTDDFT、TurboEELS、PWcond、GIPAW、KCW 等高级包还需要补更完整的 source-index 入口。

## 4. 来源支撑状态

- QE 参数、程序行为和版本敏感字段：以 QE 官方 `INPUT_*`、PHonon guide、PostProc guide、EPW/Wannier90/Yambo 官方文档为准。
- DFT 基础和物理判断：由 [references/canonical-literature.md](references/canonical-literature.md) 与 [references/canonical.bib](references/canonical.bib) 维护。
- 本轮新增 `functional-choice.md` 的字段边界以 `INPUT_PW` 和 pseudopotential/physics judgement 页面为入口。
- 本轮 workflow 精修主要引用 `INPUT_PP`、PostProc guide、`INPUT_PH`、PHonon guide 和 XCrySDen/Yambo/ASE 等公开入口。

## 5. Boundary-level 判断

以下内容仍按 boundary-level 维护，不写成定量结论：

- 不给材料固定 cutoff、k mesh、degauss、U 值或 vdW 模型推荐。
- 不把 Kohn-Sham band gap 当作实验 gap。
- 不把 ASR 当作 phonon 收敛替代。
- 不把 work function 写成无条件单行公式。
- 不把 Wannier spread 或 EPC 单个 summary 数值当作完整可信证据。
- 不把高级工具作为 QE 基础学习的替代路线。

## 6. 验证命令与结果

本轮使用以下检查：

```bash
python3 - <<'PY'
from pathlib import Path
import re, sys
md_files=[p for roots in ['README.md','learn','workflows','theory-minimum','physics-judgement','references','standards','PROJECT_REALIGNMENT_AUDIT.md','PROJECT_COMPLETION_REPORT.md'] for p in ([Path(roots)] if Path(roots).is_file() else Path(roots).rglob('*.md'))]
missing=[]
for p in md_files:
    text=p.read_text(encoding='utf-8')
    for m in re.finditer(r'\[[^\]]+\]\(([^)]+)\)', text):
        target=m.group(1).split('#',1)[0]
        if not target or '://' in target or target.startswith('mailto:'):
            continue
        resolved=(p.parent/target).resolve()
        if not resolved.exists():
            missing.append((str(p), target))
if missing:
    for item in missing:
        print('MISSING', item)
    sys.exit(1)
print(f'checked {len(md_files)} markdown files; local links ok')
PY
```

禁用词、本地私有路径、固定材料案例、自动化系统偏移和失效旧链接使用 `rg` 做全仓库扫描；实际命令和结果在最终交付回复中列出，避免把禁用词本身写入正文造成误命中。

```bash
git diff --check
```

```bash
python3 - <<'PY'
from pathlib import Path
allowed={'README.md','learn','workflows','theory-minimum','physics-judgement','references','standards','PROJECT_REALIGNMENT_AUDIT.md','PROJECT_COMPLETION_REPORT.md','.gitignore','.git'}
actual={p.name for p in Path('.').iterdir() if p.is_dir() or p.name in allowed or p.name.endswith('.md') or p.name=='.gitignore'}
extra=sorted(x for x in actual-allowed if x!='.git')
missing=sorted((allowed-{'.git','PROJECT_COMPLETION_REPORT.md'})-actual)
print('extra:', extra)
print('missing:', missing)
raise SystemExit(1 if extra or missing else 0)
PY
```

最终验证结果见本轮最终回复。
