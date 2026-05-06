# 釜山四天三夜行程規劃

單一檔案靜態網站，內容為繁體中文釜山 4 天 3 夜團體旅遊行程。主方案是 2026 年 6 月或 9 月出發，預設獨旅 1 人，也可切換到多人同行；已買釜山 Pass，交通以地鐵 / 步行優先，距離或步行時間不適合時保留計程車 / 包車。

## Preview

沒有 build tools 或 npm 依賴，直接用瀏覽器開啟 `index.html` 即可預覽。正式站台部署為 Netlify 靜態網站。

## Content

- D1-D4 固定行程卡：住宿、交通、timeline、每日費用小計
- 可拖曳資訊卡：六月 / 九月出發資訊、出發前清單、IG / Thread 收藏、費用試算、交通估算、航班、住宿資訊、拍片靈感
- 費用已扣除部分釜山 Pass 可覆蓋項目：Hotel Aqua Palace、BUSAN X the Sky、Skyline Luge、釜山樂天世界
- 天空膠囊暫不扣 Pass，因常見 Pass 覆蓋的是 Beach Train，不一定包含 Sky Capsule

## Data Source

動態資料由 Google Sheets + Google Apps Script 提供。

- Google Sheets ID: `1ZXw0zHHBW48ApYBE070nGuB5ZGsTE_lvmVtQIMNLmVs`
- GAS Web App: `https://script.google.com/macros/s/AKfycbyCHxzRostMcu7OFiAJYqb0Q188qF_wYDMoODfE9os5NZ9INQTD_w4z-UXmS-78Q8B4Gg/exec`

主要動態表：`ig_spots`、`checklist`、`budget`、`transport`、`flights`、`flight_screenshots`、`hotels_info`、`threads_spots`、`video_spots`、`hotel_bar`、`car_bar`、`trip_settings`。

## Current Budget Notes

`#card-budget` 透過 `get_budget` 載入資料。資料列以 1 人基準儲存，頁面會依人數調整每人估算與全團總額：機票 / Pass / 備品固定每人，住宿以雙人房分攤，計程車以每台最多 4 人分攤，吃玩交通會依多人共食做小幅下修。近期已新增六月相關費用：

- `六月來回機票`: NT$6,575-8,450，依航班資訊表台灣虎航 6/1 去程 + 6/4 回程
- `六月梅雨備品`: NT$300-1,200，摺疊傘、防水鞋套 / 防潑水噴霧、薄外套、防曬
- `四天吃玩＋市區交通`: NT$5,193-10,855，依 D1-D4 日小計，已扣 Pass 可覆蓋門票
- `住宿 3 晚`: NT$4,200-9,000，獨旅以單人房估，多人可分攤
- `釜山 Pass`: NT$1,625-2,125，Big5 ₩65,000 或 48H ₩85,000 粗估
- `保留計程車 / 遠距移動`: NT$800-2,600
