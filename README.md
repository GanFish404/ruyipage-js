# ruyipage-js

`ruyipage-js` 是基于 WebDriver BiDi 的 Firefox 自动化框架（Node.js 版）。

英文文档请查看：`README.en.md`

---

## 仓库关系

- JS 实现仓库：[`GanFish404/ruyipage-js`]（https://github.com/GanFish404/ruyipage-js）
- Go 实现仓库：[`pll177/ruyipage-go`](https://github.com/pll177/ruyipage-go)
- Python 基线仓库：[`LoseNine/ruyipage`](https://github.com/LoseNine/ruyipage)

`ruyipage-js` 延续 Python `ruyipage` 的 Firefox + BiDi 技术路线；Go 版 `ruyipage-go` 也是同路线的并行实现。三者都围绕同一类高层自动化能力建设，但分别面向不同语言生态。

---

## 项目定位

`ruyipage-js` 不是“只包一层底层 BiDi 命令”的轻封装，而是面向业务自动化的高层 API：

- 页面自动化（导航、定位、交互、脚本执行）
- Prompt/对话框处理（含自动策略和手动处理）
- 网络相关能力（监听、拦截、上下文隔离）
- 页面上下文能力（tab、frame、storage、wait、actions）

如果你希望写的是“可维护的业务流程脚本”，而不仅是临时脚本，这套 API 会更顺手。

---

## 能力总览

| 能力域 | 主要入口 | 典型用途 |
| --- | --- | --- |
| 浏览器启动 | `FirefoxPage.create()` | 创建会话并进入页面 |
| 页面导航 | `get/back/forward/refresh` | 页面流转与跳转控制 |
| 元素定位与操作 | `ele/eles` + `click/input` | 表单填写、按钮点击、信息提取 |
| 脚本执行 | `run_js` | 复杂 DOM 读取与页面内逻辑执行 |
| Prompt 管理 | `set_prompt_handler` / `wait_prompt` / `handle_prompt` | 统一处理 alert/confirm/prompt |
| 业务等待 | `page.wait` / `ele.wait` | 降低动态页面时序问题 |
| 存储操作 | `local_storage` / `session_storage` | 登录态、页面缓存、状态读写 |
| 多上下文 | `new_tab` / `get_frame` | 多 tab、iframe 自动化 |

---

## 目录

- [1. 安装与环境](#1-安装与环境)
- [2. 快速开始](#2-快速开始)
- [3. 核心对象](#3-核心对象)
- [4. 页面能力](#4-页面能力)
- [5. 元素能力](#5-元素能力)
- [6. Prompt 与对话框](#6-prompt-与对话框)
- [7. 常见场景示例](#7-常见场景示例)
- [8. 示例测试用例](#8-示例测试用例)
- [9. 使用注意事项](#9-使用注意事项)
- [10. 常见问题（FAQ）](#10-常见问题faq)

---

## 1. 安装与环境

```bash
npm install
```

- Node.js 版本要求：`>=16`
- 浏览器：Firefox（建议使用稳定版最新版本）

---

## 2. 快速开始

运行内置示例：

```bash
node quickstart_bing_search.js
```

最小可运行示例：

```js
const { FirefoxOptions, FirefoxPage } = require('./index');

async function main() {
  const opts = new FirefoxOptions();
  const page = await FirefoxPage.create(opts);
  try {
    await page.get('https://example.com');
    console.log('title =', await page.title);
    console.log('url =', await page.url);
  } finally {
    await page.quit();
  }
}

main().catch(console.error);
```

---

## 3. 核心对象

- `FirefoxOptions`：浏览器启动配置
- `FirefoxPage`：页面主对象（大多数 API 入口）
- `FirefoxTab`：标签页对象
- `FirefoxFrame`：iframe 对象
- `FirefoxElement`：元素对象

常用导入：

```js
const { FirefoxOptions, FirefoxPage, Keys, By } = require('./index');
```

---

## 4. 页面能力

### 4.1 导航

- `await page.get(url)`
- `await page.back()`
- `await page.forward()`
- `await page.refresh()`

### 4.2 元素查找

- `await page.ele(locator, index?, timeout?)`
- `await page.eles(locator, timeout?)`

定位写法示例：
- CSS: `'css:#login-btn'` or `'#login-btn'`
- XPath: `'xpath://button[@id="login-btn"]'`

### 4.3 脚本执行

- `await page.run_js(script, ...args)`
- `await page.run_js('document.title', { as_expr: true })`

### 4.4 页面信息

- `await page.title`
- `await page.url`
- `await page.html`

### 4.5 页面结束

- `await page.close()`
- `await page.quit()`

---

## 5. 元素能力

拿到元素后常用方法：

- `await ele.click()`
- `await ele.input(text)`
- `await ele.text`
- `await ele.attr(name)`
- `await ele.run_js(script, ...args)`

示例：

```js
const searchInput = await page.ele('#sb_form_q');
await searchInput.input('ruyipage');
```

---

## 6. Prompt 与对话框

### 6.1 设置自动处理策略

```js
await page.set_prompt_handler({
  alert: 'dismiss',
  confirm: 'accept',
  prompt: 'ignore',
  default: 'dismiss',
  // prompt_text: 'optional text'
});
```

支持的策略：
- `'accept'`
- `'dismiss'`
- `'ignore'`

### 6.2 手动处理

- `await page.wait_prompt(timeout?)`
- `await page.accept_prompt(text?)`
- `await page.dismiss_prompt()`
- `await page.handle_prompt(accept?, text?)`

### 6.3 登录弹窗场景

- `await page.prompt_login(locator, username, password, trigger?, timeout?)`

---

## 7. 常见场景示例

### 7.1 打开页面并点击元素

```js
await page.get('https://example.com');
const btn = await page.ele('#submit');
await btn.click();
```

### 7.2 获取列表文本

```js
const items = await page.eles('.item');
for (const it of items) {
  console.log(await it.text);
}
```

### 7.3 执行页面脚本

```js
const title = await page.run_js('return document.title;');
console.log(title);
```

---

## 8. 示例测试用例

下面示例演示 `set_prompt_handler` + prompt 状态检查。

```js
const { FirefoxOptions, FirefoxPage } = require('./index');

async function casePromptHandler() {
  const page = await FirefoxPage.create(new FirefoxOptions());
  try {
    await page.set_prompt_handler({
      alert: 'dismiss',
      confirm: 'accept',
      prompt: 'ignore',
      default: 'dismiss',
    });

    await page.get('https://example.com');

    const opened = page.get_last_prompt_opened();
    const closed = page.get_last_prompt_closed();
    console.log({ opened, closed });
  } finally {
    await page.quit();
  }
}

casePromptHandler();
```

---

## 9. 使用注意事项

- 大多数浏览器交互 API 需要 `await`
- 建议每个脚本都在 `finally` 里调用 `await page.quit()`
- 对于动态页面，元素查找建议加超时与重试策略
- Prompt 自动处理与手动处理不要混用在同一时刻，避免语义冲突

---

## 10. 常见问题（FAQ）

### Q1：为什么有些属性也需要 `await`？
`ruyipage-js` 与浏览器通信是异步模型，像 `page.title`、`page.url` 这类值通过 BiDi 获取，因此使用时建议统一写成 `await page.title`。

### Q2：`set_prompt_handler` 和 `handle_prompt` 怎么选？
如果你的流程里 prompt 逻辑固定，优先用 `set_prompt_handler` 做统一策略；如果每次弹窗处理不同，使用 `wait_prompt` + `handle_prompt` 更灵活。

### Q3：元素偶尔找不到怎么办？
先确认定位器稳定，再增加等待（如 `page.wait` / `ele.wait`）；动态页面建议避免立即查找，先等待关键区域渲染。

### Q4：脚本结束后浏览器没关？
确保把 `await page.quit()` 放在 `finally`，不要只放在正常分支里。
