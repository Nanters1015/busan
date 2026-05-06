# codebase.md — index.html 架構速查

單一靜態 HTML 檔，所有 CSS / HTML / JavaScript 都在 `index.html`。沒有 build 工具與 npm 依賴，直接用瀏覽器開啟即可預覽；正式部署為 Netlify 靜態網站。內容語系為繁體中文（zh-TW），主題是釜山 4 天 3 夜團體旅遊行程。

---

## 版型骨架

```html
<header>                  <!-- 藍色漸層標題列 -->
<div class="page-toolbar"> <!-- 編輯版面 / 重置版面 -->
<div class="dashboard">
  <div id="fixed-area">    <!-- D1-D4 固定行程卡，不進 Sortable -->
  <div id="sortable-area"> <!-- 其他資訊卡，可拖曳、刪除、調整欄寬 -->
<footer>
<div id="modal">           <!-- 單一共用 modal，openModal() 動態換內容 -->
<script src="SortableJS CDN">
<script async src="Instagram embed.js">
<script>...</script>
```

---

## Grid 與卡片

| 區域 | 行為 |
|------|------|
| `#fixed-area` | 4 欄 grid；D1-D4 卡片固定，不可拖曳 |
| `#sortable-area` | 3 欄 grid；資訊卡支援拖曳排序、刪除、欄寬切換 |

卡片欄寬 class：`col-1` / `col-2` / `col-3`。  
響應式斷點：`<=1200px` 變 2 欄；`<=480px` 變單欄。

### 卡片通用結構

```html
<div class="card col-N" id="card-xxx" [data-fixed]>
  <div class="card-header">
    <div class="card-header-left">
      <span class="card-title-text">標題</span>
      <button class="ig-add-btn" onclick="openModal('mode')">＋ 新增</button>
    </div>
    <div class="card-toolbar">
      <button class="card-resize" onclick="resizeCard('card-xxx')">⊞</button>
      <span class="drag-handle">⠿</span>
      <button class="card-delete" onclick="deleteCard('card-xxx')">✕</button>
    </div>
  </div>
  <div class="card-body">...</div>
</div>
```

- `data-fixed`：固定卡片，不進 `SortableJS`，不顯示一般 toolbar。
- `body.edit-mode`：顯示 toolbar、D1-D4 的新增/編輯按鈕。
- `resizeCard(id)`：非鎖定卡片循環切換 `col-1/2/3`。
- 鎖定欄寬卡片：`card-checklist`、`card-video`、`card-ig`、`card-thread`。
- 底部固定順序卡片：`card-video`、`card-ig`、`card-thread` 會被 `enforceLockedCardLayout()` 移到 sortable 區尾端。

---

## 卡片清單

### Fixed area（D1-D4）

| ID | 標題 | 主色 | Timeline |
|----|------|------|----------|
| `card-d1` | 廣安里 — 抵達放鬆日 | 藍 `#378ADD` | `#timeline-d1` |
| `card-d2` | 海雲台 — 雲海台觀景日 | 綠 `#2db87a` | `#timeline-d2` |
| `card-d3` | 機張＋西面 — 瘋玩購物日 | 紫 `#9966e0` | `#timeline-d3` |
| `card-d4` | 南浦洞 — 掃貨返程日 | 紅 `#e05252` | `#timeline-d4` |

Day 卡內容順序：`card-header` → `.hotel-bar`（D1-D3）→ `.car-bar` → 可選 `.tip-box` → `.timeline`。  
每張 Day 卡在 edit mode 顯示：

- `openModal('timeline', day)`：新增行程。
- `openModal('hotel-bar', day)`：編輯住宿列（D1-D3）。
- `openModal('car-bar', day)`：編輯交通列。

### Sortable area

