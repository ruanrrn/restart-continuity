# Restart Continuity

[English](README.md) | 简体中文

![Restart Continuity banner](assets/social-preview.svg)

![OpenClaw Skill](https://img.shields.io/badge/OpenClaw-Skill-2D3142?style=flat-square)
![Focus-Restart Recovery](https://img.shields.io/badge/Focus-Restart%20Recovery-EF8354?style=flat-square&labelColor=2D3142)
![Works-Standalone](https://img.shields.io/badge/Works-Standalone-EAEAEA?style=flat-square&labelColor=4F5D75)
![Artifact-.skill Included](https://img.shields.io/badge/Artifact-.skill%20Included-BFC0C0?style=flat-square&labelColor=4F5D75)
![README-Bilingual](https://img.shields.io/badge/README-Bilingual-EAEAEA?style=flat-square&labelColor=4F5D75)
![License-MIT](https://img.shields.io/badge/License-MIT-EAEAEA?style=flat-square&labelColor=2D3142)

让 OpenClaw 在 gateway 重启后继续干活，而不是像失忆了一样让用户重新解释一遍。

## Quick pitch

把正在进行的任务在重启前记清楚、重启后接回来，别把“继续”这件事甩给用户。
保住顶层任务、关键 ID 和下一步动作，回来就接着干。

## Why this exists

重启不该等于上下文清零。现实里最烦人的情况就是：gateway 半路重启，代理像被雷劈了一样忘了自己刚才在做什么，然后还得用户手动提醒“继续”。这不叫中断恢复，这叫流程摆烂。

OpenClaw 虽然有内建的重启后 ping，但光靠那个还不够。要想恢复靠谱，必须在重启前把状态记扎实，重启后第一时间把正确的任务续上。

`restart-continuity` 干的就是这件事。

它给代理一条很聚焦的工作流，用来：

- 在重启前持久化当前顶层任务
- 记录重启后还需要的关键 ID
- 为主动重启安排兜底提醒
- 重启后优先恢复正确任务
- 主动告诉用户恢复了什么

## Works independently

`restart-continuity` 单独用就成立。

哪怕你不引入更大的多任务编排或状态同步技能，它也已经能明显改善：

- 重启前准备
- 重启后恢复准确度
- fallback cron 处理
- 顶层任务连续性
- 重启后的主动用户更新

别的仓可以互补，但这个仓本身不靠它们才讲得通。

## Family role

在这一组 repo 里，`restart-continuity` 是专门守重启边界的那一个。

当核心问题发生在“重启前后这道坎”上时，就该用它：保住顶层任务、跨重启带上关键 ID、回来后先恢复正确 lane。

别把它膨胀成泛化状态维护包。如果真正的问题是工作进行中 `TODO.md` 和 `memory/active-task.md` 会漂，那是 `task-state-sync` 的地盘；如果问题是整个多任务操作模型，那是 `multi-task-continuity` 的范围。

## What the skill teaches

这个 skill 会要求代理：

- 在重启前更新 `memory/active-task.md`，写清目标、状态、阻塞、下一步和重启后要发给用户的话
- 记录审批 ID、job ID、session、进程、端口、文件路径等活跃标识
- 对主动重启安排一次性的 cron fallback，并保存返回的 `jobId`
- 重启后在安全前提下立刻恢复顶层未完成任务
- 在第一条实质性回复里告诉用户恢复了什么、接下来做什么
- 恢复成功后清理过期的 fallback 状态

## When to use it

这些场景就该上 `restart-continuity`：

- 任务会跨越重启，无论计划内还是计划外
- 需要重启 gateway，但工作不能断片
- 助手必须主动继续重启前的任务
- 关键 ID 必须跨重启保留下来
- 长流程任务承受不起重启失忆

## Example behavior

### Example 1: 进行中的任务遇到计划内重启

比如正在发布仓库，中途必须重启 gateway。

靠谱的代理应该：

1. 先把真实顶层任务写进 `memory/active-task.md`
2. 记录关键 ID 和精确下一步
3. 安排 fallback cron job
4. 把 fallback `jobId` 写回 `memory/active-task.md`
5. 确认恢复线索真实存在后再重启

### Example 2: 重启后的恢复

gateway 在一次计划内重启后重新上线。

靠谱的代理应该：

1. 在做别的事之前先读 `memory/active-task.md`
2. 只要安全且不阻塞，就立刻恢复顶层未完成任务
3. 在第一条实质性回复里发出预先准备好的用户更新
4. 确认恢复成功后移除 fallback cron job

### Example 3: 重启后把任务收尾

恢复后的任务顺利完成。

靠谱的代理应该：

1. 如果没有新的顶层任务，就清空 `memory/active-task.md`
2. 如果还有后续任务，就重写它，让下一个未完成任务变成新的顶层任务
3. 不要留下过期审批 ID 或陈旧的下一步

## Related skills

这些是相关项，不是依赖项：

- `task-orchestrator`：补上多任务调度和优先级 - <https://github.com/ruanrrn/task-orchestrator>
- `task-state-sync`：保持连续性文件在工作过程中不漂 - <https://github.com/ruanrrn/task-state-sync>
- `multi-task-continuity`：把编排、状态同步和重启恢复打成一个总包 - <https://github.com/ruanrrn/multi-task-continuity>

如果真正的痛点就是重启恢复，这个仓单独用就够。

## Social preview

建议 social preview 资源：`assets/social-preview.svg`

建议一句话文案：

> Resume active work cleanly after restarts instead of making users repeat themselves.

GitHub 备注：

- 当前公开的 `gh` CLI 和 GraphQL `UpdateRepositoryInput` 都没有可写的 custom social preview 字段。
- 如果你要把这张图真正设成仓库 social preview，只能去 GitHub 仓库设置页手动上传 `assets/social-preview.svg`。

## What you get

- `restart-continuity/` - skill 源码
- `dist/restart-continuity.skill` - 可直接导入的打包产物

## Install

两种方式都可以：

1. 直接导入 `dist/restart-continuity.skill`
2. 把 `restart-continuity/` 复制到你的 skills 目录，按源码方式使用

## Repository layout

```text
restart-continuity/
├── LICENSE
├── README.md
├── README.zh-CN.md
├── CONTRIBUTING.md
├── assets/
│   └── social-preview.svg
├── restart-continuity/
│   └── SKILL.md
└── dist/
    └── restart-continuity.skill
```

## Contributing

见 `CONTRIBUTING.md`，里面写了贡献范围、PR 预期，以及怎么保证这个仓继续专注于重启恢复，而不是演变成一个什么都往里塞的 continuity 杂货铺。

## Release hygiene

- skill 有实质改动后，要重新生成 `dist/restart-continuity.skill`
- 仓库描述要和 skill 的触发语义保持一致
- 仓库保持克制和实用，别塞无关调试杂物

## Repository

- GitHub: `https://github.com/ruanrrn/restart-continuity`
- License: MIT
