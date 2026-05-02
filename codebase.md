# codebase.md — index.html 架構速查

單一靜態 HTML 檔，inline CSS + JS，無 build 工具。直接開瀏覽器預覽。部署於 Netlify。

---

## 版型骨架

```
<header>               ← 藍色漸層標題列
<div.page-toolbar>     ← 右側「✏️ 編輯版面」/「↩ 重置版面」按鈕
<div.dashboard>
  <div#fixed-area>     ← 4欄 Grid，D1–D4 固定，不可拖曳（data-fixed）
  <div#sortable-area>  ← 3欄 Grid，資訊卡，可拖曳排序
<footer>
<div#modal>            ← 通用 modal（單一，動態切換模式）
<script src="sortablejs CDN">
<script async src="instagram embed.js">
<script> ... 所有 JS ... </script>
```

---

## Grid 系統

| 容器 | 預設欄數 |
|------|----------|
| `#fixed-area` | 4 欄，D1~D4 各 `col-1` |
| `#sortable-area` | 3 欄，資訊卡用 `col-1/2/3` |

響應式斷點：`≤1200px` → 2欄 / `≤768px` → 2欄 / `≤480px` → 單欄

---

## 卡片通用結構

```html
<div class="card col-N" id="card-xxx" [data-fixed]>
  <div class="card-header">
    <div class="card-header-left">
      <span class="card-title-text">標題</span>
      <button class="ig-add-btn" onclick="openModal('mode')">＋ 新增</button>  <!-- 動態卡片才有 -->
    </div>
    <div class="card-toolbar">  <!-- 編輯模式才顯示 -->
      <span class="drag-handle">⠿</span>
      <button class="card-delete" onclick="deleteCard('card-xxx')">✕</button>
    </div>
  </div>
  <div class="card-body"> ... </div>
</div>
```

`data-fixed` → 不進 SortableJS、不顯示 toolbar。

---

## 卡片清單

### Fixed area（D1–D4，不可拖曳）

| ID | 標題 | 主色 |
|----|------|------|
| `card-d1` | 廣安里 — 抵達放鬆日 | 藍 `#378ADD` |
| `card-d2` | 海雲台 — 雲海台觀景日 | 綠 `#2db87a` |
| `card-d3` | 機張＋西面 — 瘋玩購物日 | 紫 `#9966e0` |
| `card-d4` | 南浦洞 — 掃貨返程日 | 紅 `#e05848` |

Day 卡內部：`card-header(.day-header-row)` → `.hotel-bar` → `.car-bar` → [`.tip-box`] → `.timeline`

### Sortable area（可拖曳）

| ID | 標題 | col | 資料來源 | 動態容器 |
|----|------|-----|----------|----------|
| `card-weather` | 🌤 九月出發資訊 | col-1 | 靜態 | — |
| `card-checklist` | 📋 出發前準備清單 | col-1 | GAS `get_checklist` | `#checklist-body` |
| `card-ig` | 📸 IG 收藏地點 | col-1 | GAS `get` | `#ig-cards-container` |
| `card-budget` | 💰 費用試算 | col-2 | GAS `get_budget` | `#budget-tbody`, `#budget-total-text` |
| `card-transport` | 🚗 交通方案對比 | col-3 | GAS `get_transport` | `#transport-tbody` |

---

## 內容元件

### Timeline（`.tl-item`）
```
.tl-time(42px) | .tl-dot-wrap(.tl-dot + .tl-line) | .tl-body(.tl-name + .tl-desc + .tags)
```

### Tag pills
`.tag-food` `.tag-spot` `.tag-shop` `.tag-night` `.tag-spa` `.tag-note` `.tag-ig` `.tag-car` `.tag-thrill`

`.price-tag` → 橘色價格標籤  
`.naver-link` → 綠色 Naver Map 連結（`nmap://` scheme）

### 其他元件
- `.hotel-bar` / `.car-bar` → 淡藍底資訊列
- `.tip-box` → 黃色左邊框提示框
- `.weather-grid` / `.weather-card` → 天氣格
- `.ig-card` → IG 地點卡（漸層色彩圖 + hover 顯示 `.ig-card-del`）
- `.ig-embed-wrap` → IG 貼文 embed 容器（支援 `/p/` 和 `/reel/`）
- `.budget-table` → 費用/交通表，`.budget-total` 合計列在 `<tfoot>`
- `.checklist-section` / `.check-item` → 可勾選清單
- `.item-del-btn` → hover 顯示刪除按鈕（checklist/budget/transport 共用）
- `.ig-add-btn` → 卡片 header「＋ 新增」按鈕（共用樣式）

---

## 通用 Modal（`#modal`）

```html
<div class="modal-overlay" id="modal">
  <div class="modal">
    <div class="modal-title" id="modal-title"></div>
    <div id="modal-fields"></div>   <!-- openModal() 動態注入 -->
    <div class="modal-actions">
      <button onclick="closeModal()">取消</button>
      <button id="modal-submit-btn" onclick="submitModal()">新增</button>
    </div>
  </div>
</div>
```

