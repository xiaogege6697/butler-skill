# 管家 Skill (Butler)

> 强模型 × 少量精用（决策） + 弱模型 × 大量铺开（执行） = 最高性价比

适用于任何支持"主对话+子任务"机制的 AI 平台（OpenClaw / Claude Code / Codex 等）。

## 核心理念

**选择大于努力。** 把昂贵的强算力用在决策和复盘上，大量重复执行交给廉价的弱模型。

## 包含什么

- `SKILL.md` — 核心工作流（触发词、检查点、踩坑教训、红线）
- `references/platform-adapters.md` — 各平台适配参考
- `references/dispatch-templates.md` — 5种常用派发模板（含无状态弱模型落盘）
- `references/stateless-worker.md` — 无状态弱模型（MiMo/DeepSeek 等）协作手法

## 关键特性

- **开工前检查点** — 模型可用性、API Key、外部服务验证
- **6个执行流程检查点** — CP1~CP5 + CP3.5精度守恒
- **3类Subagent角色** — 侦察兵/临时工/专家
- **5条核心踩坑教训** — 来自20+天真实实践的教训
- **7条红线** — 违反必出问题
- **无状态弱模型协作** — MiMo/DeepSeek 等 + shell 重定向落盘 + 上下文卫生（产出不进主线程上下文）

## 使用

复制 `butler/` 目录到你的 skills 路径下即可。各平台路径见 `references/platform-adapters.md`。

## License

MIT
