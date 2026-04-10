# Module 01 Detail Page Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create `learn_claude_code.html` that dynamically loads and renders `learn_claude_code.md` in the same dark sci-fi style as `index.html`, and wire up the Module 01 card to link to it.

**Architecture:** A standalone HTML page fetches `learn_claude_code.md` via `fetch()`, renders it with `marked.js` (CDN), and applies custom CSS that mirrors the `index.html` design system (same CSS variables, fonts, background grid). The Module 01 card in `index.html` becomes a clickable link.

**Tech Stack:** Vanilla HTML/CSS/JS, marked.js (CDN), Google Fonts (Orbitron + Noto Sans SC)

---

### Task 1: Create `learn_claude_code.html`

**Files:**
- Create: `learn_claude_code.html`

- [ ] **Step 1: Create the HTML file**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Learn Claude Code · AI 编程培训</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&family=Noto+Sans+SC:wght@400;500;700&display=swap" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    :root {
      --bg: #0a0a0f;
      --surface: #12121a;
      --border: rgba(0, 212, 255, 0.15);
      --cyan: #00d4ff;
      --purple: #a855f7;
      --text: #e2e8f0;
      --muted: #64748b;
    }

    body {
      background-color: var(--bg);
      color: var(--text);
      font-family: 'Noto Sans SC', sans-serif;
      min-height: 100vh;
      overflow-x: hidden;
    }

    body::before {
      content: '';
      position: fixed;
      inset: 0;
      background-image:
        linear-gradient(rgba(0, 212, 255, 0.03) 1px, transparent 1px),
        linear-gradient(90deg, rgba(0, 212, 255, 0.03) 1px, transparent 1px);
      background-size: 40px 40px;
      pointer-events: none;
      z-index: 0;
    }

    /* ── Nav ── */
    nav {
      position: relative;
      z-index: 1;
      padding: 24px 40px;
      border-bottom: 1px solid var(--border);
    }

    .back-link {
      display: inline-flex;
      align-items: center;
      gap: 8px;
      font-family: 'Orbitron', monospace;
      font-size: 11px;
      letter-spacing: 2px;
      color: var(--cyan);
      text-decoration: none;
      text-transform: uppercase;
      opacity: 0.7;
      transition: opacity 0.2s;
    }

    .back-link:hover { opacity: 1; }

    .back-link::before {
      content: '←';
      font-family: sans-serif;
      font-size: 14px;
    }

    /* ── Content ── */
    main {
      position: relative;
      z-index: 1;
      max-width: 800px;
      margin: 0 auto;
      padding: 60px 40px 100px;
    }

    .module-label {
      font-family: 'Orbitron', monospace;
      font-size: 11px;
      letter-spacing: 3px;
      color: var(--cyan);
      opacity: 0.6;
      margin-bottom: 48px;
    }

    /* ── Markdown styles ── */
    #content h1 {
      font-family: 'Orbitron', monospace;
      font-size: clamp(22px, 4vw, 36px);
      font-weight: 700;
      background: linear-gradient(135deg, var(--cyan) 0%, var(--purple) 100%);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      background-clip: text;
      line-height: 1.3;
      margin-bottom: 32px;
    }

    #content h2 {
      font-size: 20px;
      font-weight: 700;
      color: var(--cyan);
      margin: 48px 0 16px;
      padding-bottom: 8px;
      border-bottom: 1px solid var(--border);
    }

    #content h3 {
      font-size: 16px;
      font-weight: 700;
      color: #f1f5f9;
      margin: 32px 0 12px;
    }

    #content p {
      font-size: 15px;
      line-height: 1.8;
      color: var(--text);
      margin-bottom: 16px;
    }

    #content ul, #content ol {
      padding-left: 24px;
      margin-bottom: 16px;
    }

    #content li {
      font-size: 15px;
      line-height: 1.8;
      color: var(--text);
      margin-bottom: 6px;
    }

    #content a {
      color: var(--cyan);
      text-decoration: none;
      border-bottom: 1px solid rgba(0, 212, 255, 0.3);
      transition: border-color 0.2s;
    }

    #content a:hover { border-color: var(--cyan); }

    #content code {
      font-family: 'Courier New', monospace;
      font-size: 13px;
      background: rgba(0, 212, 255, 0.08);
      color: var(--cyan);
      padding: 2px 6px;
      border-radius: 3px;
      border: 1px solid rgba(0, 212, 255, 0.15);
    }

    #content pre {
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: 8px;
      padding: 20px 24px;
      overflow-x: auto;
      margin-bottom: 20px;
    }

    #content pre code {
      background: none;
      border: none;
      padding: 0;
      color: #a5f3fc;
      font-size: 13px;
      line-height: 1.7;
    }

    #content blockquote {
      border-left: 3px solid var(--purple);
      padding: 12px 20px;
      margin: 20px 0;
      background: rgba(168, 85, 247, 0.05);
      border-radius: 0 6px 6px 0;
    }

    #content blockquote p {
      color: var(--muted);
      margin-bottom: 0;
    }

    #content hr {
      border: none;
      border-top: 1px solid var(--border);
      margin: 40px 0;
    }

    #content table {
      width: 100%;
      border-collapse: collapse;
      margin-bottom: 20px;
      font-size: 14px;
    }

    #content th {
      font-family: 'Orbitron', monospace;
      font-size: 11px;
      letter-spacing: 1px;
      color: var(--cyan);
      text-align: left;
      padding: 10px 14px;
      border-bottom: 1px solid var(--border);
    }

    #content td {
      padding: 10px 14px;
      border-bottom: 1px solid rgba(255,255,255,0.04);
      color: var(--text);
    }

    #content tr:hover td { background: rgba(0, 212, 255, 0.03); }

    /* ── Loading / Error ── */
    .state-msg {
      font-family: 'Orbitron', monospace;
      font-size: 12px;
      letter-spacing: 2px;
      color: var(--muted);
      text-align: center;
      padding: 80px 0;
    }

    /* ── Footer ── */
    footer {
      position: relative;
      z-index: 1;
      text-align: center;
      padding: 32px 24px;
      border-top: 1px solid var(--border);
      color: var(--muted);
      font-size: 13px;
    }
  </style>
