# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file static HTML travel itinerary for a Busan, South Korea group trip (4 days / 3 nights, September departure, 9–13 people, chartered van). All content is in Traditional Chinese (zh-TW).

## Development

No build tools or dependencies. Open `index.html` directly in a browser to preview.

## Architecture

Everything lives in `index.html` — inline CSS (`<style>`), static HTML content, no JavaScript. The layout uses:

- `.section` cards with `.day-header`, `.timeline`, `.hotel-bar`, `.car-bar` sub-components
- `.tl-item` timeline rows (time → dot/line → body)
- Tag pills (`.tag-food`, `.tag-spot`, `.tag-car`, etc.) for category labels
- Naver Map deep links (`nmap://` scheme) for location anchors
- A budget table, checklist section, weather grid, and IG spot list at the end

Day sections follow the color convention: D1 blue → D2 green → D3 purple → D4 red.
