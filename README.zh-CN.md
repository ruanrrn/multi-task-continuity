# Multi-Task Continuity

[English](README.md) | 简体中文

![Multi-Task Continuity banner](assets/social-preview.svg)

![OpenClaw Skill](https://img.shields.io/badge/OpenClaw-Skill-283618?style=flat-square)
![Focus-Restart--Safe Multi--Task Workflow](https://img.shields.io/badge/Focus-Restart--Safe%20Multi--Task%20Workflow-DDA15E?style=flat-square&labelColor=283618)
![Works-Standalone](https://img.shields.io/badge/Works-Standalone-FEFAE0?style=flat-square&labelColor=606C38)
![Artifact-.skill Included](https://img.shields.io/badge/Artifact-.skill%20Included-DDE5B6?style=flat-square&labelColor=606C38)
![README-Bilingual](https://img.shields.io/badge/README-Bilingual-FEFAE0?style=flat-square&labelColor=BC6C25)
![License-MIT](https://img.shields.io/badge/License-MIT-FEFAE0?style=flat-square&labelColor=283618)

把多条用户请求组织成一条可跨重启恢复的工作流，供 OpenClaw agent 稳定执行。

## 概览

`multi-task-continuity` 是一个可独立使用的 OpenClaw skill，面向那些需要同时处理多条请求、又不能把任务顺序、用户可见进度和重启恢复搞乱的 agent。

它把三个经常在真实工作中互相脱节的能力收成一套统一工作流：

- 多任务编排与优先级决策
- 进行中状态的连续性文件维护
- 重启或 session reset 之后的 resume-first 恢复

这个仓库的目标很明确：当聊天不再是单条线性任务时，保证 agent 的活跃工作仍然有秩序、可恢复、可汇报。

## 为什么需要它

多任务工作真正容易翻车的地方，通常不是单个能力不存在，而是这些能力之间没有接牢。

代理可能会排优先级，但不会持续落盘主任务；可能会写 `TODO.md`，却没有让 `memory/active-task.md` 与之保持一致；也可能口头上支持重启恢复，但恢复回来以后讲的是另一件事。

`multi-task-continuity` 的作用，就是把这些断点收口成一条明确工作流。它把任务拆分、状态持久化、阶段性进度汇报和重启恢复放进同一个操作模型里，避免各做各的，最后彼此打架。

## 适用边界

当问题需要的是整套组合工作流，而不是某一个窄能力时，就适合用这个仓库。

适合这些场景：

- 用户跨多条消息连续发来多个任务
- 部分任务耗时长、可并行，或者对阻塞很敏感
- 工作进行中优先级可能变化
- 当前计划需要跨重启或 session reset 保持连续
- `TODO.md` 和 `memory/active-task.md` 必须随着状态变化持续对齐

不适合这些场景：

- 一次性的小任务
- 纯同步、单步骤的短操作
- 只需要修一个连续性文件的窄问题
- 仅靠更小的 companion skill 就足够解决的问题

这个仓库是一个总包式 workflow skill，不是所有 agent 都必须加载的基础层。

## Skill 覆盖内容

这个 skill 会要求 agent 把多请求工作当成一条受控工作流来跑：

- 把用户消息拆成离散任务，而不是把聊天当成 FIFO 队列
- 评估执行顺序，并在确实安全时使用并行车道
- 把主线程留给编排、用户沟通和关键决策
- 把按聊天维度维护的未完成队列写入 `TODO.md`
- 把必须优先恢复的单条主任务写入 `memory/active-task.md`
- 当阻塞、优先级、重要 ID 或下一步发生实质变化时，同步更新两个连续性文件
- 在工作进行中分阶段汇报进度
- 重启后先恢复正确的主任务，再重建其余队列

## 工作流摘要

一轮标准工作流通常分五步：

1. 建立任务地图：明确目标、紧急度、依赖、冲突、耗时和可并行性。
2. 选择活跃车道：优先启动解阻塞和紧急任务，再尽早启动值得并行的长任务。
3. 落盘真实状态：把队列写入 `TODO.md`，把 resume-first 主任务写入 `memory/active-task.md`。
4. 做阶段性汇报：让用户知道哪些完成了、哪些还在跑、哪些被卡住、优先级是否变化。
5. 重启后恢复：先根据 `memory/active-task.md` 继续主任务，再从 `TODO.md` 重建剩余队列。

## 何时使用

当风险主要不在单个任务本身，而在多任务协调成本时，就应该考虑 `multi-task-continuity`。

典型触发语句包括：

- “这几个请求并行处理，过程里持续告诉我进度。”
- “先救急，但别把后面的队列弄丢。”
- “如果中间重启，回来先接着做正确那件事。”
- “优先级在变，`TODO.md` 和 `memory/active-task.md` 也要跟着准。”

## 代表性结果

### 长短任务混跑

用户先后提出 bug 修复、日志总结和 PR review。

靠谱的 agent 应该先识别是否存在 blocker-sensitive 任务，尽早启动安全的后台车道，在等待窗口处理短任务，同时把队列写清楚，并在阶段性结果出现时立刻回报。

### 中途插入紧急任务

已有工作进行到一半时，用户又抛来生产事故。

靠谱的 agent 应该立刻重排队列、重写两个连续性文件以反映新的主任务，并在继续执行前明确告诉用户优先级已经变化。

### 工作中途被重启打断

用户希望当前修复工作在重启后还能准确续上，同时不丢掉次级任务。

靠谱的 agent 应该让 `memory/active-task.md` 明确记录当前主车道，用 `TODO.md` 保留完整队列，并在重启后先恢复正确任务，再继续其余工作。

## 相关 skill 仓

这些仓库是相关示例，不是必需依赖：

- `task-orchestrator`：聚焦任务编排与优先级策略的能力车道 - <https://github.com/ruanrrn/task-orchestrator>
- `task-state-sync`：聚焦连续性文件维护的能力车道 - <https://github.com/ruanrrn/task-state-sync>

如果你要的是整套组合工作模型，就用这个仓库；如果你只需要其中一个窄能力，可以直接使用对应的小 skill。

## 安装

两种方式都可以：

1. 直接把 `dist/multi-task-continuity.skill` 导入 OpenClaw 环境。
2. 如果你需要可编辑源码，就把 `multi-task-continuity/` 复制到你的 skills 目录。

## 仓库内容

- `multi-task-continuity/` - skill 源码
- `dist/multi-task-continuity.skill` - 可直接导入的打包产物
- `assets/social-preview.svg` - 仓库 banner 和建议使用的 social-preview 资源

## Social preview

建议使用的 social preview 资源：`assets/social-preview.svg`

建议一句话文案：

> One workflow for orchestration, state sync, and restart-safe recovery.

GitHub 说明：

- 当前公开的 `gh` CLI 和 GraphQL `UpdateRepositoryInput` 都没有可写的 custom social preview 字段。
- 如果要把这张图真正设成仓库 social preview，仍然需要到仓库设置页手动上传 `assets/social-preview.svg`。

## 仓库结构

```text
multi-task-continuity/
├── LICENSE
├── README.md
├── README.zh-CN.md
├── CONTRIBUTING.md
├── assets/
│   └── social-preview.svg
├── multi-task-continuity/
│   └── SKILL.md
└── dist/
    └── multi-task-continuity.skill
```

## 贡献

见 `CONTRIBUTING.md`。其中说明了贡献范围、PR 预期，以及如何保持这个仓库继续聚焦“多请求连续性工作流”，而不是滑坡成通用 agent 工作流大全。

## 发布卫生

- 每次 skill 有实质改动后，都要重新生成 `dist/multi-task-continuity.skill`
- 保持 `README.md`、`README.zh-CN.md` 和 `multi-task-continuity/SKILL.md` 一致
- 让仓库持续聚焦这条组合工作流本身，不要无限扩张职责
- 如果 companion repo 改名或迁移，及时更新相关链接

## 仓库信息

- GitHub: `https://github.com/ruanrrn/multi-task-continuity`
- License: MIT
