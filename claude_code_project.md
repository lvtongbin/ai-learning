# Claude Code 工程化实践

> AI编程真正的价值不在于会用 AI，而在于能把"用 AI 的方式"工程化——让经验可沉淀、可复用、可共享。

---

## 从 Chat 到 Engineering

之前主流的 AI 辅助编码方式：在对话框中描述需求，获取代码后手动复制粘贴并调整，下次遇到类似需求时重复同样的流程。

这是 **Chat 模式**。它的问题不是效率低——它确实比手写快。问题是：**经验没有沉淀**。

| 维度 | Chat 模式 | 工程化模式 |
|------|-----------|-----------|
| 经验存储 | 在人脑里 | 在 `.claude/` 目录里 |
| 可复制性 | 不可复制，换个人从零开始 | `git clone` 即可复用 |
| 团队共享 | 口口相传，或者 Notion 文档 | 版本控制 + plugin marketplace |
| 质量一致性 | 取决于写 prompt 的人的水平 | Skills 保证下限 |
| 持续优化 | 每次对话都是一次性的 | 迭代 skills / agents / hooks |

**`.claude/` 目录 = 团队的 AI 工程化资产**

```
.claude/
├── agents/          # 代理（自主任务）
│   └── name.md      # 代理说明
├── commands/        # 命令（手动触发）
│   └── name.md
├── skills/          # 技巧
│   └── name/        # 技巧名称
│       └── SKILL.md
├── hooks/            # 钩子
└── scripts/          # 脚本（尽可能低环境依赖）
```

该目录随代码仓库一同纳入版本控制，团队成员执行 `git pull` 后即可直接使用，也可发布为插件实现全员共享。

---

## 底层技术：Memory

Memory 是 Claude Code 的持久化上下文机制。`CLAUDE.md` 是项目与Claude Code签订的初始契约——涵盖项目背景、开发规范、代码风格及高风险注意事项。Claude Code在每次会话启动时自动加载，无需在对话中重复说明。

三层级 CLAUDE.md，按优先级依次读取：

| 路径 | 作用范围 | 优先级 |
|------|----------|--------|
| `~/.claude/CLAUDE.md` | 全局（所有项目共用，但基本不用） | 最先读取 |
| 项目根目录/`CLAUDE.md` | 项目级（当前项目） | 第二优先 |
| 项目根目录/`.claude/rules/*.md` | 模块级（特定目录） | 最晚读取 |

有了 Memory，Claude Code的使用模式就从被动变为主动：

| 场景 | 被动使用 | 主动驾驭 |
|------|----------|----------|
| 跑单测 | 执行单元测试 | 配置 hook，每次修改代码自动执行测试 |
| 写文档 | 以 Markdown 格式生成指定文档 | 配置 skill，Claude Code 识别文档需求时自动应用文档规范 |
| code review | 审查指定代码段是否存在问题 | 配置 CR sub agent，每次 MR 自动审查 |

---

## 扩展层：Skills

### Skills结构

Skill 是一份"可操作知识"，本质上就是一个 **SOP**。它具备：

- **名称**（`name`：名称，关键词）
- **触发条件**（`description`：什么时候需要这份知识）
- **执行流程**（正文：获取知识后的分步操作说明）
- **质量标准**（`templates`：输出应该长什么样）
- **工具约束**（`allowed-tools`：过程中能用什么，不能用什么）
- **自动检查**（`hooks`：完成后自动验证质量）

### 文件组织模式

当 Skill 有多重类型的资源（知识 + 模板 + 脚本），推荐按功能分类：

```
.claude/skills/my-skill/
├── SKILL.md           # 入口 + 路由（< 500 行）
├── reference/         # 知识库（公式、规范、基准）
├── templates/         # 输出模板（报告、代码骨架）
└── examples/          # 示例集（输入输出样本）
```

### Skill 写法

- name/description：是什么
- argument-hint/examples：怎么用
- parameters：参数结构
- steps：执行流程
- output-format：输出样式
- subagent/timeout/tags：调度配置

示例：

```markdown
name: file_write
description: 向指定文件写入内容
type: action
argument-hint: 文件名 + 内容，例如：main.py 写入print("hello")
input-required: true
parameters:
  - name: filename
    type: string
    required: true
  - name: content
    type: string
    required: true
steps:
  1. 解析文件名和内容
  2. 检查路径安全性
  3. 执行写入
  4. 返回结果
output-format: text
tags: [file, write, edit]
subagent: file_agent
timeout: 30
enabled: true
```