| ID | 標題 | 預設 col | 資料來源 / 容器 |
|----|------|----------|-----------------|
| `card-weather` | 九月出發資訊 | `col-1` | 靜態 `.weather-grid` |
| `card-checklist` | 出發前準備清單 | `col-2` | GAS `get_checklist` → `#checklist-body` |
| `card-ig` | IG 收藏地點 | `col-1` | GAS `get` + timeline IG mapping → `#ig-cards-container` |
| `card-budget` | 費用試算 | `col-2` | GAS `get_budget` → `#budget-tbody` |
| `card-taxi-estimate` | 計程車費用估算 | `col-3` | 靜態表格；舊 layout 沒有此卡時會自動插在 budget 後 |
| `card-transport` | 交通方案對比 | `col-3` | GAS `get_transport` → `#transport-tbody` |
| `card-flight` | 機票資訊 | `col-2` | GAS `get_flights` → `#flight-tbody`；另有截圖區 |
| `card-hotel-info` | 住宿資訊 | `col-2` | GAS `get_hotels_info` → `#hotel-info-body` |
| `card-thread` | Thread 收藏地點 | `col-1` | GAS `get_threads` → `#thread-cards-container` |
| `card-video` | 拍片靈感 | `col-1` | GAS `get_videos` → `#video-cards-container` |

---

## 主要元件

### Timeline

```text
.tl-item
  .tl-time
  .tl-dot-wrap (.tl-dot + .tl-line)
  .tl-body (.tl-name + .tl-desc + .tags + .tl-del-btn)
```

資料模型：

```js
{
  id: 'd1-1',
  time: '上午',
  name: '文字標題',
  nameHtml: '可選 HTML 標題',
  desc: '描述',
  dotColor: '#378ADD',
  tags: [['tag-food', '餐食']],
  priceTag: 'NT$...',
  naverUrl: 'nmap://...'
}
```

- `getTimelineItems(day)` 先讀 `localStorage.busanTimeline_{day}`，沒有才用 `DEFAULT_TIMELINE`。
- `renderTimeline(day, items)` 會同步抽出 `tag-ig` 行程到 `window._timelineIgItems`，讓 IG 卡顯示「來自行程」唯讀卡。
- `deleteTlItem(id, day)` / `submitAddTimeline(day)` 只寫 localStorage，不寫 GAS。

### 視覺元件

- `.hotel-bar` / `.car-bar`：Day 卡內的住宿與交通資訊列。
- `.tip-box`：黃色左邊框提示。
- `.weather-grid` / `.weather-card`：九月出發資訊。
- `.ig-card`：IG / Thread / Video 共用收藏卡。
- `.ig-embed-wrap`：Instagram 或 Threads embed 容器。
- `.budget-table`：費用、交通、航班、計程車表格共用表格樣式。
- `.flight-screenshots-grid`：機票卡月費用截圖 2 欄 grid。
- `.item-del-btn`：表格與列表刪除按鈕。

### Tag classes

`tag-food`、`tag-spot`、`tag-shop`、`tag-night`、`tag-spa`、`tag-note`、`tag-ig`、`tag-car`、`tag-thrill`、`tag-outbound`、`tag-return`。  
`price-tag` 是橘色價格標籤；`naver-link` 是 Naver Map 連結按鈕。

---

## Modal

共用 modal：`#modal`、`#modal-title`、`#modal-fields`、`#modal-submit-btn`。  
`openModal(mode, extra)` 從 `MODAL_CONFIGS` 讀設定，`submitModal()` 依 `currentModalMode` 分派 submit 函式。

| mode | 用途 | submit |
|------|------|--------|
| `ig` | 新增 IG 收藏地點 | `submitAddSpot` |
| `checklist` | 新增準備項目 | `submitAddChecklist` |
| `budget` | 新增費用 | `submitAddBudget` |
| `transport` | 新增交通對比 | `submitAddTransport` |
| `timeline` | 新增 D1-D4 行程 | `submitAddTimeline(currentModalExtra)` |
| `hotel-bar` | 編輯 Day 住宿列 | `submitHotelBar` |
| `car-bar` | 編輯 Day 交通列 | `submitCarBar` |
| `flight` | 新增航班 | `submitAddFlight` |
| `flight-screenshot` | 新增機票截圖 | `submitAddFlightScreenshot` |
| `hotel-info` | 新增住宿資訊卡資料 | `submitAddHotelInfo` |
| `thread` | 新增 Thread 地點 | `submitAddThreadSpot` |
| `video` | 新增拍片靈感 | `submitAddVideo` |

---

## GAS 串接

```js
const GAS_URL = 'https://script.google.com/macros/s/AKfycbyCHxzRostMcu7OFiAJYqb0Q188qF_wYDMoODfE9os5NZ9INQTD_w4z-UXmS-78Q8B4Gg/exec';
```

Google Sheets ID：`1ZXw0zHHBW48ApYBE070nGuB5ZGsTE_lvmVtQIMNLmVs`

