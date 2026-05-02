# codebase.md — index.html 架構速查

單一靜態 HTML 檔，inline CSS + JS，無 build 工具。直接開瀏覽器預覽。部署於 Netlify。

---

## 版型骨架

```
<header>               ← 藍色漸層標題列
<div.page-toolbar>     ← 右側「✏️ 編輯版面」/「↩ 重置版面」按鈕
<div.dashboard>
  <div#fixed-area>     ← 4欄 Grid，D1–D4 固定，不可拖曳（data-fixed）
  <div#sortable-area>  ← 3欄 Grid，資訊卡，可拖曳排序、可調整欄寬
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
      <!-- D1–D4 固定卡片有 tl-add-btn，edit mode 才顯示 -->
      <button class="ig-add-btn tl-add-btn" onclick="openModal('timeline', N)">＋ 新增行程</button>
    </div>
    <div class="card-toolbar">  <!-- 編輯模式才顯示（sortable 卡片） -->
      <button class="card-resize" onclick="resizeCard('card-xxx')" title="調整欄寬">⊞</button>
      <span class="drag-handle">⠿</span>
      <button class="card-delete" onclick="deleteCard('card-xxx')">✕</button>
    </div>
  </div>
  <div class="card-body"> ... </div>
</div>
```

`data-fixed` → 不進 SortableJS、不顯示 toolbar。  
`card-resize` → 點擊循環切換 col-1/2/3，欄寬存 `busanCardCols` localStorage。  
`tl-add-btn` → 僅 D1–D4，edit mode 顯示，呼叫 `openModal('timeline', day)`。

---

## 卡片清單

### Fixed area（D1–D4，不可拖曳）

| ID | 標題 | 主色 | Timeline ID |
|----|------|------|-------------|
| `card-d1` | 廣安里 — 抵達放鬆日 | 藍 `#378ADD` | `#timeline-d1` |
| `card-d2` | 海雲台 — 雲海台觀景日 | 綠 `#2db87a` | `#timeline-d2` |
| `card-d3` | 機張＋西面 — 瘋玩購物日 | 紫 `#9966e0` | `#timeline-d3` |
| `card-d4` | 南浦洞 — 掃貨返程日 | 紅 `#e05848` | `#timeline-d4` |

Day 卡內部：`card-header(.day-header-row + tl-add-btn)` → `.hotel-bar` → `.car-bar` → [`.tip-box`] → `.timeline#timeline-dN`

Timeline 由 `renderTimeline(day, items)` 動態渲染，資料優先讀 `busanTimeline_{day}` localStorage，否則用 `DEFAULT_TIMELINE` 種子資料。

### Sortable area（可拖曳、可調整欄寬）

| ID | 標題 | col | 資料來源 | 動態容器 |
|----|------|-----|----------|----------|
| `card-weather` | 🌤 九月出發資訊 | col-1 | 靜態 | — |
| `card-checklist` | 📋 出發前準備清單 | col-1 | GAS `get_checklist` | `#checklist-body` |
| `card-ig` | 📸 IG 收藏地點 | col-1 | GAS `get` + timeline mapping | `#ig-cards-container` |
| `card-budget` | 💰 費用試算 | col-2 | GAS `get_budget` | `#budget-tbody`, `#budget-total-text` |
| `card-transport` | 🚗 交通方案對比 | col-3 | GAS `get_transport` | `#transport-tbody` |
| `card-flight` | ✈️ 機票資訊 | col-3 | GAS `get_flights` | 動態 `<tbody>` |
| `card-hotel-info` | 🏨 住宿資訊 | col-2 | GAS `get_hotels_info` | 動態 `<tbody>` |
| `card-thread` | 🧵 Thread 收藏地點 | col-1 | GAS `get_threads` | `#thread-cards-container` |
| `card-video` | 🎬 拍片靈感 | col-1 | GAS `get_videos` | `#video-cards-container` |

---

## 內容元件

### Timeline（`.tl-item`）
```
.tl-time(42px) | .tl-dot-wrap(.tl-dot + .tl-line) | .tl-body(.tl-name + .tl-desc + .tags + .tl-del-btn)
```

- `.tl-del-btn` → hover 顯示刪除按鈕（position absolute），呼叫 `deleteTlItem(id, day)`
- 最後一個 tl-item 無 `.tl-line`

