# DSH Notifier – Background Confirmation Reminder

[English](README-en.md) | [简体中文](README.md)

When the AI Agent in DeepSeek Harness requires user confirmation, authorization, or asks a question, and the application is in the background (minimized, unfocused, or tab inactive), this plugin sends a system desktop notification to avoid missing important interactions.

## Features

- **Incremental DOM detection**: Only processes DOM changes, keeping resource usage extremely low.
- **Background reminder**: Notifies only when the Harness page is not visible.
- **Configurable**: Custom keywords, CSS selectors, and notification cooldown.
- **Cross-platform**: Based on standard Web APIs, works on desktop (Windows/macOS/Linux) and Android WebView.
- **Debounced scanning**: Merges frequent changes to avoid redundant notifications.
- **One-click focus**: Notification includes a click action that attempts to focus the Harness window.

## File Structure

```
dsh-notifier/
├── package.json          # Plugin package metadata
├── src/
│   ├── host.js           # Main process plugin entry (provides client script route)
│   └── client.js         # Renderer monitoring script (injected into Web UI)
├── config.js             # Default configuration (optional, can be merged into client.js)
└── README.md             # This file
```

## Installation

1. Place the plugin folder into your Harness plugin directory (or install via `dsh plugin add`).
2. Ensure the plugin is loaded (restart Harness or reload plugins).
3. On first use, the browser may ask for notification permission – please allow it.
4. (Optional) Customize settings by opening the browser developer tools in Harness and running `window.__dshNotifierSetConfig({...})`.

## Usage

- Use Harness normally. When a confirmation request appears and the application is in the background, a system notification will pop up.
- Click the notification to return to the Harness window.
- To adjust detection rules, use the developer tools console or edit the plugin configuration (see below).

## Configuration

The default configuration is shown below. You can modify it via `window.__dshNotifierSetConfig()` in the Harness page's developer console.

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `keywords` | string[] | `['确认','继续','是','允许','同意','确定','接受','下一步','提交']` | Button text containing any of these keywords triggers a notification |
| `selectors` | string[] | `[]` | Additional CSS selectors; matching elements trigger notifications |
| `cooldown` | number | `10000` | Minimum interval between notifications (milliseconds) |
| `requireInteraction` | boolean | `true` | Whether the notification stays until the user closes it |

Example to update settings:
```javascript
window.__dshNotifierSetConfig({
  keywords: ['确认', '允许'],
  cooldown: 5000,
  requireInteraction: false
})
```

## How It Works

1. **Incremental DOM listening**: Uses `MutationObserver` to listen for `childList` and `attributes` changes, processing only the added nodes or changed elements.
2. **Confirmation element detection**: Checks if an element is button-like (`button`, `a`, `input[type=button]`, `[role=button]`, `.btn`) and whether its text contains any configured keyword or matches a selector.
3. **Visibility check**: Determines if the Harness page is in the foreground using `document.hidden` and window focus events.
4. **System notification**: Uses the Web `Notification` API to send a system-level notification and handle click-to-focus.

## Known Limitations

- **Script injection**: This plugin assumes Harness automatically loads the `/dsh-notifier/client.js` route. If your DSH version uses a different client plugin injection mechanism, adjust `host.js` accordingly.
- **Notification click focus**: Because DSH does not expose Electron window control, clicking the notification only calls `window.focus()`; whether the window is brought to the front depends on the host environment.
- **DOM change miss**: In rare cases, a confirmation element might appear by modifying the text of an existing element rather than adding a new node or changing attributes. Official support for a "waiting for confirmation" event would improve detection accuracy.

## Contributing

Issues and pull requests are welcome. Please ensure code style consistency and test before submitting.

## License

MIT License

This README is ready to be placed in your project root. You can also create a `README.zh-CN.md` for the Chinese version using the same structure and link to it from the top as shown.
