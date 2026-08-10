# HTML 尖括號轉換器 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 建立一個單檔、可直接開啟的繁體中文網站，按下按鈕後只轉換 `<` 與 `>`，並可複製結果。

**Architecture:** 所有標記、樣式與行為集中在根目錄的 `index.html`，以原生 JavaScript 操作 DOM。核心 `escapeAngleBrackets(value)` 函式保持純函式，介面事件只負責讀寫欄位、狀態與剪貼簿，方便獨立驗證。

**Tech Stack:** HTML5、CSS3、原生 JavaScript、瀏覽器 Clipboard API

## Global Constraints

- 僅建立一個可直接開啟的 `index.html`，不使用框架、套件、後端或網路請求。
- 只把 `<` 轉成 `&lt;`，把 `>` 轉成 `&gt;`；包括 `&` 在內的其他字元保持不變。
- 介面使用繁體中文、簡潔專業的藍灰色視覺風格。
- 桌面左右並排，窄螢幕上下排列；支援鍵盤焦點、`aria-live` 與 `prefers-reduced-motion`。
- 空白輸入須提示；無結果時停用複製；複製失敗須提供手動複製指引。

---

### Task 1: 建立並驗證完整單頁工具

**Files:**
- Create: `index.html`

**Interfaces:**
- Consumes: 使用者輸入的字串、`navigator.clipboard.writeText(text)`（可用時）
- Produces: `escapeAngleBrackets(value: string): string`、`copyResult(): Promise<void>`，以及可操作的表單介面

- [ ] **Step 1: 建立功能驗收測試清單並確認初始狀態不通過**

在建立 `index.html` 前執行：

```powershell
Test-Path .\index.html
```

Expected: `False`。

後續實作必須通過以下案例：

```text
escapeAngleBrackets('<div>Hello</div>')
=> '&lt;div&gt;Hello&lt;/div&gt;'

escapeAngleBrackets('A & B')
=> 'A & B'

escapeAngleBrackets('<<tag>>')
=> '&lt;&lt;tag&gt;&gt;'

escapeAngleBrackets('plain text')
=> 'plain text'
```

- [ ] **Step 2: 建立語意化 HTML 結構**

在 `index.html` 建立：

```html
<main class="shell">
  <header class="intro">...</header>
  <section class="workspace" aria-label="HTML 尖括號轉換工具">
    <div class="editor-panel">原始內容 textarea 與字元數</div>
    <div class="editor-panel">唯讀結果 textarea 與字元數</div>
  </section>
  <div class="actions">轉換內容、清除內容、複製結果</div>
  <p id="status" role="status" aria-live="polite"></p>
</main>
```

使用 `label` 與 `for` 連結文字區；結果欄加上 `readonly`；複製按鈕初始為 `disabled`。頁面標題與說明明確指出只轉換 `<`、`>`，內容不會上傳。

- [ ] **Step 3: 實作簡潔專業的響應式視覺**

在同一檔案的 `<style>` 定義設計 token：

```css
:root {
  --canvas: #f3f6f9;
  --surface: #ffffff;
  --ink: #15243a;
  --muted: #66758a;
  --line: #d9e1ea;
  --accent: #1f5fae;
  --accent-strong: #184c8b;
  --success: #18794e;
  --danger: #b42318;
}
```

使用系統字型、清晰層級、12px 圓角與克制陰影。`.workspace` 在桌面使用兩欄 grid，`@media (max-width: 760px)` 改為單欄；按鈕須具備 hover、disabled 及 `:focus-visible` 狀態。動畫只用於狀態與按鈕細微回饋，並在 `prefers-reduced-motion: reduce` 時停用。

- [ ] **Step 4: 實作純轉換函式與按鈕流程**

在 `<script>` 實作核心函式：

```js
function escapeAngleBrackets(value) {
  return value.replaceAll('<', '&lt;').replaceAll('>', '&gt;');
}
```

「轉換內容」事件先以 `input.value.trim()` 判斷空白；空白時不寫入結果並顯示錯誤訊息。有效內容則把完整原文（保留前後空白）傳入函式，更新結果、字元數、複製按鈕狀態及成功訊息。「清除內容」清空兩欄與訊息、停用複製並將焦點移回輸入欄。

- [ ] **Step 5: 實作剪貼簿與備援複製**

優先使用：

```js
await navigator.clipboard.writeText(output.value);
```

若 Clipboard API 不可用或拒絕，選取唯讀結果欄並呼叫 `document.execCommand('copy')` 作為備援。成功顯示「已複製到剪貼簿」並短暫把按鈕文字改為「已複製」；兩種方式都失敗時顯示「無法自動複製，請選取結果後手動複製」。

- [ ] **Step 6: 執行靜態與核心邏輯驗證**

執行：

```powershell
Test-Path .\index.html
Select-String -Path .\index.html -Pattern "replaceAll\('<', '&lt;'\).*replaceAll\('>', '&gt;'\)"
Select-String -Path .\index.html -Pattern 'navigator.clipboard.writeText'
Select-String -Path .\index.html -Pattern 'aria-live="polite"'
Select-String -Path .\index.html -Pattern 'prefers-reduced-motion'
```

Expected: 檔案存在，四個搜尋各至少命中一次。

在瀏覽器主控台執行：

```js
console.assert(escapeAngleBrackets('<div>Hello</div>') === '&lt;div&gt;Hello&lt;/div&gt;');
console.assert(escapeAngleBrackets('A & B') === 'A & B');
console.assert(escapeAngleBrackets('<<tag>>') === '&lt;&lt;tag&gt;&gt;');
console.assert(escapeAngleBrackets('plain text') === 'plain text');
```

Expected: 無 assertion error。

- [ ] **Step 7: 進行瀏覽器互動與視覺驗證**

以桌面與約 390px 手機寬度檢查：輸入、空白提示、轉換、清除、複製成功／失敗訊息、Tab 鍵焦點順序、兩欄到單欄的響應式切換，以及沒有水平捲動。若發現版面或互動問題，直接修正 `index.html` 後重跑 Step 6 與本步驟。

- [ ] **Step 8: 記錄完成狀態**

由於目前目錄不是 Git repository，不執行提交。最終交付須列出 `index.html` 路徑、驗證結果與可直接雙擊開啟使用的方式。
