# Multi-Task Continuity

一个给 OpenClaw 用的 skill：把多条用户请求当成一条可持续、可恢复、可汇报的工作流来处理，而不是把聊天记录当流水线硬排队。

## 这东西为什么存在

很多代理在“多任务编排”和“跨重启连续性”这两件事上，通常只能做好一半。

有的会排优先级，但不会持续维护状态；有的会写 `TODO.md`，但一重启就恢复错任务；还有的会并行开工，结果进度、阻塞和当前主任务全写乱了。真正的事故，往往不是某一个点坏了，而是这些点之间没有接上。

`multi-task-continuity` 就是拿来补这个断层的。

它把三件事收成一个统一工作流：

- 智能编排多个任务
- 在优先级、阻塞、下一步变化时落盘真实状态
- 重启后先恢复正确的主任务，再重建剩余任务队列

如果你希望代理在真实聊天场景里像个靠谱的操盘手，而不是一个只会按时间戳排队的客服脚本，这个 skill 就是干这个的。

## 它能教会代理什么

这个 skill 会要求代理：

- 把多条消息拆成真正的任务，而不是把聊天当 FIFO 队列
- 判断哪些任务应该先做，哪些可以并行，哪些需要等
- 把主线程留给编排、汇报和决策，把慢活扔去后台或子代理
- 把当前聊天的未完成队列写进 `TODO.md`
- 把重启后必须先恢复的主任务写进 `memory/active-task.md`
- 对计划内重启安排 fallback，避免重启后丢活
- 分阶段汇报，而不是憋到最后来个大总结
- 如果连续性文件互相打架，先修状态，再继续工作

## 什么时候用

适合这些场景：

- 用户连续发来多个任务
- 有些任务很长，有些任务很短
- 任务之间有依赖、阻塞或优先级变化
- 任务需要跨 session reset 或 gateway restart 保持连续
- 你希望 `TODO.md` 和 `memory/active-task.md` 始终对得上

如果只是一次性的小活，就别上这个。那属于开着塔台指挥自行车。

## 工作流概览

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
- 当优先级、阻塞、重要 ID、下一步发生实质变化时，同步更新

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

## 示例场景

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
4. 明确告诉用户：当前优先级已经变化，现在在处理什么

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

## 和小技能的关系

这个仓是总包，不代表其他仓没用。

相关但非必需的技能：

- `task-orchestrator`：偏任务编排与优先级策略  
  <https://github.com/ruanrrn/task-orchestrator>
- `task-state-sync`：偏连续性文件维护与状态落盘  
  <https://github.com/ruanrrn/task-state-sync>

如果你只想要其中一个窄能力，用小 skill 就够；如果你想要整套工作模型，用这个总 skill。

## 仓库内容

- `multi-task-continuity/` - skill 源码
- `dist/multi-task-continuity.skill` - 可直接导入的打包产物

## 安装

两种方式都可以：

1. 直接导入 `dist/multi-task-continuity.skill`
2. 把 `multi-task-continuity/` 复制到你的 skills 目录，按源码方式使用

## 仓库结构

```text
multi-task-continuity/
├── LICENSE
├── README.md
├── README.zh-CN.md
├── multi-task-continuity/
│   └── SKILL.md
└── dist/
    └── multi-task-continuity.skill
```

## 仓库地址

- GitHub: `https://github.com/ruanrrn/multi-task-continuity`
- License: MIT
