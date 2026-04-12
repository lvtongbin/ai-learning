# Learn Claude Code

Claude Code 是 Anthropic 官方推出的 AI 编程助手 CLI 工具，直接运行在终端中，能够理解整个代码库上下文，帮助开发者完成从代码编写、调试到重构的全流程任务。

---

## Claude Code 基础概念

### 它是什么

Claude Code 是一个在终端运行的 agentic 编程工具。与普通聊天式 AI 不同，它可以：

- 直接读写本地文件
- 执行 shell 命令
- 搜索代码库
- 调用外部工具（MCP）
- 自主完成多步骤任务

> 参考文档：[Learn Claude Code](https://learn.shareai.run/zh/)

### 工作原理

Claude Code 通过系统提示将你的工具、文件内容、shell 输出等上下文打包发送给模型，模型返回工具调用指令，客户端执行后再将结果反馈给模型，形成一个 agentic 循环。

```
用户输入 → 构建上下文 → 发送给 Claude → 解析工具调用 → 执行操作 → 返回结果 → 循环
```

每一轮对话都会携带完整的对话历史和工具结果，这就是为什么 Claude Code 能"记住"之前做了什么。

> 参考文档：[Claude Code 官方文档](https://code.claude.com/docs/zh-CN/how-claude-code-works)

### 模型选择与 Haiku 的作用

Claude Code 支持多个模型，不同场景下选择不同模型可以平衡速度与成本：

| 模型 | 特点 | 适用场景 |
|------|------|----------|
| Claude Opus 4.6 | 最强推理能力 | 复杂架构设计、难题调试 |
| Claude Sonnet 4.6 | 能力与速度均衡 | 日常编程任务（默认推荐） |
| Claude Haiku 4.5 | 极速、低成本 | 简单补全、快速问答 |

**Haiku 的典型用途**：在 agentic 任务中，Claude Code 会用 Haiku 处理一些"子任务"，比如判断某个文件是否相关、提取简单信息等，从而降低整体 token 消耗。

## 命令速查

### 基础操作

```bash
# 启动交互模式
claude

# 直接执行单次任务（非交互）
claude -p "帮我解释这段代码"

# 指定工作目录
claude --cwd /path/to/project

# 查看帮助
claude --help
```

### 对话中的斜杠命令

| 命令 | 说明 |
|------|------|
| `/init` | 初始化项目，生成 CLAUDE.md |
| `/clear` | 清空当前对话上下文 |
| `/compact` | 压缩对话历史，节省 token |
| `/cost` | 查看本次对话的 token 消耗 |
| `/model` | 切换使用的模型 |
| `/review` | 对当前改动进行代码审查 |
| `/simplify` | 三合一代码审查，同时启动3个angent，分别从代码复用、代码质量和运行效率审查代码 |
| `/commit` | 生成 commit message 并提交 |
| `/btw` | 并行提问，让claude在干活的时候插入一个问题 |
| `/plugin` | 插件管理，添加/列表 |
| `/export` | 导出对话为markdown |

### CLAUDE.md 项目配置

在项目根目录创建 `CLAUDE.md`，Claude Code 每次启动都会读取它作为上下文：

```markdown
# 项目名称

## 技术栈
- Node.js + TypeScript
- PostgreSQL

## 开发规范
- 使用 ESLint + Prettier
- 测试框架：Vitest
- 提交前必须通过所有测试

## 常用命令
- 启动开发服务器：npm run dev
- 运行测试：npm test
```

### 权限控制

Claude Code 在执行危险操作前会请求确认。可以通过配置调整：

```bash
# 允许所有操作（谨慎使用）
claude --dangerously-skip-permissions

# 在 settings.json 中配置允许的工具
```

---

## OpenCode：开源替代方案

[OpenCode](https://github.com/code-yeongyu/oh-my-openagent) 是一个开源的 Claude Code 替代客户端，支持接入多种 AI 服务，适合希望自定义或降低成本的开发者。

### 为什么考虑 OpenCode

- **成本控制**：可以接入 GitHub Copilot、Kiro 等订阅服务，复用已有订阅
- **开源可控**：代码透明，可自行修改和部署
- **多模型支持**：不绑定单一 AI 提供商

### 常见组合方案

**方案一：OpenCode + OMO + GitHub Copilot 订阅**

通过 [oh-my-openagent (OMO)](https://github.com/code-yeongyu/oh-my-openagent) 将 GitHub Copilot 订阅转为兼容 OpenAI API 格式，再接入 OpenCode。

适合：已有 GitHub Copilot 订阅的开发者，可以零额外成本使用 Claude 模型。

**方案二：OpenCode + OMO + Kiro 订阅**

通过 `@zhafron/opencode-kiro-auth` 插件接入 Kiro 订阅。

适合：Kiro IDE 用户，在终端中复用 Kiro 的 AI 能力。

### 选择建议

| 场景 | 推荐方案 |
|------|----------|
| 初学者、追求稳定 | 官方 Claude Code |
| 已有 Copilot 订阅 | OpenCode + OMO + Copilot |
| 重度用户、需要省钱 | OpenCode + 自定义接入 |

---

## Skills：给 AI 装上技能包

Skills（技能）是一种结构化的提示词框架，让 Claude Code 在特定任务上遵循固定的工作流程，从而获得更稳定、更专业的输出。

### 渐进式披露（Progressive Disclosure）

渐进式披露（Progressive Disclosure） 是一种分层、按需、动态的信息供给与能力释放架构，核心是最小化初始信息、最大化按需加载，避免一次性全量输入导致的效率、成本与安全问题。

**传统 System Prompt vs. Agent Skill 架构对比**

| 维度 | 传统 System Prompt 模式 | Agent Skill 模式 |
|------|------------------------|-----------------|
| 规则载体 | 纯文本，随会话发送 | 本地结构化文件（.md / .py） |
| 上下文占用 | 全量占用（无论是否用到，所有规则都在窗口内） | 按需占用（仅加载被触发的技能规则） |
| 可维护性 | 极低（修改一处需测试整体影响） | 高（模块化独立封装） |
| 执行能力 | 仅限于文本生成 | 原生支持脚本执行与文件操作 |


### 什么是 Skills

Skills 本质上是一段 Markdown 文件，描述了"遇到某类任务时，AI 应该按什么步骤来做"。当 Claude Code 加载某个 skill 后，它会严格按照 skill 定义的流程工作。

**没有 Skills**：每次都要在提示词里重复说明要求，输出质量不稳定。

**有了 Skills**：一次定义，反复复用，AI 行为可预期。

### Superpowers：Skills 框架

[Superpowers](https://github.com/obra/superpowers) 是目前最成熟的 Claude Code Skills 框架，提供了一套完整的开发方法论：

- `brainstorming`：需求分析与方案设计
- `writing-plans`：将设计转化为实现计划
- `test-driven-development`：TDD 工作流
- `systematic-debugging`：系统化调试流程
- `code-review`：代码审查

安装方式：

```bash
# Claude Code
/plugin marketplace add obra/superpowers-marketplace

```

### UI 优化 Skill

[ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) 是一个专注于 UI/UX 设计的 skill，能让 Claude Code 在生成前端代码时遵循专业的设计原则：

- 自动考虑响应式布局
- 遵循无障碍访问标准
- 输出符合现代审美的组件

安装方式：

```bash
# Claude Code
/plugin marketplace add nextlevelbuilder/ui-ux-pro-max-skill

```

**使用示例**：

```
使用 ui-ux-pro-max-skill，帮我用 Vue 实现一个黏土风格的 TODO LIST，
配色要显眼、引人入胜。
```

### Skills 资源

- Superpowers 框架：[github.com/obra/superpowers](https://github.com/obra/superpowers)
- UI/UX Skill：[github.com/nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill)

---

## 进阶技巧

### 代码是ai的产物

[voice-input-src](https://github.com/yetone/voice-input-src) 是一个为 Claude Code 添加语音输入能力的工具，让你可以用说话代替打字来下达指令，在长时间编程时特别有用。**其核心就是一段prompt**。

### 两种使用 Claude Code 的思路

**思路一：对话驱动（Chat-driven）**

像聊天一样逐步引导 Claude Code 完成任务。适合探索性工作，你保持对每一步的控制权。

```bash
# 示例：逐步引导
> 帮我看看 src/auth.ts 有什么问题
> 好，现在修复第二个问题
> 写一个测试覆盖这个修复
```

**思路二：任务驱动（Task-driven）**

一次性给出完整需求，让 Claude Code 自主规划并执行。适合明确的功能开发，效率更高。

```bash
# 示例：一次性任务
> 在用户登录流程中添加 JWT 刷新机制，
  要求：token 过期前 5 分钟自动刷新，
  失败时跳转登录页，并写好单元测试
```

### 构造自我验证的运行环境

让 Claude Code 在完成任务后自动验证结果，是提升可靠性的关键技巧。

**方法**：在提示词中明确要求验证步骤：

```
实现功能后，请：
1. 运行相关测试确认通过
2. 用 curl 或脚本验证接口返回正确
3. 如果验证失败，自动修复后重新验证
```

这样 Claude Code 就会形成"实现 → 验证 → 修复"的自闭环，减少你手动检查的次数。

### 高效使用建议

1. **善用 CLAUDE.md**：把项目规范、常用命令、技术栈写进去，每次对话都自动生效
2. **拆分大任务**：复杂功能分成多个小任务，每个任务验证通过后再进行下一个
3. **明确验证条件**：告诉 Claude Code 如何判断任务完成，让它自我验证
4. **保持上下文干净**：用 `/clear` 或 `/compact` 避免上下文过长导致质量下降
5. **利用 git 做安全网**：在 Claude Code 大规模修改前先 commit，出问题可以回滚

### 不同模型的选择时机

- **复杂架构讨论、难以复现的 bug**：用 Opus，推理能力最强
- **日常功能开发、代码重构**：用 Sonnet，性价比最高
- **快速问答、简单补全**：用 Haiku，速度快成本低
