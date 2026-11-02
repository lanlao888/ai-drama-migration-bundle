# 自动编排工作流

本文件定义无需人工复制四份 Prompt 的单次任务执行流程。执行 Agent 读取 `AGENT.md` 后，以本文件作为 orchestration checklist。

## 触发入口

- `全流程：<项目>/<集号>`：从最新 source 开始，依次执行 Gate 1–4。
- `只改剧本：...`：执行阶段 1，生成影响报告，再按依赖图重跑下游。
- `只提取资产：...`：加载已批准剧本与 bible，执行阶段 2。
- `拆分分镜：...`：加载已批准剧本与资产，执行阶段 3。
- `生成提示词：角色/场景/道具/分镜 ID`：执行阶段 4 指定子任务。
- `检查连续性：项目/集号`：只读 bible 与已批准产物，输出问题清单，不修改数据。

## 自动执行序列

1. Resolve project + episode; if missing, ask only for the missing identifier/source.
2. Load `PROJECT.md`, `bible/*`, previous approved episode manifests and current `manifest.yml`.
3. Build a dependency snapshot and identify stale/missing outputs.
4. Run only the necessary stages in topological order.
5. After each stage, validate schema + stage gate. If failed, mark `blocked`, report exact defects, and do not run downstream stages.
6. Persist outputs, update hashes/version references, and record provenance.
7. Produce a final run report and list downstream invalidations.

## Run manifest

```yaml
project: my-drama
episode: E001
run_id: 2026-01-01T00:00:00Z
mode: full
stages:
  script: {status: approved, input_hash: "", output_hash: "", prompt: workspace/prompts/script_rewriter.md, skill: skills/script-rewriter/SKILL.md}
  extraction: {status: approved, input_hash: "", output_hash: "", prompt: workspace/prompts/extractor.md, skill: skills/extractor/SKILL.md}
  storyboard: {status: review, input_hash: "", output_hash: "", prompt: workspace/prompts/storyboard.md, skill: skills/storyboard/SKILL.md}
  prompts: {status: pending, input_hash: "", output_hash: ""}
invalidated: []
blockers: []
```

## Retry and recovery

- 模型/工具暂时失败：保留已成功阶段产物，只重试失败步骤；最多 3 次，之后 blocked 并给出错误。
- 资产保存部分成功：重新读取项目资产并按 ID/normalized_name 对账，不得盲目重复新增。
- 分镜批次中断：重新读取已保存 shot_number；按缺失序号续写，整集重做时才先清空。
- 任何写入失败：QA 标记 `not_persisted`，不可宣称完成。
