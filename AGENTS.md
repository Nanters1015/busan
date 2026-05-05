# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

A single-file static HTML travel itinerary for a Busan, South Korea group trip (4 days / 3 nights, September departure, 9–13 people, chartered van). All content is in Traditional Chinese (zh-TW).

## Development

No build tools or dependencies. Open `index.html` directly in a browser to preview.
Deployed to Netlify as a static site.

## Architecture

Everything lives in `index.html` — inline CSS (`<style>`), HTML content, and JavaScript. The layout uses:

- `.section` cards with `.day-header`, `.timeline`, `.hotel-bar`, `.car-bar` sub-components
- `.tl-item` timeline rows (time → dot/line → body)
- Tag pills (`.tag-food`, `.tag-spot`, `.tag-car`, etc.) for category labels
- Naver Map deep links (`nmap://` scheme) for location anchors
- A budget table, checklist section, weather grid, and IG spot list
- SortableJS (CDN) for drag-and-drop card layout; order persisted in `localStorage`

Day sections follow the color convention: D1 blue → D2 green → D3 purple → D4 red.

## IG 收藏地點（動態）

The IG spot list (`#card-ig`) is dynamically loaded from Google Sheets via Google Apps Script.

- **Google Sheets ID**: `1ZXw0zHHBW48ApYBE070nGuB5ZGsTE_lvmVtQIMNLmVs`
- **Sheet tab name**: `ig_spots`
- **GAS Web App URL**: `https://script.google.com/macros/s/AKfycbyCHxzRostMcu7OFiAJYqb0Q188qF_wYDMoODfE9os5NZ9INQTD_w4z-UXmS-78Q8B4Gg/exec`
- **Columns**: `id | name | detail | day | icon | color1 | color2 | naverUrl`
- **API actions** (all via GET params): `?action=get`, `?action=add&...`, `?action=delete&id=...`

Card colors are auto-assigned by day (D1 blue / D2 green / D3 purple / D4 red); `color1`/`color2` in Sheets can override the gradient per card.

Key JS functions: `loadIgSpots()`, `renderIgCards(spots)`, `showAddModal()`, `submitAddSpot()`, `deleteIgSpot(id)`.
