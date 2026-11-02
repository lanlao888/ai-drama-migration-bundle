# AGENT.md — 短剧自动化创作总控

本文件是本仓库短剧创作流水线的唯一总控入口。任何 AI Agent、IDE Agent 或人工协作流程都必须先读取本文件，再按任务类型调用对应阶段。阶段规范以 `workspace/prompts/` 与 `skills/` 为准；本文件负责编排、上下文、依赖、验收与恢复，不重复定义各阶段的专业创作细节。

## 1. 总目标与边界

将小说/故事原文转化为可制作的短剧生产资料：

`原文 → 剧本 → 角色/场景/关键道具资产 → 分镜段落 → 图片/视频提示词 → 生产包`

- 不得跳过上游依赖；下游输出必须引用已确认的上游版本。
- 不得凭空编造资产 ID；没有持久化工具时使用稳定的项目内 slug，并在清单中标注 `local-only`。
- 不得把“给出建议/提示词文本”冒充“已保存/已完成”。每阶段必须记录产物路径、版本、状态与验收结果。
- 原项目的 Mastra Agent 工具名是来源系统接口契约。若当前运行环境没有这些工具，先产出结构化文件，不得假称调用了数据库保存工具。
- 任何破坏性重做（覆盖整集分镜、删除资产、批量重生成）必须先确认目标集和影响范围；除非用户已明确要求整集重做。

## 2. 项目工作区约定

每个项目至少维护以下目录（已有目录可复用，不要重复造一套）：

```text
projects/<project-slug>/
  PROJECT.md                 # 项目设定、视觉基调、版本与约束
  bible/
    characters.yml           # 角色唯一档案与别名
    locations.yml            # 场景/地点档案
    props.yml                # 关键道具档案
    continuity.md            # 跨集连续性与禁改项
  episodes/E001/
    source.md                # 原始小说/大纲及来源
    script.md                # 已确认剧本
    assets.yml               # 本集资产引用/新增/复用关系
    storyboard.yml           # 分镜段落结构化数据
    prompts/
      image-prompts.md
      video-prompts.md
    QA.md                    # 各阶段验收记录
    manifest.yml             # 依赖版本、产物哈希、状态
```

命名规则：项目 slug、集号 `E001`、场景 `S01`、分镜 `SB001`、资产 ID 全项目唯一。角色/道具以规范名为主，别名写入 aliases，不另建重复实体。

## 3. 总控执行协议

每次任务先判断用户意图：

1. **新建项目/新剧**：建立 PROJECT.md 与 bible，再开始第一集。
2. **新集全流程**：按阶段 1→2→3→4 顺序执行，每个阶段通过验收门后才能进入下一阶段。
3. **单阶段任务**：仅运行指定阶段，但先加载其上游最新产物与项目 bible。
4. **修改已有产物**：识别变更字段，按第 7 节依赖图使下游失效并重跑。
5. **续作/跨集任务**：必须先加载 bible、continuity.md 和前集已确认资产，禁止重新设计既有角色外观。

每个阶段通用步骤：`读取输入与版本 → 执行对应 Skill → 生成产物 → 自检 → 对照验收清单 → 保存 → 更新 manifest/QA → 才可进入下一阶段`。

阶段调用映射：

