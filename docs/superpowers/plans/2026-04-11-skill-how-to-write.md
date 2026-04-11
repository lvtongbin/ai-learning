# Skill 怎么写 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在 `claude_code_project.md` 的 `### Skill 怎么写` 章节补充内容，传达"写什么 > 怎么写，判断力来自人"的核心思想，并给出判断标准和协作流程。

**Architecture:** 直接编辑 `claude_code_project.md`，在 `### Skill 怎么写` 标题下插入正文内容。无需新建文件，无需修改 HTML。

**Tech Stack:** 纯 Markdown 文本编辑

---

### Task 1: 写入 `### Skill 怎么写` 正文内容

**Files:**
- Modify: `claude_code_project.md:123-125`

- [ ] **Step 1: 确认插入位置**

读取 `claude_code_project.md` 第 123-127 行，确认 `### Skill 怎么写` 标题后是空行，内容为空。

- [ ] **Step 2: 插入正文内容**

将以下内容替换 `claude_code_project.md` 中 `### Skill 怎么写` 标题后的空行（第 124-125 行）：

```markdown
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
```

- [ ] **Step 3: 验证渲染效果**

在浏览器打开 `claude_code_project.html`，确认新增内容在 Skills 章节下正确显示，格式无误。

- [ ] **Step 4: Commit**

```bash
git add claude_code_project.md
git commit -m "docs: add Skill 怎么写 section content"
```