### Actions

| action | 用途 |
|--------|------|
| `get` / `add` / `delete` | IG 收藏地點 |
| `get_checklist` / `add_checklist` / `delete_checklist` | 準備清單 |
| `get_budget` / `add_budget` / `delete_budget` | 費用試算 |
| `get_transport` / `add_transport` / `delete_transport` | 交通方案對比 |
| `get_layout` / `save_layout` | 共享版面設定 |
| `get_hotel_bar` / `set_hotel_bar` | Day 卡住宿列 |
| `get_car_bar` / `set_car_bar` | Day 卡交通列 |
| `get_headcount` / `set_headcount` | 費用試算人數 |
| `get_flights` / `add_flight` / `delete_flight` | 航班資料 |
| `get_flight_screenshots` / `add_flight_screenshot` / `delete_flight_screenshot` | 機票月費用截圖 |
| `get_hotels_info` / `add_hotel_info` / `delete_hotel_info` | 住宿資訊卡 |
| `get_threads` / `add_thread` / `delete_thread` | Thread 收藏地點 |
| `get_videos` / `add_video` / `delete_video` | 拍片靈感 |

### Sheet 欄位

| Tab | 欄位 |
|-----|------|
| `ig_spots` | `id, name, detail, day, icon, color1, color2, naverUrl, igUrl` |
| `checklist` | `id, label, note` |
| `budget` | `id, label, desc, price_min, price_max` |
| `transport` | `id, item, charter, taxi` |
| `flights` | `id, direction, airline, flight_no, date, depart_time, arrive_time, price_no_bag, price_bag, note` |
| `flight_screenshots` | `id, title, img_url` |
| `hotels_info` | `id, day, hotel_name, address, checkin, checkout, price_per_night, naverUrl, note` |
| `threads_spots` | `id, name, detail, day, icon, color1, color2, naverUrl, threadUrl` |
| `video_spots` | `id, name, detail, day, icon, color1, color2, naverUrl, igUrl, threadUrl` |
| `hotel_bar` | `id, day, hotel_name, hotel_detail` |
| `car_bar` | `id, day, car_title, car_desc` |
| `trip_settings` | `key, value`，目前用於 `headcount` 與 `layout` |

注意：目前 GAS 端慣例是 `else if` action 鏈搭配 generic row helper，不是 `switch/case`。

---

## JavaScript 函式索引

### Layout

```js
toggleEdit()
saveOrder()
deleteCard(id)
resetLayout()
applyCardOrder(order)
applyCardCols(cols)
loadState()
enforceLockedCardLayout()
resizeCard(id)
saveCardCols()
```

排版同步流程：

- `saveOrder()` 存 `busanCardOrder`、`busanCardCols`、`busanDeletedCards`，並 fire-and-forget 寫 GAS `save_layout`。
- `loadState()` 先用 localStorage 快速渲染，再永遠 fetch GAS `get_layout` 套用共享版面。
- `resetLayout()` 清本地排版、timeline 與航班排序快取，再從 GAS 讀回共享 layout 後 reload。
- `applyCardOrder()` 會補上舊 layout 可能缺少的 `card-taxi-estimate`。

### 資料載入與渲染

```js
loadIgSpots() / renderIgCards(spots)
loadChecklist() / renderChecklist(items)
loadBudget() / renderBudget(items)
loadTransport() / renderTransport(items)
loadHotelCarBars() / renderHotelBar(day, data) / renderCarBar(day, data)
loadFlights() / renderFlights(items)
loadFlightScreenshots() / renderFlightScreenshots(items)
loadHotelsInfo() / renderHotelsInfo(items)
loadThreadSpots() / renderThreadCards(spots)
loadVideoSpots() / renderVideoCards(spots)
```

### 新增 / 刪除 / 編輯

```js
submitAddSpot() / deleteIgSpot(id)
submitAddChecklist() / deleteChecklistItem(id)
submitAddBudget() / deleteBudgetItem(id)
submitAddTransport() / deleteTransportItem(id)
submitHotelBar() / submitCarBar()
submitAddFlight() / deleteFlightItem(id)
submitAddFlightScreenshot() / deleteFlightScreenshot(id)
submitAddHotelInfo() / deleteHotelInfoItem(id)
submitAddThreadSpot() / deleteThreadSpot(id)
submitAddVideo() / deleteVideoSpot(id)
submitAddTimeline(day) / deleteTlItem(id, day)
```