</head>
<body>

  <nav>
    <a class="back-link" href="index.html">返回课程列表</a>
  </nav>

  <main>
    <div class="module-label">MODULE · 01</div>
    <div id="content"><p class="state-msg">LOADING...</p></div>
  </main>

  <footer>&copy; 2026 AI 编程培训 · 内容持续更新中</footer>

  <script>
    fetch('learn_claude_code.md')
      .then(r => {
        if (!r.ok) throw new Error(r.status);
        return r.text();
      })
      .then(text => {
        document.getElementById('content').innerHTML = marked.parse(text);
      })
      .catch(() => {
        document.getElementById('content').innerHTML =
          '<p class="state-msg">内容加载失败，请检查文件是否存在。</p>';
      });
  </script>

</body>
</html>
```

- [ ] **Step 2: Verify the file was created**

```bash
ls -la "learn_claude_code.html"
```
Expected: file exists with non-zero size.

- [ ] **Step 3: Commit**

```bash
git add learn_claude_code.html
git commit -m "feat: add module 01 detail page with markdown renderer"
```

---

### Task 2: Wire up Module 01 card in `index.html`

**Files:**
- Modify: `index.html` (Module 01 card block, ~lines 247-257)

- [ ] **Step 1: Add `.card--link` style to `index.html` `<style>` block**

Find the `.card:hover` rule and add after it:

```css
    .card--link {
      cursor: pointer;
      text-decoration: none;
      display: block;
      color: inherit;
    }
```

- [ ] **Step 2: Replace the Module 01 `<div class="card">` with an `<a>` tag and fill in content**

Replace:
```html
      <!-- 模块 01 -->
      <div class="card">
        <div class="card-header">
          <span class="module-num">MODULE · 01</span>
          <span class="status available">已开放</span>
        </div>
        <div class="card-title"><!-- TODO: 模块标题 --></div>
        <div class="card-desc"><!-- TODO: 模块描述 --></div>
        <div class="card-tags">
          <span class="tag"><!-- tag --></span>
        </div>
      </div>
```

With:
```html
      <!-- 模块 01 -->
      <a class="card card--link" href="learn_claude_code.html">
        <div class="card-header">
          <span class="module-num">MODULE · 01</span>
          <span class="status available">已开放</span>
        </div>
        <div class="card-title">Learn Claude Code</div>
        <div class="card-desc">系统学习 Claude Code CLI 工具的核心用法，掌握 AI 辅助编程的基础工作流。</div>
        <div class="card-tags">
          <span class="tag">Claude Code</span>
          <span class="tag">CLI</span>
          <span class="tag">AI 编程</span>
        </div>
      </a>
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: link module 01 card to learn_claude_code detail page"
```
