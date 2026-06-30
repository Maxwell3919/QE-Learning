# Calculation Record Template

每次 QE 计算都应保留一份 `record.md`。它的作用不是重复 input 文件，而是记录科学上下文、执行环境、输出判断和下一步决策。

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