| mode | 欄位 IDs |
|------|----------|
| `'ig'` | `f-name`, `f-detail`, `f-day`, `f-icon`, `f-naver`, `f-igurl` |
| `'checklist'` | `f-label`, `f-note` |
| `'budget'` | `f-label`, `f-desc`, `f-price-min`, `f-price-max` |
| `'transport'` | `f-item`, `f-charter`, `f-taxi` |

---

## GAS 串接

```js
const GAS_URL = 'https://script.google.com/macros/s/AKfycbyCHxzRostMcu7OFiAJYqb0Q188qF_wYDMoODfE9os5NZ9INQTD_w4z-UXmS-78Q8B4Gg/exec';
```

Sheets ID：`1ZXw0zHHBW48ApYBE070nGuB5ZGsTE_lvmVtQIMNLmVs`

### action 一覽

| action | 說明 | params |
|--------|------|--------|
| `get` | IG 地點列表 | — |
| `add` | 新增 IG 地點 | `name,detail,day,icon,color1,color2,naverUrl,igUrl` |
| `delete&id=` | 刪除 IG 地點 | — |
| `get_checklist` | 清單列表 | — |
| `add_checklist` | 新增清單項目 | `label,note` |
| `delete_checklist&id=` | 刪除清單項目 | — |
| `get_budget` | 費用列表 | — |
| `add_budget` | 新增費用 | `label,desc,price_min,price_max` |
| `delete_budget&id=` | 刪除費用 | — |
| `get_transport` | 交通對比列表 | — |
| `add_transport` | 新增對比列 | `item,charter,taxi` |
| `delete_transport&id=` | 刪除對比列 | — |

### Sheet 欄位

| Tab | 欄位 |
|-----|------|
| `ig_spots` | `id, name, detail, day, icon, color1, color2, naverUrl, igUrl` |
| `checklist` | `id, label, note` |
| `budget` | `id, label, desc, price_min, price_max` |
| `transport` | `id, item, charter, taxi` |

### Day 顏色常數
```js
DAY_COLORS = {
  1: { c1:'#1a3c5e', c2:'#378ADD', badge:'#378ADD' },
  2: { c1:'#1a5a3a', c2:'#2db87a', badge:'#2db87a' },
  3: { c1:'#5030a0', c2:'#9966e0', badge:'#9966e0' },
  4: { c1:'#8b2000', c2:'#e05252', badge:'#e05252' }
}
```

---

## JavaScript 函式一覽

### 版型
```js
toggleEdit()       // 切換 body.edit-mode；啟用/銷毀 SortableJS
saveOrder()        // 儲存排序到 localStorage
deleteCard(id)     // 移除卡片 + 記錄 localStorage
resetLayout()      // 清 localStorage + reload
loadState()        // 還原刪除/排序（DOMContentLoaded）
```

### Modal
```js
openModal(mode)    // 注入欄位、顯示 modal
closeModal()       // 隱藏 modal
submitModal()      // 依 currentModalMode 分派 submit
```

### IG 地點
```js
extractIgCode(url)   // 抓 /p/ 或 /reel/ shortcode；無則回 null
renderIgCards(spots) // 渲染卡片；有 igUrl 插入 embed；呼叫 instgrm.Embeds.process()
loadIgSpots()        // fetch get → renderIgCards
submitAddSpot()      // modal 欄位 → fetch add → reload
deleteIgSpot(id)     // fetch delete → reload
```

### 清單
```js
loadChecklist()           // fetch get_checklist → renderChecklist
renderChecklist(items)    // 動態生成 .check-item；讀 localStorage 還原勾選
toggleCheckItem(id, box)  // 切換勾選，key: busanCheck_{id}
submitAddChecklist()      // fetch add_checklist → reload
deleteChecklistItem(id)   // fetch delete_checklist → reload
```

### 費用
```js
loadBudget()        // fetch get_budget → renderBudget
renderBudget(items) // 動態 <tr>；加總 price_min/max → #budget-total-text
submitAddBudget()   // fetch add_budget → reload
deleteBudgetItem(id)// fetch delete_budget → reload
```

### 交通
```js
loadTransport()          // fetch get_transport → renderTransport
renderTransport(items)   // 動態 <tr>；結論列在 <tfoot>（hardcoded）
submitAddTransport()     // fetch add_transport → reload
deleteTransportItem(id)  // fetch delete_transport → reload
```

---

## localStorage 鍵值

| Key | 內容 |
|-----|------|
| `busanCardOrder` | `string[]` 卡片 ID 排列順序 |
| `busanDeletedCards` | `string[]` 已刪除的卡片 ID |
| `busanCheck_{id}` | `'1'` 表示該清單項目已勾選 |

---

## 外部依賴

| 用途 | 載入 |
|------|------|
| SortableJS 1.15.2 | `cdn.jsdelivr.net/npm/sortablejs@1.15.2` |
| Instagram embed.js | `https://www.instagram.com/embed.js`（async） |
