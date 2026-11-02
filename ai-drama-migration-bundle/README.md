# AI 短剧自动化创作流水线

本仓库以 Agent 总控编排四阶段短剧生产流程，避免再手动复制四份 Prompt。

## 开始使用

1. 让支持仓库文件读写的 AI Agent 先读取根目录 [`AGENT.md`](AGENT.md)（Codex/兼容 Agent 可从 [`AGENTS.md`](AGENTS.md) 进入；Claude Code 从 [`CLAUDE.md`](CLAUDE.md) 进入）。
2. 按 `workflows/short-drama-pipeline.md` 发起任务，例如：`全流程：我的短剧/E001`。
3. Agent 自动装载项目圣经、当前集上游产物和对应 Skill，按阶段顺序执行；每阶段必须通过 Gate 才能继续。
4. 产物保存在 `projects/<project-slug>/episodes/<episode-id>/`，跨集角色/场景/道具档案保存在 `bible/`。

## 四阶段

| 阶段 | Prompt | Skill |
|---|---|---|
| 剧本改编 | `workspace/prompts/script_rewriter.md` | `skills/script-rewriter/SKILL.md` |
| 资产提取 | `workspace/prompts/extractor.md` | `skills/extractor/SKILL.md` |
| 分镜拆解 | `workspace/prompts/storyboard.md` | `skills/storyboard/SKILL.md` |
| 图片/视频提示词 | `workspace/prompts/prompt_generator.md` | `skills/prompt-generator/{character,scene,prop,video}-prompt/SKILL.md` |

## 自动化规则

- 总控：`AGENT.md`
- 编排：`workflows/short-drama-pipeline.md`
- 跨集一致性：`templates/PROJECT-BIBLE.md`
- 阶段验收：`templates/QA.md`
- 依赖失效：剧本或资产变更时，按总控依赖表只重跑受影响的下游阶段。

## 运行环境说明

原 `ai-drama-master` 应用使用 Mastra Agent、数据库和服务端工具执行持久化。本仓库提供 Agent 编排规范及文件式回退模式：如果运行环境具备原工具就调用工具；否则读写项目文件并明确标记 `file-based`。这套 Markdown/Skill 本身不会自动创建 GitHub Actions 后端，也不会在没有模型 Runner/API 凭据时自行后台运行。
