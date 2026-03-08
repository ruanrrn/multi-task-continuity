# Multi-Task Continuity

[English](README.md) | 简体中文

![Multi-Task Continuity banner](assets/social-preview.svg)

![OpenClaw Skill](https://img.shields.io/badge/OpenClaw-Skill-283618?style=flat-square)
![Focus-Recommended Bundle](https://img.shields.io/badge/Focus-Recommended%20Bundle-DDA15E?style=flat-square&labelColor=283618)
![Works-Standalone](https://img.shields.io/badge/Works-Standalone-FEFAE0?style=flat-square&labelColor=606C38)
![Artifact-.skill Included](https://img.shields.io/badge/Artifact-.skill%20Included-DDE5B6?style=flat-square&labelColor=606C38)
![README-Bilingual](https://img.shields.io/badge/README-Bilingual-FEFAE0?style=flat-square&labelColor=BC6C25)
![License-MIT](https://img.shields.io/badge/License-MIT-FEFAE0?style=flat-square&labelColor=283618)

把多条用户请求当成一条可持续、可恢复、可汇报的工作流来跑，而不是把聊天记录当成一条会自燃的 FIFO 队列。

## Quick pitch

一套工作流，同时处理任务编排、状态落盘和重启恢复。
当聊天本身已经变成一个会移动的系统时，这个 skill 负责让系统别把自己吃掉。

## Why this exists

很多代理在“多任务编排”和“跨重启连续性”这两件事上，通常只能做好一半。

有的会排优先级，但不会持续维护状态；有的会写 `TODO.md`，但一重启就恢复错任务；还有的会并行开工，结果进度、阻塞和当前主任务全写乱了。真正的事故，往往不是某一个点坏了，而是这些点之间没有接上。

`multi-task-continuity` 就是拿来补这个断层的。

它把三件事收成一个统一工作流：

- 智能编排多个任务
- 在优先级、阻塞和下一步变化时把真实状态落盘
- 重启后先恢复正确的主任务，再重建剩余任务队列

把它看成一个推荐总包更合适，不是某种强制基础层。

## Works independently

`multi-task-continuity` 是一个完整 skill，不是那种默认你还得装一堆别的 repo 才能看懂的 meta README。

如果你想一次拿到完整工作模型，而不是手动把几个窄 skill 拼起来，就可以单独用这个仓：

- 任务拆分与优先级判断
- 安全并行执行
- 连续性文件维护
- 重启后恢复主任务
- 分阶段进度汇报

那些更小的 repo 仍然是正经选项。这个总包的意义是降低协调成本，不是把它们判成下岗工人。

## What the skill teaches

这个 skill 会要求代理：

- 把多条消息拆成真正的任务，而不是把聊天当 FIFO 队列
- 判断哪些任务应该先做，哪些可以并行，哪些需要等
- 把主线程留给编排、汇报和决策，把慢活扔去后台或子代理
- 把当前聊天的未完成队列写进 `TODO.md`
- 把重启后必须先恢复的主任务写进 `memory/active-task.md`
- 对计划内重启安排 fallback，避免重启后丢活
- 分阶段汇报，而不是憋到最后来个大总结
- 如果连续性文件互相打架，先修状态，再继续工作

## When to use it

适合这些场景：

- 用户连续发来多个任务
- 有些任务很长，有些任务很短，或者可以并行
- 任务之间有依赖、阻塞或优先级变化
- 活跃计划需要跨 session reset 或 gateway restart 保持连续
- 你希望 `TODO.md` 和 `memory/active-task.md` 始终对得上

如果只是一次性的小活，就别上这个。那属于开着塔台指挥自行车。

## Workflow overview

### 1. 先建任务地图

把用户发来的内容拆成离散任务，并记录：

- 目标是什么
- 紧急度如何
- 依赖什么
- 有没有冲突
- 大概耗时多久
- 能不能并行
- 该向用户汇报什么结果

### 2. 再决定当前活跃车道

默认优先级：

1. 解阻塞和紧急任务
2. 值得尽早启动的长任务
3. 可以塞进等待窗口的短任务
4. 清理和次级优化

### 3. 然后把真实状态写下来

- 把当前聊天的任务队列写进 `TODO.md`
- 把“如果现在重启，回来第一件该干的事”写进 `memory/active-task.md`
- 当优先级、阻塞、重要 ID、下一步发生实质变化时，同步更新这两个文件

### 4. 对用户做阶段性汇报

汇报内容应包括：

- 什么已经完成
- 什么还在跑
- 什么被卡住了
- 优先级有没有变化
- 接下来做什么

### 5. 重启后正确恢复

- 先从 `memory/active-task.md` 恢复主任务
- 第一条实质性回复里告诉用户恢复了什么
- 再从 `TODO.md` 重建剩余队列
- 恢复确认后清掉过期 fallback 状态

## Example scenarios

### 场景 1：长短任务混跑

用户发来：

- “修一下配置 bug”
- “顺便总结这个日志”
- “再开一个 PR review”

靠谱的代理应该：

1. 先看配置 bug 是否会阻塞其他事
2. 如果合适，把 PR review 丢到后台车道
3. 在等待长任务时顺手处理日志总结
4. 把当前任务队列写进 `TODO.md`
5. 配置 bug 一出结果就先回，不要硬憋到最后

### 场景 2：中途插入紧急任务

用户原本在聊一堆正常任务，后来突然来一句：

- “先别管那个了，生产挂了”

靠谱的代理应该：

1. 立刻重排优先级
2. 重写 `TODO.md`，让生产事故变成主车道
3. 重写 `memory/active-task.md`，确保重启恢复也会先救火
4. 明确告诉用户当前优先级已经变化，现在在处理什么

### 场景 3：重启打断进行中的工作

用户发来：

- “修 deploy 脚本”
- “顺便 review 这个 PR”
- “如果重启打断了，记得继续”

靠谱的代理应该：

1. 先判断 deploy 问题是不是当前主任务
2. 把 PR review 放到安全的后台车道
3. 把当前聊天队列写进 `TODO.md`
4. 把 deploy 车道写进 `memory/active-task.md`
5. 如果计划内重启，安排 fallback 并记录 `jobId`
6. 重启后先恢复 deploy，再继续 review

## Related skills

这些是相关项，不是依赖项：

- `task-orchestrator`：偏任务编排与优先级策略 — <https://github.com/ruanrrn/task-orchestrator>
- `task-state-sync`：偏连续性文件维护与状态落盘 — <https://github.com/ruanrrn/task-state-sync>

如果你只想要其中一个窄能力，用小 skill 就够；如果你想要整套工作模型，用这个总 skill。

## Social preview

建议 social preview 资源：`assets/social-preview.svg`

建议一句话文案：

> One workflow for orchestration, state sync, and restart-safe recovery.

GitHub 备注：

- 当前公开的 `gh` CLI 和 GraphQL `UpdateRepositoryInput` 都没有可写的 custom social preview 字段。
- 如果你要把这张图真正设成仓库 social preview，只能去 GitHub 仓库设置页手动上传 `assets/social-preview.svg`。

## What you get

- `multi-task-continuity/` - skill 源码
- `dist/multi-task-continuity.skill` - 可直接导入的打包产物

## Install

两种方式都可以：

1. 直接导入 `dist/multi-task-continuity.skill`
2. 把 `multi-task-continuity/` 复制到你的 skills 目录，按源码方式使用

## Repository layout

```text
multi-task-continuity/
├── LICENSE
├── README.md
├── README.zh-CN.md
├── assets/
│   └── social-preview.svg
├── multi-task-continuity/
│   └── SKILL.md
└── dist/
    └── multi-task-continuity.skill
```

## Contributing

见 `CONTRIBUTING.md`，里面写了贡献范围、PR 预期，以及怎么让这个仓继续保持为一个完整、可独立理解的总包型 skill 仓。

## Release hygiene

- 每次 skill 有实质改动后，都要重新生成 `dist/multi-task-continuity.skill`
- 保持仓库只服务于这条总包工作流，不要变成杂项代理习惯回收站
- README 里的示例要和 `multi-task-continuity/SKILL.md` 里的实际工作流保持一致
- 相关 skill 仓的链接变了就更新，别让 README 长出死链

## Repository

- GitHub: `https://github.com/ruanrrn/multi-task-continuity`
- License: MIT