### Skill 怎么写

Skill 的质量瓶颈不在格式，在判断力。格式可以让 AI 写，但"这个场景值不值得固化成 Skill"只有人能判断。

**什么值得写成 Skill？三个维度，都是 yes 才写：**

- **重复性**：这个流程你是否已经在脑子里执行过 3 次以上？
- **专业性**：这里有你的团队/项目特有的知识，AI 默认不知道？
- **可描述性**：你能用一段话说清楚"什么时候触发、做什么"？

**人 + AI 协作流程：**

1. **人定方向**：描述场景、触发条件、期望输出
2. **AI 生成草稿**：根据描述生成完整 Skill 文件
3. **人审核两处**：`description`（触发条件准不准）+ 正文流程（关键步骤有没有遗漏）

审核 `description` 比审核正文更重要——触发条件错了，再好的正文也没用。

---

## 扩展层：SubAgents

SubAgents 是 Claude Code 中最易被忽视的能力。它用于独立完成专项任务，最适合**隔离执行**——日志查询、资源检索、需要特定权限的操作。

本质上是为了解决：**单一 Agent 的上下文、权限与职责无法无限膨胀**的问题。

### 工程价值：隔离 / 约束 / 复用

**隔离 —— 解决上下文污染问题**

主对话只看到子代理的结论，而不需要承载 500 行搜索输出日志以及分析过程。你的对话不会因为一次检索就被中间输出吞没。

```
主对话上下文：
┌──────────────────────────────────────────────┐
│ 用户：帮我看看这个 bug                          │
│ Claude：正在分析...                             │
│ [子代理返回：发现 3 个相关文件]                   │
│ Claude：问题定位在以下三个文件...                 │
└──────────────────────────────────────────────┘

子代理上下文（独立的，执行完就释放）：
┌─────────────────────────────────────────┐
│ 任务：查找 bug 相关文件                    │
│ [搜索输出 500 行日志]                     │
│ [分析过程...]                            │
│ 结论：3 个相关文件                         │
└─────────────────────────────────────────┘
```

**约束 —— 解决行为不可控问题**

通过工具权限限制角色职责。以 CR 子代理为例，其职责仅限于审查，不应具备修改代码的能力：

```markdown
# 只读型子代理（cr）
tools: Read, Grep, Glob

# 开发型子代理
tools: Read, Write, Edit, Bash

# 调研型子代理
tools: Read, WebFetch, WebSearch
```

**复用 —— 解决经验无法沉淀的问题**

子代理配置保存在文件中，版本控制、跨项目复用、渐进式优化：

```
.claude/agents/
├── test-runner.md      # 测试运行专员
├── code-reviewer.md    # 代码审查专员
├── log-analyzer.md     # 日志分析专员
└── bug-fixer.md        # Bug 修复专员
```

### 案例：Bug 修复流水线

每个代理职责清晰，每个节点可被单独替换 / 回滚 / 审计：

```
Locator → Analyzer → Fixer → Verifier
  定位      分析原因   修复    验证结果
```

- **Locator**：只回答"在哪"，不建议修复方案
- **Analyzer**：只回答"为什么"，不实现修复
- **Fixer**：执行最小化更改，考虑回滚方案（`tools: Read, Edit, Write`）
- **Verifier**：执行测试验证（`tools: Read, Bash, Grep`）

### 案例：影响面分析子代理

```yaml
---
name: impact-analyzer
description: Analyze the impact scope of code changes on the full call chain.
             Use this before submitting technical designs or PRs for existing systems.
tools: Read, Grep, Glob, Bash
model: sonnet
permissionMode: plan
skills:
  - chain-knowledge    # 链路拓扑和 SLA 约束（启动时自动注入）
  - recent-incidents   # 近期事故记录
---
```

注意 `skills` 字段：子代理启动时系统自动注入 skill 的完整内容，链路知识在工作开始之前就已经在上下文里了。

> **注意**：子代理不会继承主 agent 里可用的 skill，必须在子代理配置中显式声明。

| 方式 | 可靠性 |
|------|--------|
| prompt 要求"去读某文件" | 不稳定，子代理可能忘记读、找不到文件 |
| `skills` 字段预加载 | 可靠，知识一定在上下文中 |

---

## 扩展层：Hooks

Hooks 在特定事件触发时自动执行，是流程自动化的关键。