### 工具函式與狀態

```js
fetchWithRetry(url, timeout)
extractIgCode(url)
hasThreadUrl(url)
escHtml(s)
extractLeadingEmoji(s)
initHeadcountSelect()
loadHeadcount()
setHeadcount(val)
recalcBudgetPerPerson()
getFlightOrder()
applyFlightOrder()
saveFlightOrder()
initFlightSortable()
```

---

## localStorage 鍵值

| Key | 內容 |
|-----|------|
| `busanCardOrder` | sortable 卡片 ID 排序 |
| `busanCardCols` | `{ [cardId]: 'col-N' }` 欄寬 |
| `busanDeletedCards` | 已刪除 sortable 卡片 ID |
| `busanTimeline_1` - `busanTimeline_4` | 每日 timeline 覆蓋資料 |
| `busanCheck_{id}` | checklist 勾選狀態，值為 `'1'` |
| `busanFlightOrder` | 航班表格列排序 |

---

## 全域狀態

| 變數 | 用途 |
|------|------|
| `sortable` | sortable 卡片拖曳實例 |
| `flightSortable` | 航班列拖曳實例 |
| `isEditMode` | 是否處於編輯版面模式 |
| `currentModalMode` / `currentModalExtra` | modal submit 分派狀態 |
| `window._timelineIgItems` | timeline 中 `tag-ig` 項目快取 |
| `window._lastIgSpots` | 最近一次 GAS IG 地點結果 |
| `window._hotelCarData` | Day 住宿/交通列資料快取 |
| `window._budgetTotal` | 費用試算總額，供人均計算 |
| `window._threadsProcess` | Threads embed process 函式快取 |

---

## 特殊功能

### 費用試算人數

`initHeadcountSelect()` 動態產生 1-13 人選項，預設 12 人。`loadHeadcount()` 從 GAS 讀 `trip_settings.headcount`；`setHeadcount(val)` 寫回 GAS 並重新計算人均費用。

### 航班排序

航班列使用 SortableJS，拖曳 handle 是 `.flight-row-handle`。排序只存在 `localStorage.busanFlightOrder`，不寫 GAS。

### 機票截圖

`card-flight` 內有 `#flight-screenshots-section`，圖片 URL 建議指向 Netlify 上的 `/screenshots/檔名.jpg`。資料來源為 `flight_screenshots` sheet。

### Instagram / Threads embed

- IG：`extractIgCode()` 支援 `/p/` 與 `/reel/`，渲染後呼叫 `window.instgrm?.Embeds?.process()`。
- Threads：載入 `https://www.threads.com/embed.js`。因 Instagram 和 Threads 會共用 `window.instgrm.Embeds.process`，載入 Threads 前會暫時清空 IG process，載入後存成 `window._threadsProcess` 再還原。
- `card-video` 優先顯示 IG embed；沒有 IG 但有 Threads URL 時顯示 Threads embed。

### Day 顏色

```js
const DAY_COLORS = {
  1: { c1: '#1a3c5e', c2: '#378ADD', badge: '#378ADD' },
  2: { c1: '#1a5a3a', c2: '#2db87a', badge: '#2db87a' },
  3: { c1: '#5030a0', c2: '#9966e0', badge: '#9966e0' },
  4: { c1: '#8b2000', c2: '#e05252', badge: '#e05252' }
};
```

---

## 初始化流程

`DOMContentLoaded` 後依序：

```js
await loadState();
loadIgSpots();
loadChecklist();
loadBudget();
loadTransport();
loadHotelCarBars();
initHeadcountSelect();
loadHeadcount();
loadFlights();
loadFlightScreenshots();
loadHotelsInfo();
loadThreadSpots();
loadVideoSpots();
```

`loadState()` 會在其他資料載入前先渲染 timeline，因為 IG 卡需要合併 timeline 中的 `tag-ig` 項目。

---

## 外部依賴

| 用途 | URL |
|------|-----|
| SortableJS 1.15.2 | `https://cdn.jsdelivr.net/npm/sortablejs@1.15.2/Sortable.min.js` |
| Instagram embed | `https://www.instagram.com/embed.js` |
| Threads embed | `https://www.threads.com/embed.js`，需要時動態載入 |
