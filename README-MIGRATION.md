# AI Drama Pipeline Migration Bundle

本目录是针对 `lanlao888/my-script-agent-repo` 的迁移候选内容，依据 `ai-drama-master.7z` 原始源码提取。

## 内容
- `AGENT.md`：总控 Agent 指令，定义四阶段路由、上下文加载、门禁、依赖失效与工具映射。
- `workflows/short-drama-pipeline.md`：自动编排入口、阶段运行、重试与恢复协议。
- `workspace/prompts/`：四个可执行 Prompt，保留原项目内容并追加工具/文件双模式兼容；保留 en/ja/ko 语言版本；`storyboard_breaker` 映射为目标仓库既有的 `storyboard` 文件名。
- `source-reference/`：原始压缩包中的 Prompt 与 Skills 原文快照，供逐项追溯；不作为运行入口。
- `skills/`：原项目七个技能目录及可用语言变体；`storyboard-breaker` 映射到目标仓库既有 `storyboard` 路径。
- `templates/`：项目圣经和阶段 QA 模板。

## 合并方式
将本目录内容合并到目标仓库根目录。先审阅差异，再更新 README 的人工复制说明为总控入口说明。不要把 `ai-drama-master` 原应用的 `node_modules`、数据库、构建产物、运行时媒体或密钥文件提交到提示词仓库。

## 重要兼容说明
原项目的四个 Mastra Agent 及其数据库工具不是仅靠 Markdown 就能在 GitHub 自动运行的。此迁移包完整保留 Agent 角色/工具契约，并提供文件式工作区的编排协议；若要在 GitHub Actions 中无人值守执行，仍需接入可调用模型 API 的 runner，并安全配置密钥。当前 GitHub Copilot/IDE Agent 可依据 `AGENT.md` 编排，但是否能直接执行工具取决于宿主环境。
