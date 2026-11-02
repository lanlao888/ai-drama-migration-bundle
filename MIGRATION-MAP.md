# 迁移映射与差异说明

## 原项目 → 目标仓库

| 原项目路径 | 目标仓库路径 | 处理方式 |
|---|---|---|
| `backend/workspace/prompts/script_rewriter.md` | `workspace/prompts/script_rewriter.md` | 保留原提示词，追加无工具时的文件式执行说明 |
| `backend/workspace/prompts/extractor.md` | `workspace/prompts/extractor.md` | 同上 |
| `backend/workspace/prompts/storyboard_breaker.md` | `workspace/prompts/storyboard.md` | 映射到目标仓库现有文件名；追加文件式输出契约 |
| `backend/workspace/prompts/prompt_generator.md` | `workspace/prompts/prompt_generator.md` | 保留双模式执行说明 |
| `backend/workspace/skills/script-rewriter/` | `skills/script-rewriter/` | 保留 SKILL.md 与 en/ja/ko 变体 |
| `backend/workspace/skills/extractor/` | `skills/extractor/` | 保留 SKILL.md 与 en/ja/ko 变体 |
| `backend/workspace/skills/storyboard-breaker/` | `skills/storyboard/` | 映射到目标仓库现有目录；保留全部语言变体 |
| `backend/workspace/skills/prompt-generator/character-prompt/` | `skills/prompt-generator/character-prompt/` | 保留原技能与语言变体 |
| `backend/workspace/skills/prompt-generator/scene-prompt/` | `skills/prompt-generator/scene-prompt/` | 同上 |
| `backend/workspace/skills/prompt-generator/prop-prompt/` | `skills/prompt-generator/prop-prompt/` | 同上 |
| `backend/workspace/skills/prompt-generator/video-prompt/` | `skills/prompt-generator/video-prompt/` | 同上 |
| 原 Mastra Agent 总控逻辑（`backend/src/agents/`） | `AGENT.md` + `workflows/short-drama-pipeline.md` | 提炼四 Agent 编排、上下文注入、技能路由、工具边界与批处理约束 |
| 原 Mastra RequestContext/DB 持久化机制 | `templates/` + `AGENT.md` 文件式合同 | 用项目圣经、集级 manifest、QA 与稳定 ID 补足当前仓库没有数据库工具的运行环境 |

## 源码事实与新增增强的边界

**直接保留/提炼自原项目：**四个 Agent 的职责；剧本场景头与 30–60 秒场景节奏；角色/场景/道具提取与去重原则；关键道具 0–3 件筛选；分镜段落 8–15 秒、2–4 子镜头、单段不跨场景、台词时长下限；分镜批次最多 8 条且首批 replace_existing；角色三视图、无人物场景空镜、白底道具单品；视频提示词 3 秒时间轴与 @资产引用；项目风格注入与精确资产名引用。

**为目标仓库补上的编排能力（属于新增规范，不冒充原项目已有功能）：**总控 AGENT.md；任务路由；跨集项目圣经与连续性台账；阶段 Gate；输入/输出结构化文件合同；上游变更影响分析与下游失效规则；文件式执行回退；manifest/QA/失败恢复。

## 不能从 Markdown 单独提供的能力

原项目的真实持久化依赖 Mastra tools、数据库和服务端接口。当前目标仓库是 Prompt/Skill 规范仓库，现有内容要求人工复制粘贴，且没有上述数据库工具。因此本迁移将“编排规则”转成 Agent 可执行的工作协议，并为没有原工具的环境提供文件式读写模式；它不会凭空变成一个自动运行的 GitHub Actions 后端。若要求定时/无人值守 API 执行，需另建 runner 并配置模型 API 与持久化层。
