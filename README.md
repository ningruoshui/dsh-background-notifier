# DSH Notifier – 后台确认提醒插件

当 DeepSeek Harness 中的 AI Agent 需要用户确认、授权或询问时，如果 Harness 应用处于后台（最小化、失焦或标签页未激活），本插件会发送系统桌面通知，避免用户错过重要交互请求。

## 功能特性

- **增量检测**：仅处理 DOM 变化部分，资源占用极低，不影响 Harness 性能。
- **后台提醒**：仅当页面不可见时才发送通知，避免干扰正常操作。
- **可配置**：支持自定义关键词、CSS 选择器和通知冷却时间。
- **跨平台**：基于标准 Web API，适用于桌面端（Windows/macOS/Linux）和 Android 客户端（WebView）。
- **防抖机制**：合并高频变化，避免重复通知。

## 安装方法

1. 将本插件文件夹放置到 Harness 的插件目录（或通过 `dsh plugin add` 安装）。
2. 确保插件被加载（重启 Harness 或重新加载插件）。
3. 首次使用时，浏览器可能会询问通知权限，请允许。
4. （可选）修改配置：在 Harness 页面打开开发者工具，执行 `window.__dshNotifierSetConfig({...})` 可动态调整。

## 配置说明

| 配置项 | 类型 | 默认值 | 描述 |
|--------|------|--------|------|
| `keywords` | string[] | `['确认','继续','是','允许','同意','确定','接受','下一步','提交']` | 按钮文本包含这些关键词时触发通知 |
| `selectors` | string[] | `[]` | 额外的 CSS 选择器，匹配的元素会触发通知 |
| `cooldown` | number | `10000` | 两次通知的最小间隔（毫秒） |
| `requireInteraction` | boolean | `true` | 通知是否常驻，直到用户手动关闭 |

修改配置示例：
```javascript
window.__dshNotifierSetConfig({
  keywords: ['确认', '允许'],
  cooldown: 5000,
  requireInteraction: false
})
