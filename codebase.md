# index.html 結構說明

單一靜態 HTML 檔，內含 inline CSS + JS，無任何 build 工具。直接用瀏覽器開啟預覽。

---

## 整體版型

```
<header>               ← 藍色漸層標題列
<div.page-toolbar>     ← 右側「✏️ 編輯版面」/ 「↩ 重置版面」按鈕
<div.dashboard>
  <div#fixed-area>     ← 4欄 CSS Grid，D1–D4 固定在此，不可拖曳
  <div#sortable-area>  ← 3欄 CSS Grid，資訊卡在此，可拖曳
<footer>
```

---

## Grid 系統

| 容器 | 預設欄數 | 說明 |
|------|----------|------|
| `#fixed-area` | 4 欄 | D1~D4 各佔 `col-1` |
| `#sortable-area` | 3 欄 | 資訊卡用 `col-1/2/3` |

欄寬 class：`col-1`（1欄）、`col-2`（2欄）、`col-3`（全寬）

響應式斷點：
- `≤1200px` → fixed/sortable 各 2 欄，`col-3` 縮為 `span 2`
- `≤768px`  → 同上，`col-2/3` 皆 `span 2`
- `≤480px`  → 全部單欄

---

## 卡片結構（統一格式）

```html
<div class="card col-N" id="card-xxx" [data-fixed]>
  <div class="card-header">
    <div class="card-header-left">
      <!-- 標題或 .day-header-row -->
    </div>
    <div class="card-toolbar">   <!-- 編輯模式才顯示 -->
      <span class="drag-handle">⠿</span>
      <button class="card-delete" onclick="deleteCard('card-xxx')">✕</button>
    </div>
  </div>
  <div class="card-body">
    <!-- 內容 -->
  </div>
</div>
```

`data-fixed` → 該卡片不進入 SortableJS、不顯示 toolbar。

---

## 現有卡片清單

### Fixed area（D1–D4，不可拖曳）

| ID | 標題 | 顏色 |
|----|------|------|
| `card-d1` | 廣安里 — 抵達放鬆日 | 藍 `#378ADD` |
| `card-d2` | 海雲台 — 雲海台觀景日 | 綠 `#2db87a` |
| `card-d3` | 機張＋西面 — 瘋玩購物日 | 紫 `#9966e0` |
| `card-d4` | 南浦洞 — 掃貨返程日 | 紅 `#e05848` |

每張 Day 卡內部結構：`card-header(.day-header-row)` → `.hotel-bar` → `.car-bar` → [`.tip-box`] → `.timeline`

### Sortable area（可拖曳）

| ID | 標題 | col |
|----|------|-----|
| `card-weather` | 🌤 九月出發資訊 | col-1 |
| `card-checklist` | 📋 出發前準備清單 | col-1 |
| `card-ig` | 📸 IG 收藏地點 | col-1 |
| `card-budget` | 💰 費用試算 | col-2 |
| `card-transport` | 🚗 交通方案對比 | col-3 |

---

## 內容元件

### Timeline（`.tl-item`）
```
.tl-time(42px) | .tl-dot-wrap(.tl-dot + .tl-line) | .tl-body(.tl-name + .tl-desc + .tags)
```

### Tag pills
`.tag-food` `.tag-spot` `.tag-shop` `.tag-night` `.tag-spa` `.tag-note` `.tag-ig` `.tag-car` `.tag-thrill`

`.price-tag` → 橘色價格標籤  
`.naver-link` → 綠色 Naver Map 連結按鈕（`nmap://` scheme）

### 其他元件
- `.hotel-bar` → 淡藍底飯店資訊列
- `.car-bar` → 淡藍底包車資訊列
- `.tip-box` → 黃色左邊框提示框
- `.weather-grid` → 3欄天氣格，`.weather-card`
- `.ig-cards` / `.ig-card` → IG 收藏卡片（含漸層色彩圖）
- `.budget-table` → 費用表，`.budget-total` 為合計列
- `.checklist-section` / `.check-item` → 可勾選清單（`toggleCheck()` 互動）

---

## JavaScript 行為

### 編輯模式
```js
toggleEdit()    // 切換 body.edit-mode class；啟用/銷毀 SortableJS
```
- `edit-mode` class → `.card:not([data-fixed]) .card-toolbar` 變為可見
- `edit-mode` class → 可拖曳卡片出現藍色虛線 outline

### SortableJS（CDN `sortablejs@1.15.2`）
- 掛載於 `#sortable-area`
- `handle: '.drag-handle'`
- `ghostClass: 'sortable-ghost'` / `chosenClass: 'sortable-chosen'`
- 拖曳結束呼叫 `saveOrder()`

### localStorage 鍵值
| Key | 內容 |
|-----|------|
| `busanCardOrder` | `string[]` 卡片 id 排列順序 |
| `busanDeletedCards` | `string[]` 已刪除的卡片 id |

### 函式一覽
```js
toggleEdit()        // 編輯模式開關
saveOrder()         // 儲存 sortable-area 子元素順序
deleteCard(id)      // 移除卡片並記錄到 localStorage
resetLayout()       // 清除 localStorage 並 reload
toggleCheck(box)    // checklist 勾選視覺切換
loadState()         // DOMContentLoaded 時還原刪除/排序狀態
```

---

## 已完成進度（對應 tasks.md）

- [x] Phase 1：全寬 Grid，`col-1/2/3`，D1–D4 固定第一行，響應式
- [x] Phase 2：統一 `card-header` + `card-body` 結構，toolbar 僅編輯模式顯示
- [x] Phase 3：SortableJS 拖曳，localStorage 儲存順序與刪除狀態，重置版面
- [ ] Phase 4：新增卡片（FAB + modal）
- [ ] Phase 5：卡片內容編輯（contenteditable）
- [ ] Phase 6：資料持久化補強
