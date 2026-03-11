# Double Click Close Tab

A lightweight Chrome extension that closes the current tab when you double left-click on an empty area of a web page.

## Features

- **Double-click to close** — Double left-click on any empty space on a page to instantly close the tab.
- **On/Off toggle** — Click the extension icon in the toolbar to enable or disable with a single switch.
- **Smart detection** — Won't trigger when you're selecting text, clicking links, buttons, inputs, or other interactive elements.
- **Minimal footprint** — No analytics, no tracking, no external requests. Just a content script and a service worker.

## Install from Chrome Web Store

> Coming soon

## Install (Development)

1. Clone or download this repository.
2. Open `chrome://extensions` in Chrome.
3. Enable **Developer mode** (top-right toggle).
4. Click **Load unpacked** and select the project folder.
5. Reload any open tabs for the content script to activate.

## How It Works

1. A content script is injected into every page.
2. It listens for `mouseup` events (left button only) and tracks click timing.
3. When two clicks happen within 500 ms:
   - If text was selected — **skip** (user is selecting text).
   - If the target is interactive (input, link, button, video, contenteditable, etc.) — **skip**.
   - Otherwise — send a `closeTab` message to the background service worker, which calls `chrome.tabs.remove()`.

The `mouseup` approach (instead of the native `dblclick` event) gives precise control over click intervals and avoids issues with text selection.

## Project Structure

```
manifest.json   — Extension manifest (Manifest V3)
background.js   — Service worker: handles closeTab messages & on/off state
content.js      — Content script: detects double left-click on empty areas
popup.html      — Toolbar popup with on/off toggle switch
popup.js        — Popup logic: reads/writes enabled state
icons/          — Extension icons (on/off variants at 16, 48, 128px)
doc/            — Privacy policy
```

## Permissions

| Permission | Why |
|---|---|
| `<all_urls>` (content script) | Detect double-click events on web pages |
| `storage` | Save the on/off toggle preference |

No data is collected or transmitted. See the full [Privacy Policy](doc/privacy-policy.html).

## Known Limitations

- **Does not work on `chrome://` pages**, the Chrome Web Store, or `about:blank` — Chrome blocks content script injection on these pages.
- **Does not work on the browser tab strip** — The tab strip is browser UI; no extension can listen for mouse events there. All similar extensions share this limitation.
- **Newly installed** — Already-open tabs need a reload for the content script to activate.

## License

MIT