```
事件：Claude Code 即将执行 git commit 操作
hook：自动检查 git staged 文件内是否有安全敏感内容
结果：发现敏感内容，阻止 commit 并警告
```

### 案例：Playwright 视觉验证闭环

前端代码的"正确性"很大程度上是视觉的——按钮的位置对不对、颜色对不对、间距对不对。这些东西没法用单元测试的 pass/fail 来表达。

传统前端 AI-Coding 流程的断层：

```
AI 写代码 → ??? → 人肉打开浏览器 → "这个按钮偏了" → 手动反馈给 AI
```

解法：让 AI 自主截图、自主分析、自主判断结果是否符合预期。

```
AI 写代码 → Playwright 截图 → AI 读截图自我验证 → 自动修正
```

技术方案：
- **Playwright** 作为截图引擎，启动无头浏览器访问页面，输出 PNG 截图
- **Claude** 通过 Read 工具读取截图文件，利用多模态能力"看"截图内容
- AI 对比截图与需求描述，判断 UI 是否符合预期
- 不符合则修改代码，重新截图，再次验证

---

## 扩展层：Computer Use

Computer Use 使 AI 能够像人类一样直接操作计算机。

传统 AI 仅能响应问题。Computer Use 赋予 AI 操作能力——感知屏幕状态、规划下一步动作、执行点击与输入、观察执行结果，循环迭代直至任务完成。

### 技术栈

| 层级 | 技术 | 作用 |
|------|------|------|
| 底层协议 | [Chrome DevTools Protocol (CDP)](https://chromedevtools.github.io/devtools-protocol/) | 浏览器自动化基础，暴露 WebSocket 控制接口 |
| 自动化框架 | [Playwright](https://github.com/microsoft/playwright) | 跨浏览器高层 API，自动等待，提供 MCP 服务 |
| AI 集成层 | [browser-use](https://github.com/browser-use/browser-use)<br>[agent-browser](https://github.com/vercel-labs/agent-browser) | 连接 LLM 与浏览器，DOM 提取替代截图，减少幻觉，提供skills |

```python
from browser_use import Agent
from langchain_anthropic import ChatAnthropic

agent = Agent(
    task="打开 Hacker News，告诉我今天排名第一的文章标题",
    llm=ChatAnthropic(model="claude-sonnet-4-6"),
)
await agent.run()
```

```
# 浏览器操作

优先使用agent-browser skill 操作浏览器 安装文档：https://github.com/vercel-labs/agent-browser

```

> 延伸阅读：[OpenAI Computer-Using Agent](https://openai.com/zh-Hans-CN/index/computer-using-agent/)

---

## 组合与总结

### 案例：Code Review 流水线

```
GitHub Actions 监听 MR 创建
  → claude --headless（Headless 模式触发）
    → 调用 code-reviewer sub agent（隔离 review 上下文）
      → sub agent 使用 code-review skill（按团队规范审查）
        → hook 记录审查日志（便于调试和审计）
          → 结果通过 MCP 发送到企业微信（通知相关人员）
```

### Handoffs 模式（Oncall 场景）

适用于有明确阶段划分的流程型场景，每个阶段有显式的完成条件（Exit Criteria）：

```
前台接待 agent → 技术支持 agent → 高级工程师 agent
  信息收集         技术诊断          深度修复 + 报告
```

每个阶段必须满足前置条件才能进入下一阶段，否则卡在当前阶段。这是 Handoffs 能稳定运行的核心。

### 三层递进

把所有内容串起来，本质上是三个层次：

```
Skills   → 把经验 SOP 化   → 解决"怎么做"
Agents   → 把职责隔离化    → 解决"谁来做"
Hooks    → 把流程自动化    → 解决"什么时候做"
```

| 组件 | 触发方式 | 触发者 | 确定性 |
|------|----------|--------|--------|
| Commands | 用户输入 `/xxx` | 用户 | 100% 确定 |
| Skills | 语义理解 | Claude Code | 概率性 |
| Sub Agents | 用户指定或 Claude Code 自主决定 | 用户和 Claude Code | 可控 |
| Hooks | 事件触发 | 系统 | 100% 确定 |

三层叠加，构成完整的 AI-Coding 工程化体系。支持版本控制、团队共享与持续迭代优化。

这才是 AI-Coding 的核心壁垒——不在于是否会用 AI，而在于能否将使用 AI 的方式本身工程化。


