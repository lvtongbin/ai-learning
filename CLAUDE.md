# AI 编程交流网站

一个静态课程展示网站，暗色科幻风格，用于展示 AI 编程交流课程内容。

## 项目结构

```
index.html                  # 课程主页，6 个模块卡片网格
learn_claude_code.html      # Module 01 详情页，动态渲染 Markdown
learn_claude_code.md        # Module 01 课程内容（Markdown）
claude_code_project.html    # Module 03 详情页
claude_code_project.md      # Module 03 课程内容
harness_engineering.html    # Module 04 详情页
harness_engineering.md      # Module 04 课程内容
docs/superpowers/plans/     # 实现计划文档
```

## 技术栈

- 纯 HTML/CSS/JS，无构建工具，无框架
- marked.js（CDN）渲染 Markdown
- Google Fonts：Orbitron（标题）+ Noto Sans SC（正文）

## 设计系统

CSS 变量（所有页面共用）：

```css
--bg: #0a0a0f
--surface: #12121a
--border: rgba(0, 212, 255, 0.15)
--cyan: #00d4ff
--purple: #a855f7
--text: #e2e8f0
--muted: #64748b
```

背景：固定定位的 40px 网格线（极低透明度青色）。

## 开发规范

- 新模块详情页复用 `learn_claude_code.html` 的结构和样式
- 模块卡片链接使用 `<a class="card card--link" href="...">` 替换 `<div class="card">`
- 不引入任何构建工具或 npm 依赖，保持纯静态
- `fetch` 加载 `.md` 文件时加 `?v=` + `Date.now()` 防止浏览器缓存
