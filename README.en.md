# ruyipage-js

`ruyipage-js` is a Firefox automation framework for Node.js based on WebDriver BiDi.

Chinese documentation: `README.md`

---

## Repository Relationship

- JS implementation repository: [`GanFish404/ruyipage-js`]（https://github.com/GanFish404/ruyipage-js）
- Python baseline repository: [`LoseNine/ruyipage`](https://github.com/LoseNine/ruyipage)
- Go implementation repository: [`pll177/ruyipage-go`](https://github.com/pll177/ruyipage-go)

`ruyipage-js` follows the same Firefox + BiDi direction as Python `ruyipage`; the Go version `ruyipage-go` is a parallel implementation on the same track. They target similar high-level automation capabilities in different language ecosystems.

---

## Contents

- [1. Install](#1-install)
- [2. Quick Start](#2-quick-start)
- [3. Core Objects](#3-core-objects)
- [4. Page APIs](#4-page-apis)
- [5. Element APIs](#5-element-apis)
- [6. Prompt & Dialogs](#6-prompt--dialogs)
- [7. Common Scenarios](#7-common-scenarios)
- [8. Example Test Case](#8-example-test-case)
- [9. Notes](#9-notes)

---

## 1. Install

```bash
npm install
```

- Required Node.js version: `>=16`
- Browser: Firefox (latest stable recommended)

---

## 2. Quick Start

Run built-in example:

```bash
node quickstart_bing_search.js
```

Minimal runnable example:

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

## 3. Core Objects

- `FirefoxOptions`: browser launch options
- `FirefoxPage`: main page object (primary API entry)
- `FirefoxTab`: tab object
- `FirefoxFrame`: iframe object
- `FirefoxElement`: element object

Common import:

```js
const { FirefoxOptions, FirefoxPage, Keys, By } = require('./index');
```

---

## 4. Page APIs

### 4.1 Navigation

- `await page.get(url)`
- `await page.back()`
- `await page.forward()`
- `await page.refresh()`

### 4.2 Element Query

- `await page.ele(locator, index?, timeout?)`
- `await page.eles(locator, timeout?)`

Locator examples:
- CSS: `'css:#login-btn'` or `'#login-btn'`
- XPath: `'xpath://button[@id="login-btn"]'`

### 4.3 JavaScript Execution

- `await page.run_js(script, ...args)`
- `await page.run_js('document.title', { as_expr: true })`

### 4.4 Page Info

- `await page.title`
- `await page.url`
- `await page.html`

### 4.5 Page Shutdown

- `await page.close()`
- `await page.quit()`

---

## 5. Element APIs

Common element methods:

- `await ele.click()`
- `await ele.input(text)`
- `await ele.text`
- `await ele.attr(name)`
- `await ele.run_js(script, ...args)`

Example:

```js
const searchInput = await page.ele('#sb_form_q');
await searchInput.input('ruyipage');
```

---

## 6. Prompt & Dialogs

### 6.1 Auto Handling Strategy

```js
await page.set_prompt_handler({
  alert: 'dismiss',
  confirm: 'accept',
  prompt: 'ignore',
  default: 'dismiss',
  // prompt_text: 'optional text'
});
```

Supported strategies:
- `'accept'`
- `'dismiss'`
- `'ignore'`

### 6.2 Manual Handling

- `await page.wait_prompt(timeout?)`
- `await page.accept_prompt(text?)`
- `await page.dismiss_prompt()`
- `await page.handle_prompt(accept?, text?)`

### 6.3 Prompt Login Flow

- `await page.prompt_login(locator, username, password, trigger?, timeout?)`

---

## 7. Common Scenarios

### 7.1 Open page and click element

```js
await page.get('https://example.com');
const btn = await page.ele('#submit');
await btn.click();
```

### 7.2 Read text from list

```js
const items = await page.eles('.item');
for (const it of items) {
  console.log(await it.text);
}
```

### 7.3 Run page script

```js
const title = await page.run_js('return document.title;');
console.log(title);
```

---

## 8. Example Test Case

This example shows `set_prompt_handler` and prompt state checks.

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

## 9. Notes

- Most browser-interaction APIs require `await`
- Always call `await page.quit()` in `finally`
- For dynamic pages, add timeout/retry strategy for element lookup
- Avoid mixing auto prompt handling and manual handling at the same moment
