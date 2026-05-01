# 釜山行程頁面 — Dashboard 重構任務清單

## 目標
把目前的單欄捲動頁面改成全寬可拖曳 Dashboard，支援自由新增卡片。
頁面定位：一目瞭然看清這次旅遊全貌，方便大家一起討論。

## 已確認決策
- **D1～D4 行程卡各佔 1 欄，固定不可拖曳**
- **其餘資訊卡（天氣、預算、清單、IG、交通等）可自由拖曳調整位置**
- **匯出功能不做**，排列存 localStorage 即可
- **新增卡片** 是核心功能，要支援

---

## Phase 1：版型基礎

- [ ] 移除 `max-width: 800px`，改成全寬 CSS Grid（3 欄為主）
- [ ] 定義三種卡片尺寸：`col-1`（1欄）、`col-2`（2欄）、`col-3`（全寬）
- [ ] 把現有所有 `.section` 遷移進 Grid 容器，指定初始欄寬
- [ ] 響應式：768px 以下降為 2 欄，480px 以下單欄
- [ ] 設計第一屏預設排版：
  - 上半區：D1／D2／D3／D4 各佔 1 欄，固定排滿第一行
  - 下半區：資訊卡（天氣、預算、清單、IG、交通）可拖曳的 Grid 區域

---

## Phase 2：卡片元件統一化

- [ ] 統一所有卡片 HTML 結構：`card-header`（標題＋工具列）＋ `card-body`（內容）
- [ ] 定義卡片類型（每種有對應模板）：
  - `type-day` — 行程時間軸（D1～D4）
  - `type-checklist` — 準備清單
  - `type-budget` — 費用試算表
  - `type-weather` — 天氣資訊
  - `type-transport` — 交通方案對比
  - `type-ig` — IG 收藏卡片組
  - `type-note` — 自訂文字/提示框
- [ ] 每張卡片右上角加工具列按鈕區（拖曳手把、刪除）

---

## Phase 3：拖曳功能

- [ ] 引入 SortableJS（CDN，不需安裝）
- [ ] 實作「編輯模式」toggle 按鈕（預設關閉，開啟才可拖曳）
- [ ] 拖曳時高亮 drop zone
- [ ] 拖曳完成後更新 Grid 排列（行程卡 D1-D4 設 `data-fixed` 不進 SortableJS）
- [ ] 用 `localStorage` 儲存目前卡片順序，重整頁面保留排列

---

## Phase 4：新增卡片 UI

- [ ] 畫面右下角固定「＋ 新增卡片」按鈕（FAB 樣式）
- [ ] 點擊後開啟卡片類型選擇器（modal 或 side panel）
- [ ] 選擇類型後插入對應空白模板卡片到 Grid
- [ ] 新增的卡片資料也存進 `localStorage`

---

## Phase 5：卡片內容編輯

- [ ] 雙擊卡片標題可以 inline 編輯（`contenteditable`）
- [ ] `type-note` 卡片：雙擊 body 可直接編輯文字內容
- [ ] 其他類型（行程、費用表）：先做刪除＋複製，內容編輯列為 v2
- [ ] 每張卡右上角加「🗑 刪除」按鈕（編輯模式才顯示）

---

## Phase 6：資料持久化

- [ ] 所有卡片狀態（順序、欄寬、自訂內容）存到 `localStorage`
- [ ] 加「↩ 重置版面」按鈕（恢復預設排列）
- [ ] 「↩ 重置版面」按鈕恢復預設排列即可，不做匯出

---

## 技術選型

| 需求 | 方案 |
|------|------|
| 拖曳排列 | SortableJS CDN |
| 資料儲存 | localStorage（不需後端） |
| 樣式 | 現有 inline CSS 擴充，不引入 framework |
| 編輯 | contenteditable（原生，不需 library） |

---

## 備註

- 所有功能維持在單一 `index.html` 內，不拆檔
- SortableJS 只在編輯模式啟用，不影響分享給其他人的閱讀體驗
- Phase 1–3 是核心，Phase 4–6 可分批做
