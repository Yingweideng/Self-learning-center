 5.8 `locator.hover()` — # Playwright 命令手册

## 第 1 章：Playwright 安装与配置

### 1.1 安装 Playwright

**Node.js**
```bash
npm init playwright@latest
```

**Python**
```bash
pip install playwright
playwright install
```

### 1.2 安装浏览器

**Node.js**
```bash
npx playwright install
```

**Python**
```bash
playwright install
```

### 1.3 安装有头浏览器（用于调试）

**Node.js**
```bash
npx playwright install --with-deps
```

**Python**
```bash
playwright install --with-deps
```

### 1.4 配置 playwright.config.ts

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  timeout: 30 * 1000,
  expect: {
    timeout: 5000
  },
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
  ],
});
```

## 第 2 章：基础概念

### 2.1 Browser, Context, Page

- **Browser**: 浏览器实例，对应一个浏览器进程
- **BrowserContext**: 浏览器上下文，相当于一个无痕浏览会话
- **Page**: 页面，一个标签页

### 2.2 创建浏览器实例

**Node.js**
```javascript
const { chromium } = require('playwright');

(async () => {
  const browser = await chromium.launch();
  const context = await browser.newContext();
  const page = await context.newPage();
  
  // ... 执行操作
  
  await browser.close();
})();
```

**Python**
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    context = browser.new_context()
    page = context.new_page()
    
    # ... 执行操作
    
    browser.close()
```

### 2.3 Locator 定位器

Locator 是 Playwright 的核心定位方式，支持自动等待和重试。

**常用定位方法：**
- `page.locator(selector)` - CSS 选择器
- `page.getByText(text)` - 按文本内容
- `page.getByRole(role)` - 按 ARIA 角色
- `page.getByLabel(label)` - 按 label 文本
- `page.getByPlaceholder(placeholder)` - 按占位符
- `page.getByAltText(alt)` - 按图片 alt 文本
- `page.getByTitle(title)` - 按 title 文本

## 第 3 章：页面导航

### 3.1 打开页面

**Node.js**
```javascript
await page.goto('https://example.com');
```

**Python**
```python
page.goto('https://example.com')
```

### 3.2 等待页面加载完成

**Node.js**
```javascript
await page.goto('https://example.com', { waitUntil: 'networkidle' });
```

**Python**
```python
page.goto('https://example.com', wait_until='networkidle')
```

**waitUntil 选项：**
- `load` - 等待 load 事件
- `domcontentloaded` - 等待 DOMContentLoaded 事件
- `networkidle` - 等待网络空闲
- `commit` - 等待网络响应接收

### 3.3 刷新页面

**Node.js**
```javascript
await page.reload();
```

**Python**
```python
page.reload()
```

### 3.4 前进/后退

**Node.js**
```javascript
await page.goBack();
await page.goForward();
```

**Python**
```python
page.go_back()
page.go_forward()
```

### 3.5 获取当前 URL

**Node.js**
```javascript
const url = page.url();
```

**Python**
```python
url = page.url
```

## 第 4 章：页面内容

### 4.1 获取页面标题

**Node.js**
```javascript
const title = await page.title();
```

**Python**
```python
title = page.title
```

### 4.2 获取页面 HTML

**Node.js**
```javascript
const html = await page.content();
```

**Python**
```python
html = page.content()
```

### 4.3 截图

**Node.js**
```javascript
await page.screenshot({ path: 'screenshot.png' });
```

**Python**
```python
page.screenshot(path='screenshot.png')
```

### 4.4 保存 PDF

**Node.js**
```javascript
await page.pdf({ path: 'page.pdf', format: 'A4' });
```

**Python**
```python
page.pdf(path='page.pdf', format='A4')
```

### 4.5 执行 JavaScript

**Node.js**
```javascript
const result = await page.evaluate(() => {
  return document.title;
});
```

**Python**
```python
result = page.evaluate("() => document.title")
```

### 4.6 等待元素出现

**Node.js**
```javascript
await page.waitForSelector('.my-element');
```

**Python**
```python
page.wait_for_selector('.my-element')
```

### 4.7 等待函数返回真值

**Node.js**
```javascript
await page.waitForFunction(() => window.innerWidth > 100);
```

**Python**
```python
page.wait_for_function("() => window.innerWidth > 100")
```

### 4.8 等待网络请求

**Node.js**
```javascript
const [response] = await Promise.all([
  page.waitForResponse('**/api/*'),
  page.click('#submit')
]);
```

**Python**
```python
with page.expect_response('**/api/*') as response_info:
    page.click('#submit')
response = response_info.value
```

## 第 5 章：页面动作

### 5.1 点击元素

**Node.js**
```javascript
await page.click('button');
```

**Python**
```python
page.click('button')
```

### 5.2 双击元素

**Node.js**
```javascript
await page.dblclick('button');
```

**Python**
```python
page.dblclick('button')
```

### 5.3 输入文本

**Node.js**
```javascript
await page.fill('input', 'Hello World');
```

**Python**
```python
page.fill('input', 'Hello World')
```

### 5.4 追加文本

**Node.js**
```javascript
await page.type('input', 'Hello World', { delay: 100 });
```

**Python**
```python
page.type('input', 'Hello World', delay=100)
```

### 5.5 选择下拉选项

**Node.js**
```javascript
await page.selectOption('select', 'option-value');
```

**Python**
```python
page.select_option('select', 'option-value')
```

### 5.6 勾选复选框

**Node.js**
```javascript
await page.check('input[type="checkbox"]');
```

**Python**
```python
page.check('input[type="checkbox"]')
```

