# WebClaw Shell — Design Spec

## Overview

Single-page web shell at `shell.mghm.ai` that combines an xterm.js terminal (WebSocket to `wss://main.mghm.ai`) with an embedded webchat iframe (`main.mghm.ai/chat`). Deployed on Cloudflare Pages. Single `index.html`, no build step.

## Layout

```
┌─────────────────────────────────────────┐
│ STATUS BAR                              │
├──────────────────┬──────────────────────┤
│ TERMINAL (60%)   │ WEBCHAT (40%)        │
│ xterm.js         │ iframe               │
└──────────────────┴──────────────────────┘
```

Mobile: panels stack vertically (terminal on top, chat below).

## Components

### 1. Status Bar

- Left: `🦞 WebClaw Shell` branding
- Center: Gateway status indicator (green/red dot + label), Mac Studio status indicator
- Right: Active model name, session count, Monterrey time (CT)
- Health polling: `GET /health` on `main.mghm.ai` every 30s
- Ollama polling: `GET ollama.mghm.ai/api/tags` every 30s — extracts model name
- Indicators: green circle = up, red circle = down

### 2. Terminal Panel (left, 60% width)

- xterm.js loaded via CDN (v5)
- Addons: xterm-addon-fit, xterm-addon-web-links (CDN)
- Dark theme: background `#1a1a2e`, foreground `#e0e0e0`, cursor `#ff6b6b`
- Connection flow:
  1. Page loads → terminal renders banner: `🦞 WebClaw Shell v1.0`
  2. Shows `Connecting to main.mghm.ai...`
  3. Prompts `Password: ` with hidden input
  4. User types password + Enter
  5. WebSocket opens to `wss://main.mghm.ai` with password in initial auth message
  6. On success: `Connected to WebClaw Gateway ✅`
  7. On failure: `Connection failed. Retry? (y/n)`
- Auto-reconnect with exponential backoff (1s, 2s, 4s, 8s, max 30s)
- Password stored in `sessionStorage` only (for reconnect)
- Resizes with window via fit addon

### 3. WebChat Panel (right, 40% width)

- `<iframe src="https://main.mghm.ai/chat?session=main">`
- Full height of content area
- No border, seamless integration

### 4. Resizer

- Draggable divider between terminal and chat panels
- 4px wide, subtle highlight on hover

## Style

- Dark theme throughout
- Color palette: deep navy (`#1a1a2e`), lobster red accents (`#ff6b6b`), ocean teal (`#4ecdc4`), warm coral (`#ff8a5c`)
- Font: system monospace for terminal, system sans-serif for status bar
- Status bar: slightly lighter background (`#16213e`), 48px height
- Panels: no visible borders, subtle shadow on divider

## Tech Stack

- Single `index.html` file
- xterm.js v5 via CDN (jsdelivr)
- xterm-addon-fit via CDN
- xterm-addon-web-links via CDN
- Vanilla JS, CSS (no framework, no build)
- Deployed to Cloudflare Pages

## Deployment

1. GitHub repo: `webclaw-shell`
2. Cloudflare Pages project connected to repo
3. Custom domain: `shell.mghm.ai` (CNAME to Cloudflare Pages)

## Constraints

- Single file only
- No npm, no build step
- Works in any modern browser
- Password never hardcoded — sessionStorage only
- All external connections use HTTPS/WSS
