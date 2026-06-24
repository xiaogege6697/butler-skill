# Platform Adapters — 各平台实现参考

## OpenClaw

```
# 派侦察兵/临时工（隔离上下文）
sessions_spawn(task="...", mode="run")

# 需要当前对话上下文时
sessions_spawn(task="...", context="fork")

# 指定弱模型执行
sessions_spawn(task="...", model="便宜模型名")

# 等待结果
sessions_yield()

# 跨session通信
sessions_send(sessionKey="xxx", message="...")
```

## Claude Code

```
# 派子任务
Task(description="...", prompt="...")
```

## Codex / 通用沙箱

```
启动子进程/沙箱执行，等返回结果
```

## 模型分配建议

| 场景 | 强模型（管家） | 弱模型（执行） |
|------|---------------|---------------|
| 调研分析 | 规划搜索方向 + 验收 | 逐源采集 + 格式化 |
| 内容生产 | 确定主题角度 + 终审 | 草稿生成 + 配图 |
| 代码开发 | 架构设计 + Code Review | 功能实现 + 测试编写 |
| 日常运维 | 判断优先级 + 决策修复 | 执行检查脚本 + 收集日志 |
