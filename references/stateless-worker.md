# Stateless Worker — 无状态弱模型（MiMo / DeepSeek 等）协作手法

> Claude Code 下派执行类子任务，优先用**无状态弱模型**（MiMo、DeepSeek 等，经 Bash 调 API），而非 Task tool。Task tool 的 model 只接受 Claude 系（本环境=glm-5.2 强模型），派执行活反而烧强算力。无状态弱模型便宜，适合铺量执行。本文以已验证的 `mimo_worker.py` 为示例入口；DeepSeek 等其他便宜模型同理，替换调用入口即可（DeepSeek 走 OpenAI 兼容端点）。

## 1. 无状态弱模型的本质与限制（MiMo / DeepSeek 等）

- **无状态 API 调用**（如 MiMo 经 `mimo_worker.py`、DeepSeek 经 OpenAI 兼容端点），不是常驻 agent。
- **不能读写文件、不能联网、无记忆**；每次调用要重新给全上下文。
- MiMo 调用：`~/.claude/venv/bin/python ~/.claude/scripts/mimo_worker.py "<prompt>" [model]`，默认 `model=mimo-v2.5-pro`。DeepSeek 等替换为对应入口。
- 适合：写代码 / 文档 / 调研（凭训练知识）/ 翻译 / 摘要 / 分类 / 格式转换 / 初筛。
- 不适合（留强模型）：架构决策、接口拍板、审核终判、复杂推理、真跑验证。

## 2. 产出落盘（核心手法：产出不进主线程上下文）

弱模型不能写文件，但它的 stdout 可直接 shell 重定向到文件：

```bash
~/.claude/venv/bin/python ~/.claude/scripts/mimo_worker.py "<prompt：只输出纯代码/纯md>" > 目标文件
echo "bytes: $(wc -c < 目标文件)"
```

要点：
- prompt 必须严格要求"只输出纯代码/纯 md，禁止 markdown 围栏（不输出三反引号），禁止解释前后缀，第一行就是内容开头"——否则围栏/解释会污染文件。
- 重定向后主线程**只回显字节数**确认生成，不看原文。
- 落盘后用 `py_compile` 或真跑验证文件是否被围栏/解释污染。

## 3. 喂文件给弱模型（读文件不进主线程上下文）

要让弱模型读现有文件（审代码、提取指标、改写文档），用命令替换把文件内容拼进 prompt：

```bash
P=$(cat <<'EOF'
<指令：读下方文件，做 XX，只回结论摘要>
EOF
)
~/.claude/venv/bin/python ~/.claude/scripts/mimo_worker.py "$P$(cat 待读文件)"
```

要点：`cat` 的输出进弱模型的 prompt，不回显到主线程；主线程只收结论。

> ⚠️ 踩坑：若 prompt 内容里本身含 `$()`、反引号等示例文本，`P=$(cat <<'EOF' ... EOF)` 这种 heredoc 嵌套在某些 shell 的 eval 下会解析出错（如 `parse error near ')'`）。此时改用**临时文件法**：把 prompt 写到 `/tmp/p.txt`（用 Write 工具），再执行 `mimo_worker.py "$(cat /tmp/p.txt)"`。

## 4. 并行多路

同一轮发多个 Bash 调用，每个独立弱模型（不同 prompt/文件），并行跑。适用：多文件同时生成、benchmark + 调研并行、多方向侦察。

## 5. 上下文卫生（主线程上下文是稀缺资源）

- 文件原文 dump / 探索过程 / 长输出**不得灌入**主线程上下文。
- 读文件、探代码、查证、跑测试——外包给弱模型，只回收结论摘要（做什么 / 签名 / 关键约束 / 通过与否）。
- 主线程只持有：任务目标 + 精炼结论 + 决策点。
- 审核需看代码时，让弱模型回"待审 diff + 一句改动说明"，只看关键部分。

## 6. 轻量审核（不读全文）

- `grep` 抽标题 / 表格行 / 关键判断词，验证结构与核心结论。
- 代码靠 smoke check 真跑（`compileall`、`analyze`、`pytest`），跑通比逐行审更有效。
- 弱模型行号常错（如报第 104 行实际第 86 行），定位以主线程 `grep` / `Read` 核实为准。

## 7. 弱模型局限清单

- 无文件系统访问（读写靠主线程重定向 / 落盘）。
- 无联网（调研凭训练知识，要最新 / 准确用 Tavily / WebSearch）。
- 无状态（每次重给上下文，不假设有记忆）。
- 行号 / 小数字会错（关键定位主线程核实）。
- 代码一次成功率因任务而异（纯模板代码高，复杂数学低），靠审核 + 真跑兜底。