### 5.7 取消勾选

**Node.js**
```javascript
await page.uncheck('input[type="checkbox"]');
```

**Python**
```python
page.uncheck('input[type="checkbox"]')
```

















































































































































































































































































































**Node.js**
```javascript
await page.locator('.menu').hover();
```

**Python**
```python
page.locator(".menu").hover()
```

### 5.9 `locator.setInputFiles(path)` — 上传文件

**Node.js**
```javascript
await page.locator('input[type="file"]').setInputFiles('./avatar.png');
```

**Python**
```python
page.locator('input[type="file"]').set_input_files("./avatar.png")
```

### 5.10 `locator.dragTo(target)` — 拖拽到目标元素

**Node.js**
```javascript
const source = page.locator('#item-1');
const target = page.locator('#dropzone');
await source.dragTo(target);
```

**Python**
```python
source = page.locator("#item-1")
target = page.locator("#dropzone")
source.drag_to(target)
```

### 5.11 `locator.scrollIntoViewIfNeeded()` — 滚动到可见区域

**Node.js**
```javascript
await page.locator('#footer').scrollIntoViewIfNeeded();
```

**Python**
```python
page.locator("#footer").scroll_into_view_if_needed()
```

### 5.12 `locator.focus()` / `blur()` — 聚焦 / 失焦

**Node.js**
```javascript
await page.locator('#email').focus();
await page.locator('#email').blur();
```

**Python**
```python
page.locator("#email").focus()
page.locator("#email").blur()
```

---

## 六、断言 (Assertions)

Playwright 自带的 **Web-First Assertions** 会自动重试，直到条件满足或超时。Node.js 中通过 `expect`，Python 中通过 `playwright.sync_api.expect`。

### 6.1 `toHaveTitle()` — 断言页面标题

**Node.js**
```javascript
const { test, expect } = require('@playwright/test');

test('title check', async ({ page }) => {
  await page.goto('https://playwright.dev');
  await expect(page).toHaveTitle(/Playwright/);
});
```

**Python**
```python
from playwright.sync_api import expect

page.goto("https://playwright.dev")
expect(page).to_have_title("Fast and reliable end-to-end testing for modern web apps | Playwright")
```

### 6.2 `toHaveURL()` — 断言当前 URL

**Node.js**
```javascript
await expect(page).toHaveURL(/.*dashboard/);
```

**Python**
```python
expect(page).to_have_url(re.compile(r".*dashboard"))
```

### 6.3 `toBeVisible()` / `toBeHidden()` — 元素可见性

**Node.js**
```javascript
await expect(page.locator('#welcome')).toBeVisible();
await expect(page.locator('#loading')).toBeHidden();
```

**Python**
```python
expect(page.locator("#welcome")).to_be_visible()
expect(page.locator("#loading")).to_be_hidden()
```

### 6.4 `toHaveText()` / `toContainText()` — 文本断言

**Node.js**
```javascript
await expect(page.locator('h1')).toHaveText('Welcome');
await expect(page.locator('.notice')).toContainText('successfully');
```

**Python**
```python
expect(page.locator("h1")).to_have_text("Welcome")
expect(page.locator(".notice")).to_contain_text("successfully")
```

### 6.5 `toHaveValue()` — 输入框值断言

**Node.js**
```javascript
await expect(page.locator('#email')).toHaveValue('test@example.com');
```

**Python**
```python
expect(page.locator("#email")).to_have_value("test@example.com")
```

### 6.6 `toHaveAttribute()` — 属性断言

**Node.js**
```javascript
await expect(page.locator('a.docs')).toHaveAttribute('href', /docs/);
```

**Python**
```python
expect(page.locator("a.docs")).to_have_attribute("href", re.compile(r"docs"))
```

### 6.7 `toHaveClass()` — class 断言

**Node.js**
```javascript
await expect(page.locator('#submit')).toHaveClass(/primary/);
```

**Python**
```python
expect(page.locator("#submit")).to_have_class(re.compile(r"primary"))
```

### 6.8 `toHaveCount()` — 元素数量断言

**Node.js**
```javascript
await expect(page.locator('li.todo')).toHaveCount(3);
```

**Python**
```python
expect(page.locator("li.todo")).to_have_count(3)
```

### 6.9 `toBeEnabled()` / `toBeDisabled()` / `toBeChecked()` — 状态断言

**Node.js**
```javascript
await expect(page.locator('#submit')).toBeEnabled();
await expect(page.locator('#cancel')).toBeDisabled();
await expect(page.locator('#agree')).toBeChecked();
```

**Python**
```python
expect(page.locator("#submit")).to_be_enabled()
expect(page.locator("#cancel")).to_be_disabled()
expect(page.locator("#agree")).to_be_checked()
```

### 6.10 `toBeFocused()` / `toBeEmpty()` / `toBeEditable()` — 其他常用断言

**Node.js**
```javascript
await expect(page.locator('#email')).toBeFocused();
await expect(page.locator('#notes')).toBeEmpty();
await expect(page.locator('#title')).toBeEditable();
```

**Python**
```python
expect(page.locator("#email")).to_be_focused()
expect(page.locator("#notes")).to_be_empty()
expect(page.locator("#title")).to_be_editable()
```

---

## 附：推荐实践

在实际项目中，建议优先使用 **Locator + Web-First Assertion** 组合，避免使用 `waitForTimeout` 这类硬等待。Locator 内置自动等待与重试机制，可显著提升测试稳定性。对于复杂的端到端测试，结合 `page.route()` 进行接口 Mock，结合 `page.addInitScript()` 注入测试态，可以让测试既快又稳。



































































































































