| 阶段 | Prompt | Skill | 必需输入 | 标准输出 |
|---|---|---|---|---|
| 1 剧本改编 | `workspace/prompts/script_rewriter.md` | `skills/script-rewriter/SKILL.md` | source.md、PROJECT.md、continuity.md | script.md |
| 2 资产提取 | `workspace/prompts/extractor.md` | `skills/extractor/SKILL.md` | script.md、bible 全量资产 | assets.yml、bible 更新候选 |
| 3 分镜拆解 | `workspace/prompts/storyboard.md` | `skills/storyboard/SKILL.md` | script.md、assets.yml、bible | storyboard.yml |
| 4 提示词生成 | `workspace/prompts/prompt_generator.md` | 对应 4 个 prompt-generator 子 Skill | storyboard.yml、资产档案、视觉风格 | prompts/* |

注意：原项目 Agent 实际有四个角色：`script_rewriter`、`extractor`、`storyboard_breaker`、`prompt_generator`。提示词生成 Agent 内部再分角色/场景/道具/视频四种 Skill，不把它们误算成额外 Agent。

## 4. 上下文装载规则

每次阶段执行前，按优先级加载：

1. 用户本轮明确指令（最高优先级，但不能违反已确认的事实与安全边界）。
2. `PROJECT.md`：题材、时代、画幅、目标平台、语言、视觉风格、禁用元素。
3. `bible/continuity.md`：已发生事件、角色状态、服装/伤势/关系变化、时间线、伏笔状态。
4. `bible/*.yml`：角色、场景、道具规范档案及别名/参考图。
5. 当前集上游产物与 manifest 版本。
6. 对应 Prompt 与 Skill。

上下文冲突时不静默覆盖：列出冲突字段，优先保留已确认 bible 值；只有剧情明确发生变化时才创建新状态/造型版本。

## 5. 阶段门禁（必须逐项通过）

### Gate 1 — 剧本改编
- 主线事件、人物关系、关键因果与原文一致；删改有理由。
- 场景头符合 `## S编号 | 内景/外景 · 地点 | 时间段`，编号连续。
- 动作可拍、对白推动剧情；心理活动转成动作/表情或必要旁白。
- 每场约 30–60 秒；没有景别、运镜等分镜语言。
- 开场钩子、冲突升级、集尾钩子完整；角色说话风格连续。
- 输出完整剧本并更新 source/script 版本，不可只交片段。

### Gate 2 — 资产提取
- 当前集所有有台词或重要动作的角色均已覆盖；角色 appearance/styling 字段充分且可视化。
- 场景按地点+时间段拆分；prompt 只写空间/陈设，lighting 写光线。
- 道具严格筛选：剧情推动且值得独立出图；通常 0–3 件；日常物品/场景陈设不提取。
- 角色/道具按规范名与 normalized_name 去重；场景按地点+时间段去重；复用既有 ID。
- 当前集关联完整；无新增时明确记录“复用既有资产”，不得重复建档。

### Gate 3 — 分镜拆解
- 以“一个分镜段落=一个视频任务”为单位；每段 8–15 秒，通常 2–4 个子镜头，单段不跨场景。
- 先按叙事节拍切段；目标总时长约为剧本字数÷500字/分钟，段数约总时长÷12秒（允许±20%）。
- 台词时长满足：段落秒数 ≥ 台词/旁白字数÷4.5 + 2 秒表演余量。
- description 的【镜头N】结构清楚；画面先于台词；角色、道具、场景绑定均来自本集资产。
- video_prompt 每 3 秒一段，时间轴连续；每个子镜头映射 1–2 个连续 3 秒段；不增删台词，不跨场景。
- 全集 shot_number 连续；批量保存时首批 replace_existing=true，之后追加；每批最多 8 段（适用于原 Mastra 工具）。

### Gate 4 — 图片/视频提示词
- 角色图：左侧正脸特写+右侧正/侧/背三视图；同一张脸、发型、服装一致；白底中性站姿。
- 场景图：无人物广角空镜；前/中/后景、出入口、地面、墙面、主要陈设相对位置明确。
- 道具图：白底单品、比例准确、完整无裁切、无手/人物/其他物件。
- 视频提示词：第一行人物/场景信息头；每段有正确 @ 引用；引用名与资产规范名完全一致；动作可见；台词来自 description；时间轴连续。
- 图片提示词不重复添加项目视觉风格（原项目保存工具会自动注入）；如无该工具，需在生产配置层显式合并且只合并一次。
- 所有目标资产/分镜均已生成或明确标记待生成；保存结果逐条核验。

## 6. 结构化产物契约

`assets.yml` 最少包含：

```yaml
characters:
  - id: CHAR_001
    name: 林小雨
    aliases: []
    appearance: ""
    styling: ""
    source_episode: E001
    status: approved
scenes:
  - id: SCN_001
    location: 咖啡厅
    time: 黄昏
    prompt: ""
    lighting: ""
    status: approved
props: []
```

`storyboard.yml` 每段最少包含：

```yaml
- id: SB001
  shot_number: 1
  title: ""
  scene_id: SCN_001
  character_ids: [CHAR_001]
  prop_ids: []
  duration: 12
  description: "【镜头1】...
【镜头2】..."
  atmosphere: ""
  video_prompt: ""
  status: draft
```

状态统一使用 `draft → review → approved → produced`；失败使用 `blocked` 并记录原因。未通过门禁不得标记 approved。

## 7. 依赖与自动重跑规则

| 上游变更 | 必须失效/重验的下游 | 默认动作 |
|---|---|---|
| 原文/故事大纲 | 剧本、资产、分镜、全部提示词 | 重跑阶段 1–4 |
| 剧本对白/动作/事件/场景 | 资产、分镜、视频提示词；受影响角色/场景图片提示词按字段判断 | 重跑阶段 2–4 |
| 角色 appearance/styling | 该角色图片提示词、绑定该角色的分镜视频提示词/参考图 | 重跑角色提示词与相关视频提示词 |
| 场景 location/time/prompt/lighting | 场景图片提示词、绑定场景的分镜/视频提示词 | 重跑场景提示词与相关分镜提示词 |
| 道具 description | 道具图片提示词、引用该道具的分镜视频提示词 | 重跑道具提示词与相关视频提示词 |
| 分镜 description/绑定/时长 | 该分镜 video_prompt、对应视频任务 | 只重跑受影响分镜及视频 |
| 项目视觉风格 | 全部图片最终提示词；视频提示词仅在风格规则要求时 | 重跑全部图片提示词 |
| 单个 video_prompt | 该分镜视频生成任务 | 只重跑该分镜 |

重跑前先生成 impact report：变更文件/字段、受影响实体、受影响集数、待重跑任务。保留旧版本与哈希，禁止无记录覆盖。

## 8. 原项目工具映射（能力存在时必须优先使用）

- 剧本：`read_episode_script` → `rewrite_to_screenplay`（如可用）→ `save_script`
- 资产：`read_script_for_extraction`、`read_existing_characters/scenes/props` → `save_dedup_characters/scenes/props`
- 分镜：`read_storyboard_context` → `save_storyboards`（首批 replace_existing=true，后续追加）→ `update_storyboard`
- 图片提示词：`read_characters/scenes/props` → `save_character_final_prompt` / `save_scene_final_prompt` / `save_prop_final_prompt`
- 视频提示词：`read_storyboard_context` → `update_storyboard`（只传 storyboard_id 与 video_prompt）

工具缺失时，将相同逻辑落实为本仓库 Markdown/YAML 文件读写，并在 QA 中标记执行模式 `file-based`；不可编造工具调用结果。

## 9. 交付汇报格式

每次任务结束只报告可核验事实：完成阶段、输出路径、版本、通过/未通过门禁、未解决阻塞、下游是否需要重跑。禁止只说“已完成”而不列出产物。
