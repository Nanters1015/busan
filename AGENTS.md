# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

A single-file static HTML travel itinerary for a Busan, South Korea trip (4 days / 3 nights, June or September departure, solo-first with optional group scaling, Busan Pass + subway/walking-first transport). Chartered van/taxi is reserved for long-distance or station-inconvenient segments. All content is in Traditional Chinese (zh-TW).

## Development

No build tools or dependencies. Open `index.html` directly in a browser to preview.
Deployed to Netlify as a static site.

## Architecture

Everything lives in `index.html` — inline CSS (`<style>`), HTML content, and JavaScript. The layout uses:

- `.card` sections with `.day-header`, `.timeline`, `.hotel-bar`, `.car-bar` sub-components
- `.tl-item` timeline rows (time → dot/line → body)
- Tag pills (`.tag-food`, `.tag-spot`, `.tag-car`, etc.) for category labels
- Naver Map deep links (`nmap://` scheme) for location anchors
- A budget table, checklist section, June/September weather grid, transport estimate, flight table, hotel info, and IG/Thread/video spot lists
- SortableJS (CDN) for drag-and-drop card layout; order persisted in `localStorage`

Day sections follow the color convention: D1 blue → D2 green → D3 purple → D4 red.

## Trip planning conventions

- Header currently presents `6月 / 9月出發 × 獨旅到多人彈性版 × Pass＋地鐵/步行優先`.
- Transport principle: if walking to a station/destination is within about 10 minutes, prefer subway/walking; otherwise retain taxi/chartered van.
- Pass-covered items are deducted in timeline/day cost notes: Hotel Aqua Palace, BUSAN X the Sky, Skyline Luge, and Lotte World Adventure Busan.
- Sky Capsule is not deducted from Pass savings because common Busan Pass coverage is Beach Train and may not include Sky Capsule.
- `#card-weather` includes both June and September departure info. June copy should mention early/mid-June as preferable and late-June rainy season risk.

## IG 收藏地點（動態）

The IG spot list (`#card-ig`) is dynamically loaded from Google Sheets via Google Apps Script.

- **Google Sheets ID**: `1ZXw0zHHBW48ApYBE070nGuB5ZGsTE_lvmVtQIMNLmVs`
- **Sheet tab name**: `ig_spots`
- **GAS Web App URL**: `https://script.google.com/macros/s/AKfycbyCHxzRostMcu7OFiAJYqb0Q188qF_wYDMoODfE9os5NZ9INQTD_w4z-UXmS-78Q8B4Gg/exec`
- **Columns**: `id | name | detail | day | icon | color1 | color2 | naverUrl | igUrl`
- **API actions** (all via GET params): `?action=get`, `?action=add&...`, `?action=delete&id=...`

Card colors are auto-assigned by day (D1 blue / D2 green / D3 purple / D4 red); `color1`/`color2` in Sheets can override the gradient per card.

Key JS functions: `loadIgSpots()`, `renderIgCards(spots)`, `openModal('ig')`, `submitAddSpot()`, `deleteIgSpot(id)`.

## 費用試算（動態）

The budget card (`#card-budget`) is dynamically loaded from Google Sheets via GAS.

- **Sheet tab name**: `budget`
- **Columns**: `id | label | desc | price_min | price_max`
- **API actions**: `?action=get_budget`, `?action=add_budget&...`, `?action=delete_budget&id=...`
- **Semantics**: rows are stored as 1-person baseline costs. The card recalculates per-person and whole-party totals by headcount: flights/pass/gear are fixed per person, lodging is split as a double room, taxis are split by 4-seat cars, and shared food/play costs receive a small group-dining adjustment.
- **Current June rows**:
  - `六月來回機票`: NT$6,575-8,450, based on Tigerair 6/1 outbound + 6/4 return rows in the flights table
  - `六月梅雨備品`: NT$300-1,200, umbrella/waterproof shoe cover or spray/light jacket/sunscreen
  - `四天吃玩＋市區交通`: NT$5,193-10,855, based on D1-D4 day totals after Pass-covered attractions are deducted
  - `住宿 3 晚`: NT$4,200-9,000, solo room estimate; group travel can split this
  - `釜山 Pass`: NT$1,625-2,125, estimated from Big5 ₩65,000 or 48H ₩85,000
  - `保留計程車 / 遠距移動`: NT$800-2,600
