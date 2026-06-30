# Calculation Record Template

该模板用于个人学习和科研计算记录。它不要求本仓库保存具体 input/output 文件，而是规定每次 QE 计算应记录哪些上下文、参数、输出判断和下游决策。

````markdown
# Calculation Record

## System

- Material:
- Formula:
- Structure type:
- Dimensionality:
- Magnetic / SOC / charge state:

## Goal

- Workflow:
- Target property:
- Downstream dependency:
- Acceptance criterion:

## Structure source

- Source:
- File:
- Transformation:
- Relaxation status:
- Residual force / stress if available:

## Pseudopotential source

- Library:
- Exchange-correlation:
- Files:
- PP type: NC / USPP / PAW
- Recommended cutoff:
- Notes:

## Model / boundary provenance

- Exchange-correlation functional:
- Pseudopotential consistency with functional:
- Hubbard U / J / V source and projector, if used:
- Spin / magnetic configuration:
- SOC / fully relativistic pseudopotential status:
- vdW model, if used:
- Slab / vacuum / dipole / charged-state boundary:
- Excited-state / GW / BSE / TDDFT boundary, if relevant:

## Input files

- `pw.scf.<system>.in`:
- Other inputs:
- Parameter changes from previous run:

## Command

```bash
pw.x -in pw.scf.<system>.in > pw.scf.<system>.out
```

## Environment

- QE version:
- Machine / cluster:
- MPI ranks:
- OpenMP threads:
- Scheduler job id:
- Working directory:
- `prefix`:
- `outdir`:

## Output summary

- Job status:
- Final total energy:
- SCF iterations:
- Estimated SCF accuracy:
- Fermi energy:
- Forces:
- Stress:
- Warnings:

## Convergence status

- SCF convergence: PASS / WARN / BLOCK
- Cutoff convergence:
- k-point convergence:
- Smearing convergence:
- q-grid convergence:
- Notes:

## Physical result

- Main result:
- Units:
- Comparison with previous run / reference:
- Interpretation limit:

## Uncertainty statement

- Numerical uncertainty:
- Model uncertainty:
- Pseudopotential uncertainty:
- Finite-size / boundary uncertainty:
- Workflow propagation uncertainty:
- Version-sensitive items:
- Claim strength: Strong / Moderate / Boundary / Version-sensitive

## Problems encountered

- Error or warning:
- Suspected cause:
- Evidence from output:
- Action taken:

## Next action

- Continue to:
- Repeat with:
- Stop because:
````

## PASS / WARN / BLOCK 建议

- `PASS`：程序完成，关键输出达到阈值，参数收敛证据足以支持下游。
- `WARN`：程序完成，但存在需要标注的数值或物理风险；可以用于探索，不应作为最终结论。
- `BLOCK`：不能进入下游 workflow；需要重算、修复结构、换赝势或重新设计参数。
