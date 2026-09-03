# Speclarify

## 简介

Speclarify 是一套以规范文档驱动的 AI 开发工作流 Skill 集，名字由 spec（规范文档）与 clarify（澄清）组合而成，对应它的两大支柱：文档驱动与逐项澄清。整个流程从项目规划到功能落地分成若干个环节，每个环节由一个 Skill 负责：决策由人逐项确认，事实由 AI 自行查证，产出统一落在目标项目的 specs/ 目录中，并通过验收标准编号和文档引用保持全程可追溯。

## 安装

使用 [skills](https://github.com/vercel-labs/skills) 命令一键安装，支持 Claude Code、Codex、Cursor 等主流 AI 编程工具：

```bash
npx skills add zloreye/speclarify
```

运行后按提示选择要安装的 Skill 和目标工具即可。

## 使用建议

使用 Speclarify 时，建议搭配顶级模型以获得最佳体验，例如 Claude Fable 5、Claude Opus 5、ChatGPT 5.6 Sol 等。模型能力越强，使用体验通常也会越好。

## 核心理念

- 决策与事实分离：能通过探索环境（文件系统、代码、工具）查到的事实由 AI 自行查找；无法查到的决策通过 clarify 逐项向用户确认，达成共同理解前不动手。
- 结构预览先行：每份文档都先给出结构预览，用户确认后再生成正文。
- 单一来源：内容迁移到专门文档后，原处只保留引用；AC（验收标准，判断功能是否完成的可验证条件）只在功能文档中定义，其他文档只引用其编号。
- 全程可追溯：每条 AC 都有唯一编号，从技术设计、任务规划到编码验证都引用该编号，保证没有一条验收标准被遗漏。

## 仓库结构

```
speclarify/
├── common/                  通用 Skill
│   ├── clarify/
│   └── explain/
├── feature/                 功能阶段 Skill
│   ├── feature-document/
│   ├── feature-design/
│   ├── feature-plan/
│   └── feature-code/
└── project/                 项目阶段 Skill
    ├── project-document/
    ├── project-tech/
    ├── project-structure/
    └── project-roadmap/
```

每个 Skill 目录下的 SKILL.md 是该环节的完整指令。

## 产出的文档体系

所有文档生成在目标项目的 specs/ 目录：

```
specs/
├── 项目文档.md              项目整体决策的共同理解
├── 技术栈.md                项目级技术选型
├── 项目结构.md              项目目录与模块划分
├── 开发路线图.md            开发顺序，区分 MVP 与后续任务
└── features/
    └── {功能名}/
        ├── 功能文档.md      功能决策与 AC，AC 的唯一来源
        ├── 技术方案.md      该功能的实现设计，逐条对应 AC
        ├── 任务规划.md      可执行任务清单，用 - [ ] 跟踪进度
        └── 开发报告.md      实现范围、验证方式与结果、待验收事项
```

## 使用流程

### 项目阶段

clarify → project-document → project-tech → project-structure → project-roadmap

1. 通过 clarify 就项目整体达成共同理解，用 project-document 生成项目文档。
2. project-tech 读取项目文档，沿用已确认的技术决策，补齐未决选型，生成技术栈文档，并把项目文档中已迁移的技术选型替换为引用。
3. project-structure 读取项目文档和技术栈，确定目录与模块划分，生成项目结构文档。
4. project-roadmap 读取前三份文档，生成开发路线图：覆盖全部开发范围，区分 MVP（最小可用版本）与后续任务，按依赖关系排序，优先以可验证的用户行为组织垂直切片，并包含项目初始化任务。

### 功能阶段

feature-document → feature-design → feature-plan → feature-code

1. feature-document 读取项目文档和开发路线图中与目标功能有关的内容，补齐功能决策，生成带唯一编号 AC 的功能文档，并把项目文档中该功能的详情替换为引用。
2. feature-design 读取功能文档、技术栈、项目结构和相关代码，为每条 AC 完成技术设计，生成技术方案；影响全局的决策更新到项目级文档并在方案中引用。
3. feature-plan 读取功能文档和技术方案，按可独立验证的用户行为组织任务，明确顺序和依赖，确保每条 AC 都有任务承接，生成任务规划。
4. feature-code 按任务规划实现功能：未指定范围时执行下一个依赖已满足的任务；视任务性质决定是否采用 TDD（测试驱动开发）；涉及界面时优先用 Playwright MCP（让 AI 操作浏览器验证页面行为的服务）自行验证；发现影响既有决策的问题时回到 clarify 达成共同理解并同步更新文档。任务完成即勾选进度并更新开发报告；全部任务完成后对所有 AC 做整体验证，通过后才在开发报告中标记功能完成。

## Skill 一览

| 名称                | 阶段  | 作用                        | 输入                           | 产出         |
| ----------------- | --- | ------------------------- | ---------------------------- | ---------- |
| clarify           | 通用  | 逐项提问直到达成共同理解，每次一个问题并附推荐答案 | 当前环节的未决或冲突决策                 | 共同理解       |
| explain           | 通用  | 把文档转换成分层渐进的 HTML 讲解页面，降低阅读负担 | 要讲解的文档及其关联文档                 | HTML 讲解页面（临时生成，不入库） |
| project-document  | 项目  | 把共同理解沉淀为项目文档              | clarify 达成的共同理解              | 项目文档.md    |
| project-tech      | 项目  | 完成技术选型                    | 项目文档.md                      | 技术栈.md     |
| project-structure | 项目  | 确定项目结构                    | 项目文档.md、技术栈.md               | 项目结构.md    |
| project-roadmap   | 项目  | 规划开发路线                    | 项目文档.md、技术栈.md、项目结构.md       | 开发路线图.md   |
| feature-document  | 功能  | 完成功能决策并定义 AC              | 项目文档.md、开发路线图.md             | 功能文档.md    |
| feature-design    | 功能  | 为每条 AC 完成技术设计             | 功能文档.md、技术栈.md、项目结构.md、相关代码  | 技术方案.md    |
| feature-plan      | 功能  | 生成可执行、可验证的任务清单            | 功能文档.md、技术方案.md              | 任务规划.md    |
| feature-code      | 功能  | 实现任务并完成验证                 | 功能文档.md、技术方案.md、任务规划.md、相关代码 | 代码与开发报告.md |

## 反馈与建议

这套工作流目前以我自己的开发需求为准，对需求不同的人来说未必够用。欢迎大家通过 Issue 提出建议，帮助它覆盖更多场景。

## 来源声明

common/clarify 源自 mattpocock 的 [grill-me Skill](https://github.com/mattpocock/skills/tree/main/skills/productivity/grill-me)，在其连续提问、每次只问一个问题的做法基础上，调整为本工作流的决策澄清环节。
