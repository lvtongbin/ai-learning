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
├── hooks/           # 钩子
└── scripts/         # 脚本（尽可能低环境依赖）
│   ├── name.sh      # shell脚本
│   └── name.py      # python脚本

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

### Skill 字段说明

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

Skill 的质量瓶颈不在格式，在判断力。内容可以让 AI 写（写skill也是一种skill），但"这个场景值不值得固化成 Skill"需要人为（不一定是人）能判断。

**什么值得写成 Skill？三个维度，都是 yes 才写：**

- **重复性**：这个流程你是否已经在脑子里执行过 3 次以上？
- **专业性**：这里有你的团队/项目特有的知识，AI 默认不知道？
- **可描述性**：你能用一段话说清楚"什么时候触发、做什么、约束、验证、审查、安全"？

**人 + AI 协作流程：**

1. **人定方向**：描述场景、触发条件、期望输出
2. **AI 生成草稿**：根据描述生成完整 Skill 文件
3. **人审核两处**：`description`（触发条件准不准）+ 正文流程（关键步骤有没有遗漏）

> 审核 `description` 比审核正文更重要——触发条件错了，再好的正文也没用。

**生产环境共性模式**

1. **职责单一**：每个 skill 只做一件事
2. **输入输出明确**：每阶段有结构化的 JSON/Markdown 产出
3. **失败快速**：前置检查不通过则阻断后续步骤
4. **可观测性**：操作日志、状态文件、监控指标全程可查
5. **人工确认**：不可逆操作必须用户确认，不自动执行

参考：
- [skill-creator](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md)
- [hermes-agent](https://github.com/NousResearch/hermes-agent)（[skill_manager_tool](https://github.com/NousResearch/hermes-agent/blob/main/tools/skill_manager_tool.py#L654)）

---

## 扩展层：Agents/SubAgents

Agent 是 独立角色+任务执行者，适合**任务执行**——日志查询、资源检索、需要特定权限的操作。


### Agent 字段说明


**核心字段（必须 / 常用）**
- name: Agent 的名字（必须）
- description: 描述（可选），简单说明这个 Agent 是干嘛的
- model: 使用的模型（必填），opus、sonnet...
- tools: 允许使用的工具列表（必填），Read, Write, Edit, Bash, Glob, Search, Web
- system: 自定义系统提示词（可选，替代正文）

**高级字段（很少用）**
- temperature: 温度 0~1
- maxTokens: 最大token
- context: 上下文策略


示例：
```markdown
---
name: CodeMaster
description: 全能代码开发智能体
model: claude-3-5-sonnet-20241022
tools: [Read, Write, Edit, Bash, Glob, Search]
temperature: 0.1
---

# CodeMaster

你是一名资深全栈工程师，专注于编写高质量代码...

## 职责

## 输入格式

## 输出格式

## 约束
```

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

### 生产环境的研发效率和稳定性

**故障排查**

| 插件 | 解决的问题 | 思路 |
|------|-----------|------|
| 线上 Bug 修复 | bug 定位链路长，跨系统排查效率低 | 多阶段 Agent 流水线：收集 → 查日志/监控 → 根因分析 → 生成修复 → 提交 MR |
| 客诉排查 | 用户级功能异常难定位 | 以用户/设备维度为核心，结合日志、数据、配置、代码逻辑综合定位 |

**发布与运维**

| 插件 | 解决的问题 | 思路 |
|------|-----------|------|
| 服务发布编排 | 测试→灰度→全量 多阶段发布繁琐易出错 | 统一编排发布脚本，内置上线前检查和上线后监控 |
| 可观测性查询 | 排查需在日志、监控、链路追踪间反复切换 | 整合三大能力提供统一查询入口，一站式定位线上问题 |
| 数据与中间件查询 | MySQL/Redis/MQ 数据散落多平台 | 封装统一查询技能，支持只读 DB 查询、缓存检查、消息队列消费状态查询 |

**知识与文档**

| 插件 | 解决的问题 | 思路 |
|------|-----------|------|
| 代码理解与文档 | 接手陌生模块成本高，文档缺失 | 扫描 → 分类 → 深挖三阶段流水线，自动输出结构化业务文档 |
| 架构知识检索 | 架构知识分散，难以快速获取 | 通过 Prophet API 语义检索 CodeWiki，快速定位架构和接口文档 |

**开发效率**

| 插件 | 解决的问题 | 思路 |
|------|-----------|------|
| 本地开发联调 | 本地环境搭建复杂，与线上联调困难 | 封装工具链，支持远程配置拉取、Mock 规则管理 |
| 经验复用 | 同类问题反复排查，经验无法沉淀 | 处理后自动提炼经验存储，下次语义检索历史经验直接复用 |

### 三层递进

把所有内容串起来，本质上是三个层次：

```
Skills   → 把经验 SOP 化  → 解决"怎么做"
Agents   → 把职责隔离化    → 解决"谁来做"
Hooks    → 把流程自动化    → 解决"什么时候做"
```

| 组件 | 触发方式 | 触发者 | 确定性 |
|------|----------|--------|--------|
| Commands | 用户输入 `/xxx` | 用户 | 100% 确定 |
| Skills | 语义理解 | Claude Code | 概率性 |
| Agents | 用户指定或 Claude Code 自主决定 | 用户和 Claude Code | 可控 |
| Hooks | 事件触发 | 系统 | 100% 确定 |

三层叠加，构成完整的 AI-Coding 工程化体系。支持版本控制、团队共享与持续迭代优化。

这才是 AI-Coding 的核心壁垒——不在于是否会用 AI，而在于能否将使用 AI 的方式本身工程化。


