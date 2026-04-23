# pino-logviewer

A tiny, dependency-free web UI for reading [pino](https://github.com/pinojs/pino) logs in the browser. Paste raw log output, get a filterable, colour-coded table with expandable JSON detail for each entry.

It also understands the wrapper format emitted by **Azure Container Apps** (`az containerapp logs show` / log-stream), so you can paste ACA output directly without pre-processing.

## What it handles

Each pasted line is classified into one of three shapes:

| Input line                                                                                     | Result                                                               |
| ---------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| `{"level":30,"time":1776934990184,"msg":"..."}`                                                | Full pino entry, all fields available in the detail view             |
| `2026-04-23T09:03:10.1854061Z stdout F {"level":30,...}`                                       | ACA-wrapped pino — prefix stripped, pino fields preserved            |
| `2026-04-23T09:03:11.0077182Z stdout F GET /startupz 200 3.573 ms - 2`                         | ACA-wrapped plain text — shown as a raw row at the outer timestamp   |
| `2026-04-23T09:18:52.91159  Connecting to the container 'my-container-app'...`              | Bare banner/status line — shown as a raw row at the leading timestamp |

`stdout` / `stderr` are surfaced as badges on each row. Raw (non-JSON) stderr lines default to level 50 (error); raw stdout lines to 30 (info).

## Features

- **In-browser parsing** — nothing leaves your machine. The server is a static-file host; all parsing happens in `index.html`.
- **Level filter** — hide everything below trace/debug/info/warn/error/fatal.
- **Free-text filter** — substring match across the full JSON of each entry (useful for trace ids, request ids, hostnames, etc.).
- **Click a trace id** to filter down to that single request/trace.
- **Click a row** to expand pretty-printed JSON + stack trace (from `err.stack`, `error.stack`, or `stack`).
- **Colour-coded levels** — warn/error/fatal rows are tinted so you can skim a long paste.

## Running it

```bash
node server.js
```

Opens a static file server on <http://localhost:3100>. No dependencies — plain Node `http`.

You can also just open `index.html` directly in a browser (`file://…/index.html`) — the server is only there for convenience.

## Usage

1. Start the server (or open `index.html`).
2. Paste log output into the textarea. Mixed formats are fine — ACA wrappers, bare pino JSON, and non-JSON lines can be pasted together.
3. Hit **Parse** (or `⌘/Ctrl + Enter` inside the textarea).
4. Use the level dropdown and filter box to narrow things down. Click rows for detail.

### Grabbing logs from Azure Container Apps

```bash
az containerapp logs show \
  --name my-container-app \
  --resource-group my-rg \
  --tail 500 \
  > logs.txt
```

Paste the contents of `logs.txt` straight in — the ACA prefix and any connection banners will be handled automatically.

## Fields recognised

The pretty renderer will pick up the following fields if present:

- `time` — ms-since-epoch (pino default) or any string parseable by `new Date()`. If the entry came from an ACA-wrapped line and has no `time`, the outer timestamp is used as a fallback.
- `level` — numeric pino level (10/20/30/40/50/60) or a string.
- `msg` — the message column.
- `name`, `pid` — shown as badges.
- `traceId` / `trace_id` / `traceID` / `trace` / `requestId` / `reqId` — shown as a clickable trace-id column.
- `err.stack` / `error.stack` / `stack` — rendered as a stack trace block in the detail view.

Any other fields are preserved and shown in the expanded JSON view.

## Project layout

```
.
├── index.html   # the whole app — HTML, CSS, and the parser/renderer
└── server.js    # 28-line static file server (optional)
```

No build step, no bundler, no npm install. Edit `index.html` and refresh.

## License

MIT