### Timeline 資料模型
```js
{
  id: 'd1-1',          // 字串，種子用 dN-N，user-added 用 dN-timestamp
  time: '下午',
  name: '🚐 ...',      // 純文字（渲染時 escHtml）
  nameHtml: '...',     // 可選，含 HTML（種子複雜項目用，直接 innerHTML）
  desc: '...',
  dotColor: '#378ADD',
  tags: [['tag-food','標籤文字'], ...],
  priceTag: 'NT$...',  // 可選
  naverUrl: 'nmap://...' // 可選
}
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
- `.ig-source-badge` → 藍色小標籤「📅 來自行程」（timeline mapping 來源，無刪除按鈕）
- `.ig-embed-wrap` → IG 貼文 embed 容器（支援 `/p/` 和 `/reel/`）
- `.budget-table` → 費用/交通表，`.budget-total` 合計列在 `<tfoot>`
- `.checklist-section` / `.check-item` → 可勾選清單
- `.item-del-btn` → hover 顯示刪除按鈕（checklist/budget/transport 共用）
- `.ig-add-btn` / `.tl-add-btn` → 卡片 header「＋ 新增」按鈕（共用樣式）
- `.card-resize` → sortable card toolbar 欄寬切換按鈕（edit mode 才顯示）

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

| mode | 欄位 IDs | title/fields 型態 |
|------|----------|------------------|
| `'ig'` | `f-name`, `f-detail`, `f-day`, `f-icon`, `f-naver`, `f-igurl` | 字串 |
| `'checklist'` | `f-label`, `f-note` | 字串 |
| `'budget'` | `f-label`, `f-desc`, `f-price-min`, `f-price-max` | 字串 |
| `'transport'` | `f-item`, `f-charter`, `f-taxi` | 字串 |
| `'timeline'` | `f-tl-time`, `f-tl-name`, `f-tl-desc`, `f-tl-naver`, tag checkboxes | **函式**（接收 day） |

`openModal(mode, extra)` — `extra` 為 day number（timeline mode 用）；`currentModalExtra` 儲存後傳給 `submitModal`。

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
| `get_videos` | 拍片靈感列表 | — |
| `add_video` | 新增拍片靈感 | `name,detail,day,icon,color1,color2,naverUrl,igUrl,threadUrl` |
| `delete_video&id=` | 刪除拍片靈感 | — |

### GAS doGet 寫法（重要：勿用 switch/case）

GAS 腳本用 `else if` 鏈，並透過 `addGenericRow(sheetName, p)` 新增資料：

```js
if (action === 'get_flights') {
  result = getGenericRows('flights');
} else if (action === 'add_flight') {
  result = addGenericRow('flights', p);
} else if (action === 'delete_flight') {
  result = deleteGenericRow('flights', p.id);
} else if (action === 'get_layout') {
  result = getLayoutSetting();
} else if (action === 'save_layout') {
  result = saveLayoutSetting(p.data);
}
```

**新增 action 時一律用 `else if`，不用 `case`。**

### Sheet 欄位

| Tab | 欄位 |
|-----|------|
| `ig_spots` | `id, name, detail, day, icon, color1, color2, naverUrl, igUrl` |
| `checklist` | `id, label, note` |
| `budget` | `id, label, desc, price_min, price_max` |
| `transport` | `id, item, charter, taxi` |
| `flights` | `id, direction, airline, flight_no, date, depart_time, arrive_time, price_no_bag, price_bag, note` |
| `hotels_info` | `id, day, hotel_name, address, checkin, checkout, price_per_night, naverUrl, note` |
| `threads_spots` | `id, name, detail, day, icon, color1, color2, naverUrl, threadUrl` |
| `video_spots` | `id, name, detail, day, icon, color1, color2, naverUrl, igUrl, threadUrl` |
| `hotel_bar` | `id, day, hotel_name, hotel_detail` |
| `car_bar` | `id, day, car_title, car_desc` |
| `trip_settings` | `key, value`（headcount, layout 兩個 key） |

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
toggleEdit()          // 切換 body.edit-mode；啟用/銷毀 SortableJS
saveOrder()           // 儲存排序 + 欄寬 + deletedCards 到 localStorage，同時 fire-and-forget 寫 GAS save_layout
deleteCard(id)        // 移除卡片 + 記錄 localStorage
resetLayout()         // 清所有 localStorage → 從 GAS 讀回共享排版（含 deleted） → reload（async）
loadState()           // 快速初始渲染 localStorage 快取，永遠同步 GAS 共享排版；GAS 失敗時 fallback localhost（async）
applyCardOrder(order) // 將 order 陣列套用到 sortable-area DOM
applyCardCols(cols)   // 將 cols 物件套用到各卡片 className
resizeCard(id)        // 循環切換 card 的 col-1/2/3，呼叫 saveCardCols()
saveCardCols()        // 把 sortable-area 各 card 的欄寬存入 busanCardCols
```

### Modal
```js
openModal(mode, extra) // 注入欄位、顯示 modal；extra = day（timeline mode）
closeModal()           // 隱藏 modal
submitModal()          // 依 currentModalMode 分派 submit
```

