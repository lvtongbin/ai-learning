# AI 编程培训网站

一个静态课程展示网站，暗色科幻风格，用于展示 AI 编程培训课程内容。

## 项目结构

```
index.html                  # 课程主页，6 个模块卡片网格
learn_claude_code.html      # Module 01 详情页
learn_claude_code.md        # Module 01 课程内容
claude_code_project.html    # Module 03 详情页
claude_code_project.md      # Module 03 课程内容
harness_engineering.html    # Module 04 详情页
harness_engineering.md      # Module 04 课程内容
```

## 技术栈

- 纯 HTML/CSS/JS，无构建工具，无框架
- [marked.js](https://marked.js.org/)（CDN）渲染 Markdown
- Google Fonts：Orbitron（标题）+ Noto Sans SC（正文）

## 本地开发

启动本地服务：

```bash
python3 -m http.server 8080
```

然后访问 http://localhost:8080

> 各详情页通过 `fetch` 加载对应 `.md` 文件，需要 HTTP 服务才能正常运行，直接打开 HTML 文件会因跨域限制导致内容加载失败。

## 新增模块

1. 复制 `learn_claude_code.html` 为新页面
2. 修改页面标题、模块编号、`fetch` 的 `.md` 文件名
3. 创建对应的 `.md` 内容文件
4. 在 `index.html` 中将对应卡片的 `<div class="card">` 改为 `<a class="card card--link" href="...">`
