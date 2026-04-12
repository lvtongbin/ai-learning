# Module 03 — Claude Code 工程化实践 设计文档

**日期**：2026-04-11  
**状态**：已批准

---

## 目标

为 AI 编程交流网站创建 Module 03 详情页，内容基于《Claude Code 工程化手册》，帮助学员从"Chat 模式"进入"工程化模式"，系统掌握 Skills、SubAgents、Hooks 的设计与组合。

---

## 文件变更

| 文件 | 操作 | 说明 |
|------|------|------|
| `claude_code_ project.md` | 填充内容 | Module 03 课程 Markdown |
| `claude_code_project.html` | 新建 | Module 03 详情页 HTML |
| `index.html` | 更新 Module 03 卡片 | 改为可点击链接，填入标题/描述/标签 |

---

## Markdown 内容结构

### 1. 从 Chat 到 Engineering
- 两种模式对比表格（经验存储、可复制性、团队共享、质量一致性、持续优化）
- 引出 `.claude/` 目录即团队 AI 工程化资产

### 2. 底层技术：Memory
- CLAUDE.md 三级配置（全局 / 项目 / 模块）
- 路径与优先级表格
- 被动使用 vs 主动驾驭对比表格（场景举例）

### 3. 扩展层：Skills
- 什么是 Skill（SOP 化的可操作知识）
- 参考型 vs 任务型对比
- description 写法公式 + 示例
- 动态上下文注入 `!command` 及工程价值对比表格
- 文件组织模式（按功能分类）

### 4. 扩展层：SubAgents
- 工程价值：隔离 / 约束 / 复用
- 权限配置示例（只读型、开发型、调研型）
- Bug 修复流水线案例（Locator → Analyzer → Fixer → Verifier）
- 影响面分析子代理完整示例（含 skills 字段预加载说明）

### 5. 扩展层：Hooks
- 事件触发机制说明
- Playwright 视觉验证闭环案例（前端 AI-Coding 断层问题 → 解法）

### 6. 组合与总结
- Code Review 流水线组合案例（Headless + SubAgent + Skill + Hook + MCP）
- Handoffs 模式（Oncall 场景，三阶段状态机）
- 三层递进总结：Skills → Agents → Hooks
- 结语：Don't Only Chat

---

## HTML 详情页

复用 `learn_claude_code.html` 结构，改动：
- `<title>`：`Claude Code 工程化实践 · AI 编程交流`
- `.module-label`：`MODULE · 03`
- `fetch` 路径：`claude_code_ project.md`

---

## index.html 更新

Module 03 卡片：
- 元素：`<div class="card">` → `<a class="card card--link" href="claude_code_project.html">`
- 状态 badge：`即将开放` → `已开放`
- 标题：`Claude Code 工程化实践`
- 描述：`从 Chat 模式进入工程化模式，掌握 Skills、SubAgents、Hooks 的设计与组合，构建可复用的 AI-Coding 工程体系。`
- 标签：`Skills`、`SubAgents`、`Hooks`
