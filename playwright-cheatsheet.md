 5.8 `locator.hover()` — 鼠标悬停

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



































































































































































