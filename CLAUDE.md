# Double Click Close Tab

Chrome extension that closes a tab when you double left-click on an empty area of the page.

## Architecture

Three files, standard Manifest V3 content-script pattern:

```
manifest.json   — extension manifest (MV3)
background.js   — service worker: receives "closeTab" message → chrome.tabs.remove()
content.js      — content script: detects double left-click → sends "closeTab" message
```

### How it works

1. `content.js` is injected into every page (`<all_urls>`, `document_start`).
2. It listens for `mouseup` events (left button only) and tracks click timing.
3. When two clicks land within 500 ms (with a 50 ms minimum gap to debounce):
   - If text was selected by the double-click → **skip** (user is selecting text).
   - If the target is an interactive element (input, link, button, video, contenteditable) → **skip**.
   - Otherwise → send `{ action: "closeTab" }` to the background service worker.
4. `background.js` calls `chrome.tabs.remove(sender.tab.id)` to close the tab.

### Why `mouseup` instead of `dblclick`

Following the pattern from [close-tab-by-double-right-click](https://github.com/webextensions/close-tab-by-double-right-click). Using `mouseup` with manual timing gives control over the click interval and avoids issues with the browser's built-in `dblclick` event (which fires after text selection is already committed, making it harder to distinguish intentional selection from a close gesture).

## Limitations

- **Does not work on `chrome://` pages**, the Chrome Web Store, or `about:blank` — Chrome blocks content script injection on these pages.
- **Does not work on the browser tab strip itself** — the tab strip is browser chrome; no extension API can listen for mouse events there. All existing extensions (Double Click Closes Tab, Double Middle-Click Close Tab, Close Tab by Double Right Click) share this same limitation and detect clicks on the **page content** instead.
- **Newly installed**: content script only loads on new page navigations. Already-open tabs need a reload for the extension to activate.

## Install (development)

1. Open `chrome://extensions`
2. Enable **Developer mode** (top-right)
3. Click **Load unpacked** → select this folder
4. Reload any open tabs to inject the content script

## Key constants (content.js)

| Constant               | Default | Purpose                                      |
|------------------------|---------|----------------------------------------------|
| `DOUBLE_CLICK_INTERVAL`| 500 ms  | Max gap between two clicks to count as double |
| `MIN_CLICK_GAP`        | 50 ms   | Ignore impossibly fast repeated events        |

## References

- [close-tab-by-double-right-click](https://github.com/webextensions/close-tab-by-double-right-click) — open-source reference implementation (right-click variant)
- [Chrome Extensions Manifest V3 docs](https://developer.chrome.com/docs/extensions/mv3/)
