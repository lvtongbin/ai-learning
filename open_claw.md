# Computer Use

简单说：Computer Use = 让 AI 像人一样直接操作电脑。

---

## Computer Use 是什么

### 从"对话"到"操作"

传统 AI 只能回答问题、生成文本。Computer Use 让 AI 拥有了"手"——它可以：

- 看到屏幕上的内容（截图或 DOM 结构）
- 判断下一步该做什么
- 执行鼠标点击、键盘输入等操作
- 观察结果，继续下一步

这个循环让 AI 能够完成任何人类在电脑上能做的事。

### 行业现状

2024-2025 年，各大 AI 公司相继推出 Computer Use 能力：

| 产品 | 厂商 | 特点 |
|------|------|------|
| Computer Use | Anthropic (Claude) | 最早公开发布，基于截图感知 |
| CUA (Computer-Using Agent) | OpenAI | 基于 GUI 交互，支持 Responses API |
| ChatGPT Agent | OpenAI | 集成研究与操作能力的完整 Agent |

> OpenAI CUA 被训练为直接与图形界面交互——按钮、菜单、文本框——就像人类使用电脑一样。它分析屏幕内容，返回"点击坐标 (450, 320)"或"在当前输入框输入文字"这样的操作指令。

---

## 技术栈：浏览器自动化的底层

理解 Computer Use，需要先了解它依赖的三层技术。

### 第一层：Chrome DevTools Protocol (CDP)

CDP 是一切浏览器自动化的基础协议。

当 Chrome 以 `--remote-debugging-port=9222` 启动时，它会暴露一个 WebSocket 端点，接受符合 CDP 规范的 JSON 命令。通过这个协议，外部程序可以：

- 导航到任意 URL
- 截取页面截图
- 读取和修改 DOM
- 监听网络请求
- 执行 JavaScript
- 模拟鼠标和键盘事件

```
AI Agent
   ↓ 发送 JSON 命令
WebSocket (ws://localhost:9222)
   ↓
Chrome 浏览器内核
   ↓ 返回结果（截图、DOM、网络数据）
AI Agent
```

CDP 是 Puppeteer、Playwright 等所有主流自动化工具的底层基础。

> 参考：[Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/)

### 第二层：Playwright

Playwright 是微软开源的浏览器自动化框架，在 CDP 之上提供了更高层的 API。

**核心优势**：

- **跨浏览器**：单一 API 驱动 Chromium、Firefox、WebKit
- **自动等待**：无需手动设置超时，框架自动等待元素可操作
- **测试隔离**：每个测试在独立的浏览器上下文中运行
- **AI 集成**：提供 MCP 服务，让 AI 模型直接控制浏览器

```python
from playwright.async_api import async_playwright

async with async_playwright() as p:
    browser = await p.chromium.launch()
    page = await browser.new_page()
    await page.goto("https://example.com")
    await page.click("button#submit")
    screenshot = await page.screenshot()
```

Playwright 既是测试工具，也是 AI Agent 的执行引擎。

> 参考：[microsoft/playwright](https://github.com/microsoft/playwright)

### 第三层：browser-use

browser-use 是专为 AI Agent 设计的 Python 库，把 LLM 和浏览器控制连接在一起。

**它解决的问题**：AI 模型不能直接操作浏览器，需要一个"翻译层"——把自然语言任务转化为具体的浏览器操作，再把操作结果反馈给模型。

**工作方式**：

```
用户任务（自然语言）
      ↓
browser-use 提取页面 DOM 结构
      ↓
将结构化页面信息发送给 LLM
      ↓
LLM 决定下一步操作
      ↓
browser-use 通过 Playwright 执行操作
      ↓
观察结果，循环直到任务完成
```

与截图方案不同，browser-use 优先使用 DOM 结构（而非图像）作为模型输入，这让模型能更精确地理解页面元素，减少幻觉。

```python
from browser_use import Agent
from langchain_anthropic import ChatAnthropic

agent = Agent(
    task="在 GitHub 上搜索 browser-use 并打开第一个结果",
    llm=ChatAnthropic(model="claude-sonnet-4-6"),
)
await agent.run()
```

> 参考：[browser-use/browser-use](https://github.com/browser-use/browser-use)

---

## 技术选型对比

| 方案 | 感知方式 | 适用场景 | 优缺点 |
|------|----------|----------|--------|
| 截图 + Vision | 图像 | 任意界面（含桌面应用） | 通用但 token 消耗大 |
| DOM 提取 | 结构化文本 | Web 页面 | 精准、省 token，不支持非 Web |
| CDP 直连 | 底层协议 | 需要精细控制 | 灵活但开发成本高 |
| Playwright | 高层 API | 测试 + Agent | 易用、跨浏览器 |
| browser-use | LLM 集成 | AI Agent 开发 | 开箱即用，专为 AI 设计 |

---

## 快速上手

### 环境准备

```bash
# 安装 browser-use
pip install browser-use

# 安装 Playwright 浏览器
playwright install chromium
```

### 最简示例

```python
import asyncio
from browser_use import Agent
from langchain_anthropic import ChatAnthropic

async def main():
    agent = Agent(
        task="打开 Hacker News，告诉我今天排名第一的文章标题",
        llm=ChatAnthropic(model="claude-sonnet-4-6"),
    )
    result = await agent.run()
    print(result)

asyncio.run(main())
```

### 与 Claude Code 结合

Computer Use 和 Claude Code 是互补的：

- **Claude Code**：操作本地文件系统、执行代码、管理项目
- **Computer Use**：操作浏览器、与 Web 界面交互、完成在线任务

两者结合，AI 就能完成从"写代码"到"部署上线并验证结果"的完整闭环。

---

## 延伸阅读

- [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/)
- [microsoft/playwright](https://github.com/microsoft/playwright)
- [browser-use/browser-use](https://github.com/browser-use/browser-use)
- [OpenAI Computer-Using Agent](https://openai.com/index/computer-using-agent)
