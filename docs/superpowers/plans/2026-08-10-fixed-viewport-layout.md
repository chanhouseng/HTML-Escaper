# Fixed Viewport Layout Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 讓尖括號轉換器固定在單一瀏覽器視窗內，禁止頁面垂直捲動，並保留文字區內部捲動能力。

**Architecture:** 僅調整 `index.html` 的 CSS。根節點鎖定為動態視窗高度，主容器以 grid 分配標題、工具卡與提示的有限高度；工具卡與文字區使用 `min-height: 0` 允許內容在內部縮放與捲動。

**Tech Stack:** HTML5、CSS3、原生 JavaScript（不修改）

## Global Constraints

- 頁面使用 `100dvh`，`html` 與 `body` 禁止整頁捲動。
- 文字區保留內容輸入與內部捲動功能。
- 桌面與窄螢幕均不可有水平或垂直頁面溢出。
- 不改變 `<`、`>` 轉換與複製行為。

---

### Task 1: 固定視窗高度並驗證

**Files:**
- Modify: `index.html:28-340`

**Interfaces:**
- Consumes: 瀏覽器動態視窗高度 `100dvh`
- Produces: 固定頁面視窗與可捲動的文字輸入框

- [ ] **Step 1: 建立失敗驗證**

在 1280×800 與 390×844 瀏覽器尺寸，讀取：

```js
({ scrollHeight: document.documentElement.scrollHeight,
   clientHeight: document.documentElement.clientHeight })
```

Expected before implementation: 任一尺寸 `scrollHeight > clientHeight`，代表頁面可垂直捲動。

- [ ] **Step 2: 實作固定視窗 CSS**

調整 CSS：

```css
html, body { height: 100%; overflow: hidden; }
body { height: 100dvh; }
.shell { height: 100dvh; display: grid; grid-template-rows: auto minmax(0, 1fr) auto; }
.tool-card, .workspace, .editor-panel { min-height: 0; }
textarea { min-height: 0; }
```

同步縮小各斷點的容器內距、標題與文字區高度，確保工具卡與提示仍留在視窗內。

- [ ] **Step 3: 驗證固定高度與核心功能**

重新檢查兩個尺寸：

```js
document.documentElement.scrollHeight === document.documentElement.clientHeight
```

Expected: `true`。再輸入 `<tag>` 並點選轉換，結果必須是 `&lt;tag&gt;`；在輸入框放入長文字時，`textarea.scrollHeight` 可大於 `textarea.clientHeight`，表示捲動只發生於文字區。

- [ ] **Step 4: 記錄完成狀態**

此工作區未初始化 Git，跳過提交；回報 `index.html` 已更新以及桌面、手機兩種尺寸的量測結果。