### Timeline（D1–D4 行程）
```js
DEFAULT_TIMELINE       // 物件 {1:[...], 2:[...], 3:[...], 4:[...]}，種子資料
getTimelineItems(day)  // 讀 localStorage busanTimeline_{day}，無則用 DEFAULT_TIMELINE
renderTimeline(day, items) // 渲染 #timeline-dN；同時提取 tag-ig 項目到 _timelineIgItems
deleteTlItem(id, day)  // 刪除後存 localStorage，re-render timeline + IG 區
submitAddTimeline(day) // modal 欄位 → 存 localStorage → re-render
escHtml(s)             // HTML 跳脫工具函式
extractLeadingEmoji(s) // 取字串開頭 emoji，用於 IG mapping icon
```

### IG 地點
```js
extractIgCode(url)     // 抓 /p/ 或 /reel/ shortcode；無則回 null
renderIgCards(spots)   // 合併 _timelineIgItems（唯讀）+ GAS spots 渲染
                       // timeline 來源有 .ig-source-badge，無刪除按鈕
loadIgSpots()          // fetch get → 存 _lastIgSpots → renderIgCards
submitAddSpot()        // modal 欄位 → fetch add → loadIgSpots
deleteIgSpot(id)       // fetch delete → loadIgSpots
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

### 拍片靈感
```js
loadVideoSpots()         // fetch get_videos → renderVideoCards
renderVideoCards(spots)  // 渲染 .ig-card；igUrl → IG embed；threadUrl（無 igUrl 時）→ Thread embed
submitAddVideo()         // modal 欄位 → fetch add_video → loadVideoSpots
deleteVideoSpot(id)      // fetch delete_video → loadVideoSpots
```

---

## localStorage 鍵值

| Key | 內容 |
|-----|------|
| `busanCardOrder` | `string[]` 卡片 ID 排列順序；從 GAS 同步回來的快取 |
| `busanDeletedCards` | `string[]` 已刪除的卡片 ID；從 GAS `trip_settings.layout` 同步 |
| `busanCardCols` | `{[cardId]: 'col-N'}` 各 sortable card 的欄寬；從 GAS 同步回來的快取 |
| `busanTimeline_{day}` | `TlItem[]` 第 N 天行程（1–4），覆蓋 DEFAULT_TIMELINE |
| `busanCheck_{id}` | `'1'` 表示該清單項目已勾選 |

---

## 全域狀態變數

| 變數 | 用途 |
|------|------|
| `window._timelineIgItems` | timeline tag-ig 項目快取，供 renderIgCards 使用 |
| `window._lastIgSpots` | 最後一次 GAS 回傳的 IG 地點，timeline 變更後 re-render 用 |

---

## 外部依賴

| 用途 | 載入 |
|------|------|
| SortableJS 1.15.2 | `cdn.jsdelivr.net/npm/sortablejs@1.15.2` |
| Instagram embed.js | `https://www.instagram.com/embed.js`（async） |

---

## 機票資訊卡（card-flight）

表格欄位（7欄）：日期 | 方向 | 航空/班次 | 出發→抵達 | 無托運 | 有托運(20kg) | del

GAS `flights` schema：`id, direction, airline, flight_no, date, depart_time, arrive_time, price_no_bag, price_bag, note`

向後相容：舊 `price` 欄在 `price_no_bag` 為空時 fallback 顯示在無托運欄。

---

## Thread 收藏地點卡（card-thread）

Embed 方式：`<blockquote class="text-post-media" data-text-post-permalink="URL">` +
動態載入 `https://www.threads.com/embed.js`（**非** `threads.net`，舊路徑已 404）。

**注意**：Instagram 與 Threads 共用 `window.instgrm.Embeds.process` namespace。
Instagram embed.js 會搶先設好此函式，導致 Threads 的 init() 跳過 R()。
解法：載入 threads.com/embed.js 前暫時清空 `process`，onload 後將 Threads 的 R()
存成 `window._threadsProcess`，並還原 Instagram 的 process。

`hasThreadUrl(url)` — 判斷 threadUrl 是否合法（支援 threads.net / threads.com）。

Embed 流程（renderThreadCards 尾端）：
- `window._threadsProcess` 已存在 → 直接呼叫（後續重渲染用）
- 否則暫存 IG process、清空、插入 script[data-threads-embed]、onload 存 _threadsProcess 並還原 IG process

---

## 排版同步（GAS 共享）

`busanCardOrder` / `busanCardCols` / `busanDeletedCards` 除存 localStorage 外，同步至 GAS `trip_settings.layout`（JSON 字串 `{order, cols, deleted}`）。

- `saveOrder()` 每次呼叫同時 fire-and-forget `save_layout`（payload 含 order / cols / deleted）
- `loadState()` **永遠** fetch `get_layout` 同步共享排版；localStorage 作快速初始渲染快取（async）
- `resetLayout()` 清 localStorage 後從 GAS 讀回共享排版（含 deleted）再 reload

GAS actions：`get_layout`、`save_layout&data=JSON`（寫入 trip_settings.layout）
