# Playwright Automation & Scraping Mastery (S4 Edition)

## Skill Identity
**Name:** Playwright Automation & Scraping (S4)
**Version:** 1.0.0
**Description:** High-leverage browser automation and web scraping. Focuses on robust selectors, anti-bot bypass, and scalable data extraction.
**Trigger Requirements:**
- "cào dữ liệu" (scrape data)
- "automation playwright"
- "tự động hóa trình duyệt"
- "browser scraping"
- "bỏ qua bot detection"

## 💡 The S4 Philosophy (Automation Edition)
> **"Browser is the Ultimate API."**

If a website doesn't have an API, the browser *is* your API. 
The goal is not just to "make it work," but to make it **robust, stealthy, and maintainable.**
- **Robust:** It shouldn't break when a CSS class changes.
- **Stealthy:** It shouldn't get blocked by Cloudflare or Akamai.
- **Maintainable:** It should be easy to debug using Traces and Logs.

---

## 1. The "Stability" Laws (No More Flaky Scripts)

### ✅ Law 1: Never Use `page.waitForTimeout()`
Hardcoded waits are the #1 cause of flaky scripts.
- **Bad:** `await page.waitForTimeout(3000);` (Too slow or too fast).
- **Good:** `await page.waitForSelector('.success-message');` or `await page.waitForResponse(url);`.

### ✅ Law 2: Use Locators, Not Selectors
Playwright Locators have built-in auto-waiting and retry logic.
- **Bad:** `await page.click('.submit-btn');`
- **Good:** `await page.getByRole('button', { name: 'Submit' }).click();`
- **Why:** Locators are more resilient to UI changes and handle "Actionability" checks automatically.

### ✅ Law 3: State-Driven Navigation
Don't just click and hope. Verify the state.
```javascript
await page.goto('/login');
await page.fill('#user', 'admin');
await page.click('#submit');
// Verify we actually logged in
await expect(page).toHaveURL(/dashboard/);
```

---

## 2. Anti-Bot & Stealth (The "Ninja" Mode)
*How to avoid the "Access Denied" screen.*

### 🛡️ 1. Human-Like Headers
Always set a realistic User-Agent and viewport.
```javascript
const context = await browser.newContext({
  userAgent: 'Mozilla/5.0 (Windows NT 10.0; Win64; x4) AppleWebKit/537.36...',
  viewport: { width: 1920, height: 1080 }
});
```

### 🛡️ 2. The `playwright-stealth` Plugin
Essential for bypassing common headless detection (navigator.webdriver, etc.).
- Use `playwright-extra` and `stealth`.

### 🛡️ 3. Request Interception (Speed + Stealth)
Block images, CSS, and fonts to save bandwidth and look less like a bot.
```javascript
await page.route('**/*.{png,jpg,jpeg,css,woff2}', route => route.abort());
```

---

## 3. High-Leverage Scraping Patterns

### 📊 Pattern 1: Infinite Scroll Scraping
Don't just scroll. Monitor the data flow.
```javascript
// Scroll until no new items appear
let previousCount = 0;
while (true) {
  await page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));
  await page.waitForTimeout(1000); // Small buffer for network
  const currentCount = await page.locator('.item').count();
  if (currentCount === previousCount) break;
  previousCount = currentCount;
}
```

### 📊 Pattern 2: API Interception (The "Cheat Code")
Instead of scraping HTML, steal the JSON the website uses.
```javascript
page.on('response', async response => {
  if (response.url().includes('/api/v1/data')) {
    const data = await response.json();
    console.log('Got the raw data:', data);
  }
});
```

---

## 4. The "Kill List" (Immediate Rejection)
*If your script does this, it will FAIL in production.*

- ❌ **Root Selectors:** `await page.click('div > div > span > button')`. (Breaks instantly on UI update).
- ❌ **No Error Handling:** Scripts that crash and leave browser instances hanging (memory leak).
- ❌ **Hardcoded Credentials:** Use `.env` files.
- ❌ **Headless Mode Always:** Some sites detect `headless: true`. Test with `headless: false` + `slowMo`.

---

## 🛠️ Tooling & Debugging
1.  **Trace Viewer:** `npx playwright show-trace` (See exactly what happened at every step).
2.  **Codegen:** `npx playwright codegen [url]` (Generate code by clicking).
3.  **UI Mode:** `npx playwright test --ui` (Interactive runner).

---

## Summary Checklist
Before deploying a scraper:
1.  [ ] Are you using **Locators** instead of raw CSS selectors?
2.  [ ] Is **Stealth Mode** enabled?
3.  [ ] Did you block unnecessary resources (images/CSS)?
4.  [ ] Is there a **Retry** mechanism for network failures?
5.  [ ] Is the data being saved incrementally (to avoid loss)?

**Next Step:** Proceed to **Proxy Rotation** setup if scraping >1000 pages.
